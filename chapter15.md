# Notes

這章講Smart Pointer智慧型指標。

## 先備知識

指標是一種指向記憶體位址的變數。在Rust中最常見的是以參考型式`&`存在，不過沒有overhead。
而智慧型指標則是用在有更間接的情況，如指向metadata詮釋資料的情形。Rust的智慧型指標是自C++11來的，能提供比參考更強力的使用性。
通常智慧型指標會與結構連用，而其中的`Deref`與`Drop`兩種特性能讓你更便利的使用智慧型指標。
標準函式庫中常見的智慧型指標有3類：
- `Box<T>`能提供在heap上的定址。
- `Rc<T>`能允許資料的多重所有權，並在無擁有者時釋放資料。
- `Ref<T>`、`RefMut<T>`，以`RefCell<T>`存取，能讓借用規則從編譯時檢查轉向在執行時檢查，有可能造成memory leak。

## `Box<T>`

常見的使用情境：
- 有需要在執行期才能確定大小，並要精準需求記憶體的型態。
- 有大量需要轉移所有權的資料，但需要保證轉移時資料不會被複製。
- 當想擁有一個只關注能實作出特定trait的型態的值，而不是一個特定型態的值。

### 遞迴型態

遞迴型態是一個會包含自身型態的型態，因為這種巢狀結構理論上是增長至無限大，所以Rust無法推斷型態的大小。如下的程式會編不過：

``` rust
enum List {
    Cons(i32, List),
    Nil,
}
```

而`Box<T>`則因為是指標，編譯器可以確定其大小，因此編譯不會有問題：

``` rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

## `Rc<T>`

有時一個值會需要有多個擁有者。舉例來說，資料結構中的graph，會有多個edge指向同一個node，而這個node會被這些edge所共有，直到所有指向node的edge都被刪除才會釋放掉這個node。
Rust中的`Rc<T>`就是因應此而生，這是*reference counting*參考計數的縮寫。它會追蹤資料的所有權者的數量，到0時會釋放掉資料。
`Rc<T>`是只在單執行緒的情況下使用，如果要做到多執行緒則需要用到`Arc<T>`。

### 用`Rc<T>`分享資料

以下的情況用`Box<T>`會出現編譯錯誤：

``` rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

而改換成`Rc<T>`後則可以通過編譯：

``` rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Cons(5, Rc::new(Cons(10, Rc::new(Nil))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

要記得加上`use std::rc::Rc`。請注意`Rc::clone`本身不會進行深度複製，而是增加參考的計數。

## `RefCell<T>`

*Interior mutability*是一種Rust的設計模式，讓你能在不可變的參考中變動其資料，其中必定會用到`unsafe`來規避Rust的常用規則。
`RefCell<T>`讓借用與變動規則從編譯時確認改為執行時才確認，當規則被破壞時，程式會panic。
預設的靜態分析，在某些特定情況下會無法保證程式的正確性，從而給出編譯錯誤。
因為這些情況無法使用靜態分析保證，而又能在實際執行時不違反規則，所以有`RefCell<T>`型態來方便此種程式碼的編譯。
與`Rc<T>`相同，僅能用在單執行緒的情形下。`RefCell<T>`的多執行緒需要使用`Mutex<T>`。

### 使用情況：模擬物件

*Test double*測試替身是一個常用概念，用來替代測試程式中的相依型態。
*Mock objects*模擬物件是測試替身的一種，能夠記錄測試之中發生的事，讓你能夠斷定在對的地方發生對的事情。
以下是一個測試情境：我們需要寫個程式驗證使用者的請求次數是否達到上限，若超過則發送訊息。

``` rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }
    
    pub fn set_value(&mut self, value: usize) {
        self.value = value;
        
        let percentage_of_max = self.value as f64 / self.max as f64;
        
        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        }
    }
}
```

我們的模擬物件只要做出有`Messenger` trait並有一個`send`函數就行了。
以下是測試的程式碼：

``` rust
#[cfg(test)]
mod tests {
    use super::*;
    
    struct MockMessenger {
        sent_messages: Vec<String>,
    }
    
    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }
    
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message)); // error here
        }
    }
    
    #[test]
    fn it_sends_an_over_limit_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);
        
        limit_tracker.set_value(120);
        
        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

實際上的`send`實作上會出現為了追蹤訊息而變動內容，但是在參數上又不能變更為`&mut self`，因此這時需要使用到`RefCell<T>`。

``` rust
// --snip--
    use std::cell::RefCell;
    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }
// --snip--
```

## 智慧型指標的選擇

大多的選擇為單一種。
- `Rc<T>`允許同個資料有多個擁有者；`Box<T>`與`RefCell<T>`只允許單個擁有者。
- `Box<T>`允許不可變或可變的借用；`Rc<T>`只允許不可變的借用；`RefCell<T>`允許在執行期才檢查不可變與可變的借用。
- `RefCell<T>`可以本身不可變但是其借用的值仍可以改動。
也可以進行組合，如`Rc<RefCell<T>>`可達成多擁有者的可變借用。

## `Deref` Trait

這個trait讓你能自訂解參考運算子`*`的行為。如以下的程式碼：

``` rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // deref imply here
}
```

### 自訂智慧型指標

來簡單的自製一個類似`Box<T>`的型態，以了解`Deref`的功能。

``` rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

這樣的程式碼編譯後會出錯，還需要加上`Deref` trait的實作：

``` rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

### 隱藏的強制解參考

*Deref coercion*強制解參考是一個Rust中方便使用的功能，這只在有實作`Deref` trait的型態會使用。
這在我們傳一個參考進函數時，如果參考本身並沒有對應到函數的參數，則會自動轉換成`deref`所回傳型態的參考，如上面的`&MyBox<T>`會被轉為`&T`。
另一方面，強制解參考也讓程式碼中不會充斥著成對的`&`與`*`，讓程式能更易讀。

#### 強制解參考與可變性

Rust的強制解參考與可變性會有三種對應關係：
- 當`T: Deref<Target=U>`時，自`&T`轉成`&U`。
- 當`T: DerefMut<Target=U>`時，自`&mut T`轉成`&mut U`。
- 當`T: Deref<Target=U>`時，自`&mut T`轉成`&U`。

## `Drop` Trait

`Drop`是另一個智慧型指標的重點，這個trait讓你能自訂生命期要結束時會發生什麼事。
C++中，在`new`之後，若是忘記`delete`，就會造成memory leak，從而使程式越來越大而當機。
而在Rust，這些可以藉由實作`Drop`與生命期間的配合達成自動釋放，解決memory leak的可能。

### 實作

``` rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping with data `{}`!", self.data);
    }
}
```

### 提早釋放

在生命期結束時系統會自動呼叫`drop()`。若要提早讓程式釋放變數，不能直接呼叫`drop()`，而是要使用`std::mem::drop`強制一個值在其生命期結束前就被釋放。
如果有變數`c`要被手動釋放，`c.drop()`會報錯，必須使用`drop(c)`才能執行。

## Reference Cycle 循環參考

當同時使用`Rc<T>`與`RefCell<T>`時，就有機會遇到循環參考，這會造成計數無法歸零，而造成memory leak。
預防的辦法是把`Rc<T>`使用`Rc::downgrade`轉成`Weak<T>`。計數則從`strong_count`轉為從`weak_count`計算。與原本的不同點在於，`strong_count`必須為0時才清除，而`weak_count`不用。
強參考表示你分享了`Rc<T>`的實例，而弱參考則沒有此關係的表達。因此包含到弱參考的循環參考就會在強參考的計數為0時被破壞掉。

因為`Weak<T>`所參考的值可能被釋放掉，所以需要以`upgrade`確認值是否仍存在，回傳值的`Option<Rc<T>>`表示是否存在。

### 例子

``` rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    })
}
```

以上為一個很簡單的樹狀結構。
如果我們需要連回parent，則會遇到上面所說的循環參考的情況，此時改用`RefCell<Weak<Node>>`：

``` rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

最後用這來演示一下`strong_count`與`weak_count`的差別：
``` rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

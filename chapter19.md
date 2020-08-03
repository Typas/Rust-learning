# Notes

這章講進階功能。

## `unsafe`

靜態分析本身很保守，對於不能保證安全性的程式碼，編譯器會一律拒絕通過。而有些場合是編寫者能擔保安全性的，`unsafe`這個關鍵字便是標註此段程式碼是由編寫者所擔保安全，而非編譯器。

### 特別權限

這東西可以想成像linux的`sudo`一樣，是有些新的權限存在。

- 對原始指標解參考
- 呼叫`unsafe`函數或method
- 存取或改動靜態變數
- 實作`unsafe` trait
- 存取`union`的欄位

這些權限所對應到的是記憶體安全相關的檢查，其他如借用等安全檢查仍然會執行。原則是儘可能的縮小這些需要擔保的範圍，在debug時就可以不用這麼大費周章。

#### 原始指標

不可變的原始指標型態是`*const T`，可變的是`*mut T`。原始指標相對於參考與智慧型指標有以下差異：

- 允許多個可變或不可變指標指向同一位置，以忽略借用規則。
- 不保證指向合法的記憶體位址
- 允許null存在
- 沒有任何自動清理的實作

建立原始指標本身是安全的，但解參考則需要用`unsafe`擔保安全性。

``` rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

以上程式碼等同於以下的C程式碼：

``` c
int num = 5;

const int * const r1 = &num;
int * const r2 = &num;

printf("r1 is: %d\n", *r1);
printf("r2 is: %d\n", *r2);
```

#### `unsafe` 函數

只要想使用到unsafe函數就必須在unsafe區域呼叫。

``` rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

#### 內含 `unsafe` 程式碼的一般函數

程式碼中使用`unsafe`有些時候是為了效率，有些時候是編譯器不夠聰明。當函數本身可以保證安全的時候，並不需要整個標記成`unsafe`，這種包裝`unsafe`的概念非常常見。舉例來說，`split_at_mut`這個函數，因為借用檢查不夠聰明的關係，並不能判斷對不同切片之間有沒有重疊，因此需要自行擔保。

``` rust
use std::slice;

fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];

    let (a, b) = split_at_mut(r, 3);

    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
}

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

#### 跨語言函數

有些時候，會需要用到其他語言寫的程式碼，Rust有關鍵字`extern`來建立FFI存取其他語言的函數。而使用這些函數時需要用到`unsafe`包起來自行提供擔保。

``` rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absoulute value of -3 according to C: {}", abs(-3));
    }
}
```

不過如果是提供給其他語言的函數，則不需要用到`unsafe`。只要用`extern`搭配`#[no_mangle]`就可以了。

``` rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

#### 靜態變數

靜態變數的生命期必為`'static`，常數與不可變的靜態變數很類似，不過靜態變數的記憶體位址不會變，而常數則可能被複製到好幾個地方去。可變的靜態變數則需要在`unsafe`區域中使用。如果想初始化後作為常數使用，有[lazy-static](https://crates.io/crates/lazy_static)可以用。

``` rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER = {}", COUNTER);
    }
}
```

#### `unsafe` trait

跟函數一樣，用到就需要以`unsafe`標註。

``` rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementation go here
}
```

#### `union`

這東西是拿來跟C的union接的，一般情況下`enum`可以替代。

## Advanced Traits

### Associated Types 關聯型態

關聯型態可以用來指定一些trait method所需的型態，最常見的是實作`Iterator`。

``` rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

其中的`Item`是關聯型態，它能接受其他型態，而且能限縮實作的型態只有一個。

### Default Generic Type Parameters 預設泛型參數

語法是`<PlaceholderType=ConcreteType>`，例子像是用在operator overloading時。宣告如下：

``` rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

而實作會如下：
``` rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Millimeters) -> Millimeters {
        Millimeters(self.0 + other.0)
    }
}

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

#### 使用時機

- 擴展型態，而不破壞原有的程式碼。
- 允許特定情況的自訂，但大多時候沒必要。

### Fully Qualified Syntax

這東西是在有多個trait有同一個method，並且都實作在同一個struct或enum內時需要使用。

``` rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

### Supertrait

當你的trait會需要其他trait的實作時，這個trait就被稱為supertrait。如下的`Student`與`ComSciStudent`就是supertrait：

``` rust
trait Person {
    fn name(&self) -> String;
}

trait Student: Person {
    fn university(&self) -> String; 
}

trait Programmer {
    fn fav_language(&self) -> String;
}

trait ComSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}

fn com_sci_student_greeting(student: &dyn ComSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My Git username is {}.",
        student.name(),
        student.university(),
        student.git_username(),
    )
}
```

### Newtype Pattern

在Rust中有個法則稱為orphan rule，要在一個型態實作trait必須滿足型態或trait是在程式碼所在的crate內。若兩者都為外源，則需要用一個tuple struct包裝，而這種抽象化在Rust中是零成本的。

``` rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("Hello"), String::from("world")]);
    println!("w = {}", w);
}
```

## 進階型態

### 型態別名

這創出來的新型態與newtype pattern的不同，前者完全等價於原型態，而後者被視作新型態。

``` rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

這常被用來減少一些重複的長型態，以增加可讀性：

``` rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
```

### Never 型態

Rust有個非常特別的型態`!`，通常被稱作never type。在`match`中，這個型態會被當作是任何型態，因此當never型態與另一種型態混用時，編譯器會算出唯一的型態而不產生衝突。

### 動態大小的型態

Rust會需要在編譯時就知道型態的大小，而有一些型態的大小則只能在執行時知道，這些型態被稱為dynamically sized types動態大小型態，簡稱DST。DST本身的大小在編譯時未知，但是其參考的大小則是已知：變數的位址，是一個固定長度的值；變數的大小，以`usize`儲存。而其他型態的指標也都如此。最常見的DST是`str`與各種`dyn`修飾的型態。在泛型中要用到DST，除了用這些包裝外，也能用`?Sized`來解除trait限制。

``` rust
fn generic<T>(t: T) {
    // ---snip---
}

fn generic<T: Sized>(t: T) {
    // ---snip---
}

fn generic<T: ?Sized>(t: T) {
    // ---snip---
}
```

上面程式碼的前兩者是一樣的，而第三種則是表達參數的大小可能不固定，只有第三種可以使用DST作為泛型參數。

## 進階函數

### 函數指標

這東西是拿來跟沒有closure的語言對接用，例如C。一般情況下`Fn`、`FnMut`與`FnOnce`就可以達成這個效果。

``` rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

### `enum` 的特殊用法

列舉中的tuple struct其形式長得很像函數參數，而Rust中也能把該列舉項目當作是函數來初始化：

``` rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

### 回傳 Closure

函數指標是不能當作回傳型態的，而要回傳closure，則需要做成trait object回傳：

``` rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

## 巨集

Rust的巨集有四種，一種declarative macro 宣告式巨集與三種procedural macro 程序式巨集：

- 宣告式巨集：以`macro_rules!`開頭
- 自訂衍生巨集：自訂新的`derive`屬性
- 類屬性巨集：自訂新的屬性
- 類函數巨集：形似函數呼叫，但是處理上不同。

### 巨集與函數的差別

基本上，巨集是一段用來寫程式碼的程式碼，又被稱為「元程式」。相對於函數，因為巨集的展開時間在編譯時，所以巨集可以用來自動生成trait的實作，而函數則無法。

### 宣告式巨集

這算是最常寫的巨集，會有與`match`的對應關係類似的寫法。舉例來說，`vec!`這個巨集的定義可以被簡化成如此：

``` rust
#[macro_export]
macro_rules! vec {
    ( $( $x: expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

詳細的語法請見[參考](https://doc.rust-lang.org/reference/macros-by-example.html)。首先，最外層的小括號代表對應的模式分支，對應關係十分類似`match`，但最後是以分號結尾。再來的`$`十分類似shell script的`$`，而小括號再次代表了一個對應關係，其中的`$x: expr`代表任何Rust的運算式會在巨集內被當成`$x`看待。括號後接的`,`則代表這是選擇性的，而更後面的`*`是代表前面的模式會被重複零次以上。
未來的Rust會出現第二種宣告式巨集，以解決目前的`macro_rules!`在部分情況下發生的問題，屆時舊的會不推薦撰寫。不過對巨集使用來說，這兩種應該不會有所區別。

### 程序式巨集

這是巨集的第二種型態，寫法與函數比較類似。這種巨集會把一些程式碼當作輸入，處理這些程式碼，再輸出一些程式碼。
要建立程序式巨集時，必須要在有特殊crate型態的crate中定義這些巨集，這是Rust還沒整合的部分，未來可能會消除這個限制。

``` rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {}
```

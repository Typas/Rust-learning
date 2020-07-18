# Notes

這章講物件導向。

## 物件導向基本概念

SOLID原則：
- Single-responsibility：物件應只具單一功能。
- Open-closed：應該對擴充開放，但對修改封閉。
- Liskov substitution：物件可被其子類所替換，而不改變正確性。
- Interface segregation：多個特定介面優於一個泛用介面。
- Dependency inversion：依賴於抽象而不是實例。

## 常用的組合

Rust中常以`struct`與`enum`結合`impl`來提供「物件」，這種組合可以提供GoF所定義的物件所需的功能。

## Encapsulation 封裝

Rust可使用`pub`來提供對外的API。如下的`add`、`remove`與`average`是外面唯三能存取資料的方法。這能保證外部不會意外修改到內部的值。

``` rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

## Inheritance 繼承

Rust沒有繼承。會使用繼承的主因是程式碼的重複使用，而在rust中可以用trait來解決這個問題。

## Polymorphism 多型

Rust使用trait達成多型。

## Trait Object

Struct中以`Box<dyn Foo>`來代表一個trait object，所有具有`Foo` trait的物件都可以被放入。
以下這段程式碼中，`Screen`存了一個有`Draw` trait的物件陣列，並使用了`draw()`這個函式。

``` rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

### 物件安全與限制

一個trait要達成物件安全，必須達到兩個條件：
- 回傳型態不能是`Self`
- 沒有泛型參數

舉例來說，`Clone`這個trait就不是物件安全，因為它會回傳`Self`。
所有的trait object都必須不能使用到非物件安全的trait。

### 多重trait

目前仍然無法直接使用`+`，需要一些workaround。[這裡](https://github.com/rust-lang/rfcs/issues/2035)有相關討論。

## 物件導向設計模式

有一種設計模式是state pattern狀態模式。在rust中有兩種解法，一種是在結構使用trait object表示狀態，另一種是使用不同型態與其函式回傳值。

使用trait object的樣式：
``` rust
struct Foo {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Foo {
    pub fn new() -> Foo {
        Foo {
            state: Some(Bow::new(Initial {})),
            content: String::new(),
        }
    }
}

trait State {
    fn to_second(self: Box<Self>) -> Box<dyn State>;
    fn to_final(self: Box<Self>) -> Box<dyn State>;
}

struct Initial {}

impl State for Initial {
    fn to_second(self: Box<Self>) -> Box<dyn State> {
        Box::new(Second {})
    }

    fn to_final(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct Second {}

impl State for Second {
    fn to_second(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn to_final(self: Box<Self>) -> Box<dyn State> {
        Box::new(Final {})
    }
}

struct Final {}

impl State for Final {
    fn to_second(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn to_final(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

使用不同型態的樣式：
```rust
pub struct Foo {
    content: String,
}

pub struct InitialFoo {
    content: String,
}

pub struct SecondFoo {
    content: String,
}

impl Foo {
    pub fn new() -> InitialFoo {
        InitialFoo {
            content: String::new();
        }
    }
}

impl InitialFoo {
    pub fn to_second(self) -> SecondFoo {
        SecondFoo {
            content: self.content,
        }
    }
}

impl SecondFoo {
    pub fn to_last(self) -> Foo {
        Foo {
            content: self.content,
        }
    }
}
```

使用trait object的形式比較容易擴充，但也無法讓特定狀態的函式可見度降低，僅能以輸出預設值阻擋。使用不同型態的形式可使無用的函式量降到最少，但也較難擴充，且會需要使用shadowing。

# Notes

這章主要講泛型處理。
泛型最大的用途是抽象化與一般化程式碼，降低重工。

## 泛用資料型別

### 用法
#### 函式
與C++的template相同，以`<>`表示泛型。

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0]; // need trait implementation of std::Copy
    
    for &item in list.iter() {
        if item > largest { // need implementation of std::cmp::PartialOrd
            largest = item;
        }
    }
    
    largest
}
```

#### 結構
多個泛型間以`,`間隔，如函式參數。

``` rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

#### 列舉
列舉中以`()`包住型別。

``` rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

#### method
要在`impl`後面就加上`<>`表示泛型實作。

``` rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

如果只實作特定型別，則`impl`後面不需加上`<>`

``` rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

`impl`後的泛型是指結構中既有的泛型，而非method所用的泛型。

``` rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point <T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```

### 效能
在編譯期就會把所有泛型都編成實體型別，所以執行效率不會有差。

## 特性
泛型是型別的抽象，而trait可說是method的抽象。

``` rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

### 特性實作
只要結構有包含某個特性，則這個特性的所有函式都必須被實作出來。
語法為`impl`與`for`的結合如下：

``` rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

實作的限制為只能實作crate內的特性，而不能實作外部特性。這個限制被稱為程式的「連貫性」，或稱為「孤兒法則」。限制能確保沒有重複實作的可能性發生。

### 預設的實作
trait是能有預設的實作的，而特定結構裡對該trait的實作都會蓋過其預設的實作，這跟C++的行為十分相近。
而預設的實作可以用同個trait裡面沒有預設實作的method。

``` rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

### 特性作為參數
以特性為參數時，實際傳入的仍是有實作特性的結構的實例，如上面的`Tweet`。
這語法能限制範圍在特性的method裡而不失泛用性。

``` rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

#### 特性綁定語法
`impl Trait`這個語法本身是一個語法糖，原型被稱為 *trait bound* ，長這樣：

``` rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

跟C++有八成像。

但是在有兩個以上的trait bound出現時，`impl Trait`預設為不同的泛型。
若需要強制全部為同一泛型，仍需要使用此語法：

``` rust
pub fn notify<T: Summary>(item1: T, item2: T) {}
```

#### 多個特性的綁定
用`+`來連接特性。

``` rust
pub fn notify(item: impl Summary + Display) {}
```

``` rust
pub fn notify<T: Summary + Display>(item: T) {}
```

#### 特性綁定的`where`從屬式
這也是語法糖，能讓多個特性綁定看起來簡潔點：

``` rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {}
```

以上的語法可以用`where`改成

``` rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 回傳型別為特性實作
同樣是為了維持泛用性而做出來的，參數型別是`impl Trait`，會回傳的則為其中一種型別的實例：

``` rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

要注意的是回傳值只能是同一種型別的實例，否則會因編譯器的限制而報錯。

### 用特性綁定來選擇性實作method

`impl`後面的`<>`裡面也能用特性綁定，未實作特性則不會實作此method。如下例的`cmp_display`

``` rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> self {
        self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 生命期

### Preventing Dangling Reference with Lifetimes
以下的code因為有生命期問題，所以編不過。

``` rust
{
    let r;
    
    {
        let x = 5;
        r = &x;
    } // x drops here
    
    println!("r: {}", r);
}
```
 
### The Borrow Checker
借用的檢查規則為，借用者的生命期必須被包含在被借用者的生命期之內。

### 函式中的泛用生命期
參數中，參考是「借」來的，所以若回傳的值有包含借用，則必須標記其生命期為「泛用」的生命期。

#### 生命期標記語法
在`'`後加上一個「名字」，與泛型類似。常見用法是`'a`，有保留字`'static`。

``` rust
&i32        // 參考
&'a i32     // 有顯式生命期的參考
&'a mut i32 // 有顯式生命期的可變參考
```

#### 函式簽名中的生命期標記
跟泛型一樣，需要包含在`<>`之中。

``` rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
### 結構內的生命期標記
結構中如果有參考，一樣需要標記其生命期為泛用的。

``` rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```
### 免標記情況
在少數編譯器推導得出來的情況下，生命期標記語法是可省略的。
像是回傳參數的slice：

``` rust
fn first_word<'a>(s: &'a str) -> &'a str {}
```

可以被省略為

``` rust
fn first_word(s: &str) -> &str {}
```

有幾個可省略規則：
1. 參數的參考之間的生命期不互相干擾。
2. 參數只有一個參考，則其生命期為回傳參考的生命期。
3. 參數中其中一個是`&self`或`&mut self`，代表這是method。

### method中的生命期標記
若結構內有生命期標記，則每個method都需要加上泛型的參數。

``` rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part{&self, announcement: &str} -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

### 靜態生命期
標註為`'static`的參考，其生命期為整個程式的執行期。
常數字串都能做`'static`參考。但因為很貴，所以要謹慎考慮使用時機。


### 泛型參數大亂鬥
生命期的標記語法上可以看成一種泛型，所以也能做特性綁定。

``` rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```


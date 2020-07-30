# Notes

這章講模式。

## 可以使用模式的地方

### `match`分支

概念上與C的`switch`...`case`相同，而有更大的彈性空間。其中的`_`對應到C裡的`default`，必須寫完所有對應關係。

``` rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

### `if let`條件運算式

算是上面`match`的簡化版本，在只需處理特定對應情況時好用，能與`else`結合。需注意編譯器不會檢查是否對應所有情況。

``` rust
if let PATTERN = EXPRESSION {
   STATEMENTS
}
```

### `while let`條件式迴圈

`if let`的迴圈版本。

### `for`迴圈

常用於解構`.iter().enumerate()`所產生的tuple。
``` rust
for PATTERN in EXPRESSION {
    STATEMENTS
}
```

### `let`敘述

可用於宣告多個變數時使用。
``` rust
let PATTERN = EXPRESSION;
```

請注意，賦值則無法使用此方法，也無法連續賦值，只能分多行寫。

### 函數參數

參數可用模式對應的方式解構。
``` rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

## 模式的Refutability

Rust中的refutable指的是模式可以接受對應不到值的情形，而irrefutable則是模式必須對應到一個值。

### Irrefutable對應
    - `for`迴圈
    - `let`敘述
    - 函數參數

### Refutable對應
    - `match`分支
    - `if let`
    - `while let`

## 模式語法

### 對應文字值

文字值，在rust中常以`const`或是一簡單定值出現，是一個在編譯時就能確定的值。

``` rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    _ => println!("others"),
}
```

### 對應命名變數

模式中的變數名，可以看作是一個新的變數宣告，其生命期只在對應關係的大括號內。

``` rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y),
    _ => println!("Default case, x = {:?}", x),
}

println!("After match: x = {:?}, y = {:?}", x, y);
```

### 多模式

在`match`運算式中，可以用`|`語法把多種模式串接成一種。

``` rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("others"),
}
```

### 對應範圍

目前只能使用`..=`這個包含結束點的範圍，`..`這個不包含結束點的範圍目前仍未支援，請見[討論](https://github.com/rust-lang/rust/issues/37854)。

``` rust
let x = 'c';

match x {
    'A'..='Z' => println!("uppercase"),
    'a'..='z' => println!("lowercase"),
    _ => println!("something else"),
}
```

### 解構變數

#### 解構結構

結構中的欄位名可直接作為新變數使用，其他變數名需指定欄位。

``` rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y: a } = p;
    assert_eq!(0, x);
    assert_eq!(7, a);
}
```

#### 解構列舉

最常見的是解構`Result`或`Option`，但可以有更複雜的解構方式，見下面巢狀解構。

#### 巢狀解構

如果結構或列舉內包含了另一個結構或列舉，則可以進行巢狀解構：

``` rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::Move { x, y } => println!(
            "Move in the x direction {} and in the y direction {}",
            x, y
        ),
        Message::ChangeColor(Color::Rgb(r, g, b)) => println!(
            "Change the color to red {}, green {}, and blue {}",
            r, g, b
        ),
        Message::ChangeColor(Color::Hsv(h, s, v)) => println!(
            "Change the color to hue {}, saturation {}, and value {}",
            h, s, v
        ),
        _ => (),
    }
}
```

### 忽略模式中的值

#### 完全忽略

如果連變數綁定都不想要的話，可以用`_`來忽略整個模式中的特定值。

#### 部分忽略

如果只想做部分對應的話，想忽略的地方可以用`_`來忽略。

``` rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => println!(
        "Some numbers, {}, {}, {}",
        first, third, fifth
    ),
    _ => (),
}
```

還有一種部分忽略是用`..`達成，作用是略過所有沒被提到的值。但因這不是一對一關係，tuple中只能用一次來省略開頭、中間或最後。

``` rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

#### 忽略未使用的變數

在變數名前加上`_`即可，但這與單一`_`的不同點在這仍然是一個合法變數，可能會拿走所有權。

### 以Match Guards處理複雜情況

這邊所謂的match guard是在模式後加上的`if`條件，能做到更彈性的處理。

``` rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than 5: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

請注意match guard是掛在「整個」模式後面，而不是分開處理。如下的程式碼，`if`的判斷不只在`6`發生，而是在整個`4 | 5 | 6`發生。

``` rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```

### `@` 運算子

這個運算子可以讓我們在檢查模式對應的同時新增一個變數。

``` rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello {
        id: 10..=12
    } => println!("Found an id in another range"),
    Message::Hello { id } => println!("Found some other id: {}", id),
}
```

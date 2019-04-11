# 3.3 How Functions Work 函式的執行

## Functions 函式
在Rust程式碼中到處都是。用了`cargo new`新建專案之後能在`main.rs`裡看到這語言最重要的函式：`main`函式，也是大多程式的進入點。你也看到了宣告新函式用的關鍵字`fn`。

Rust程式碼通常用 *snake case* 方式來命名函式與變數。*snake case* 中所有的字母都是小寫，用底線來分隔單字。這裡有個範例：
``` rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

Rust中函式宣告以`fn`開始，在函式名稱後寫上參數。大括號代表函式從何開始與結束。

我們可以用函式名稱與參數來呼叫定義過的函式。因為在這程式中`another_function`己經被定義了，所以它能在`main`函式裡面被呼叫。請記得我們是在`main`後面才定義`another_function`，其實它也能在`main`之前定義。Rust不注重你在哪裡定義函式，只要它存在於一個找得到的地方。

### Function Parameters 函式的參數
函式也可以有包含「參數」的定義，參數是一些包含在函式簽名中的特殊變數。當一個函式有參數時，你可以提供定值給參數。技術上，這些定值被稱為「引數」，但是通常這兩個名詞會被混著用。

將上面的範例改動成有參數的函式：
``` rust
fn main() {
    another_function(5);
}

fn another_function(x : i32) {
    println!("The value of x is: {}", x);
}
```
執行之後你會看到
```
The value of x is: 5
```
函式`another_function`的定義有一個參數`x`。`x`的型態被指定成`i32`。當`5`傳給`another_function`後，`println!`這個巨集將`5`放到了格式字串的大括號中。

在函式的宣告中，每個參數的型態都需要指定。這是Rust的故意設計：函式定義中寫死型態，代表編譯器不用從其他程式碼中推測。

如果想寫一個多參數的函式，要用逗號`,`分開參數。例如：
``` rust
fn main() {
    another_function(5, 6)
}

fn another_function(x : i32, y : i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```
這個例子創造了一個有兩個參數的函式，兩個都是`i32`型態。這函式印出了兩個參數的值。請記得函式的參數不需要全都同一個型態，這只是範例中的巧合。

### Function Bodies Contain Statements and Expressions 有敘述與運算式的函式
函式的內容是由一串的敘述組成，可能以運算式結尾。目前我們講到了沒有以運算式結尾的程式，但是你已經把運算式當成是敘述的一部分。因為Rust是一個基於運算式的語言，所以要理解敘述與運算式的區分很重要。

我們已經用過了敘述與運算式。敘述(statements)是一串執行的指令，不會回傳任何數值。運算式(expressions)求得結果的數值。

舉幾個例子：
用`let`創造一個變數，並給它一個值，這是一個敘述。
``` rust
let y = 6;
```
函式的定義也是一個敘述。

敘述不會回傳值，所以你不能將一個`let`敘述指定給另一個變數，像是下面的程式碼：
``` rust
let x = (let y = 6);
```
這在編譯時就會報錯了。因為`let y = 6`敘述並不會成為一個數值，所以`x`不能被綁到任何值上。在一些語言中，你能寫`x = y = 6`而且`x`跟`y`的值都會是`6`，但這在Rust中不可行。

運算式會求得一個結果並組成你未來寫的Rust程式碼。一個簡單的數學計算，像是`5 + 6`，是一個能求得數值`11`的運算式。運算式可以是敘述的一部分，例如上面`let y = 6`的`6`就是一個運算式。一個函式呼叫，一個巨集呼叫也都是一個運算式。一個用`{}`組成的新區塊範圍也是一個運算式。例如：
``` rust
let x = 5;

let y = {
    let x = 3;
    x + 1
};

println!("The value of y is: {}", y);
```
這是一個能求出數值`4`的區塊。所以可以用`let`將`y`與這個區塊求出的值綁在一起。請記得`x + 1`這行最後面沒有任何的`;`。如果加了`;`就會把運算式轉成了敘述，也就不會回傳一個值。當你探究函式的回傳值與運算式時也要銘記在心。

### Functions with Return Values 有回傳值的函式
函式可以回傳一個值給呼叫它的程式碼。我們不命名回傳值，但是我們會在一個箭頭`->`後宣告它們的型態。Rust中，可以用最後的運算式代表函式的回傳值，也可以在這之前用關鍵字`return`來回傳。不過大多的函式會隱性回傳最後的運算式。舉個例子：
``` rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    
    println!("The value of x is: {}", x);
}
```
函式`five`裡面沒有函式呼叫，巨集，甚至是`let`敘述——只有一個數字`5`。這是個在Rust中完全有效的函式。這函式也有指定回傳的型態`i32`。

再來看一個例子：
``` rust
fn main() {
    let x = plus_one(5);
    
    println!("The value of x is: {}", x);
}

fn plus_one(x : i32) -> i32 {
    x + 1
}
```
執行後會印出`The value of x is: 6`。但如果把`x + 1`那行後面加了個分號時，會讓它從一個運算式變成一個敘述，而得到一個編譯錯誤`mismatched types`。函式`plus_one`的定義顯示它應該回傳一個`i32`的值，但是敘述不會回傳任何值，以空數據組`()`表示回傳值為空。回傳的值與定義互相矛盾，從而得到一個錯誤。

# Links
- Previous section - 3.2 [Data Types](./type.md)
- Next section - 3.4 [Comments](./comment.md)
- back to [category](./../README.md)

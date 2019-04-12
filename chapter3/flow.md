# 3.5 Control Flow 流程控制

## `if` Expressions `if`運算式
`if`運算式能讓你根據情況執行不同分支的程式碼。你要提供一個條件，再敘述「條件達成時，執行這段程式，否則不要執行。」例如：
``` rust
let number = 3;

if number < 5 {
    println!("condition was true");
} else {
    println!("condition was false");
}
```
所有的`if`運算式以關鍵字`if`開始，後面跟著一個條件。在這個例子中，條件是檢查變數`number`的值是否小於`5`。我們想在條件符合時執行的程式區段應該要緊臨條件後，並以大括號`{}`包起來。這種與`if`運算式相關的程式區段有時被稱為粗枝(arm)。

我們也可以同時包含一個`else`運算式，與其相關的程式區段則用來在條件不符合時執行。如果條件不符又沒有一個`else`運算式時，程式會跳過那個`if`相關的程式區段，並執行之後的程式碼。

有件值得記住的事，條件的型態必須是`bool`。如果條件不是一個布林值，會得到編譯上的錯誤。Rust不像一些程式語言，會把非布林值轉成布林值。

### Handling Multiple Conditions with `else if` 以`else if`處理多重條件
你可以把`if`與`else`組合成一個`else if`運算式。例如：
``` rust
let number = 6;
if number % 4 == 0 {
    println!("number is divisible by 4");
} else if number % 3 == 0 {
    println!("number is divisible by 3");
} else if number % 2 == 0 {
    println!("number is divisible by 2");
} else {
    println!("number is not divisible by 4, 3, or 2");
}
```
當程式執行時，它會檢查每個`if`運算式是否符合條件，再執行第一個符合條件的程式區段。Rust中，當找到第一個符合的條件後，剩下的連檢查都不會做。

用太多的`else if`會讓程式碼變得混亂，你可能需要重構你的程式碼。第六章寫了Rust一個強力的分支建立方式叫`match`來應對這種情況。

### Using `if` in a `let` Statement 在`let`敘述中使用`if`
因為`if`是一個運算式，所以可以用來放在`let`敘述的右邊，例如：
``` rust
let condition = true;
let number = if condition {
    5
} else {
    6
}

println!("The value of number is: {}", number);
```
變數`number`會綁定右邊運算式的結果。執行後會得到：
```
The value of number is: 5
```
記得這段程式碼會求出其中的最後運算式，而且數字本身也是一個運算式。在這個例子中，整個`if`運算式的值取決於哪段程式碼會被執行。這代表`if`的不同粗枝都可能成為結果，所以必須是同一個型態。在上面的程式碼中，兩個粗枝的型態都是`i32`的整數。如果型態不同，會出現編譯錯誤。

## Repetition with Loops 以迴圈來重複
執行多次同一段程式碼常常很有用。Rust為此準備了多種迴圈(loop)。一個迴圈會執行裡面的程式碼到尾端，再立刻回到開頭。Rust有三種迴圈：`loop`、`while`與`for`。

### Repeating Code with `loop` 以`loop`重複程式碼
關鍵字`loop`告訴Rust去一直重複不斷執行一個程式碼區段，直到你清楚的告知它停下。例如：
``` rust
loop {
    println!("again!");
}
```
當執行這段程式時，會看到`again!`被瘋狂印出來，直到手動中斷。大多是用ctrl-c來中止一個在無限迴圈中的程式。Rust提供了一個關鍵字`break`，用在迴圈中能夠中止迴圈的執行。

### Returning Values From Loops 迴圈的回傳值
`loop`的一個用途是重複一個可能會失敗的操作，像是檢查一個線程是否完成了它的工作。不過，你可能會想把結果傳給後面的程式碼。如果你在中止迴圈的`break`後加了一個運算式，它會回傳到迴圈外面成為迴圈的回傳值。舉例來說：
``` rust
let mut counter = 0;

let result = loop {
    counter += 1;
    
    if counter == 10 {
        break counter * 2;
    }
};

println!("The result is {}", result);
```
在迴圈前，先宣告了一個變數`counter`並初始化為零。然後宣告一個變數`result`來保存迴圈回傳的值。迴圈的每個迭代(iteration)都將`counter`加了一，再檢查是否等於十。當等於的時候，我們用關鍵字`break`來跳出迴圈，`break`後面的運算式`counter * 2`則是回傳值。迴圈後我們放了一個分號來結束`let`敘述，將回傳值指定給`result`。最後印出`result`的值，在這個例子中是二十。

### Conditional Loops with `while` 以`while`重複條件性程式區段
以一個條件來決定繼續與否的迴圈常常很有用。當條件成立時，迴圈繼續執行；在條件不再成立時，迴圈中止。這可以用`loop`、`if`、`else`與`break`的組合達成。但是這形式常見到Rust有個內建的語法叫`while`迴圈專門執行這種組合。下面是一個`while`的例子：程式執行了三次，每次都會倒數，在迴圈後印出一個訊息後結束。
``` rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```
這減少了很多用`loop`、`if`、`else`與`break`組成的巢狀結構，而且更清楚。當條件成立時繼續，否則離開迴圈。

### Looping Through a Collection with `for` 用`for`在集合重複執行
你可以用`while`在一個集合上執行迴圈，例如在一個陣列上。舉例來說：
``` rust
let a = [10, 20, 30, 40, 50];
let mut index = 0;
while index < 5 {
    println!("the value is: {}", a[index]);
    
    index = index + 1;
}
```
這段程式碼將陣列中的元素依序列出。從索引值`0`開始，重複迴圈直到陣列的最後一個索引值。執行這段程式會看到陣列的五個值都被依序列出。雖然`index`在某個時候達到了`5`，但是在迴圈執行第六次前就停止執行迴圈了。不過這方法很容易出錯，當陣列長度不對時就會造成程式panic。同時，因為編譯器會加入檢查邊界用的程式碼，所以也會很慢。

有另一種簡潔的方式，你可以用`for`迴圈來執行一段針對每個集合中的元素的程式碼。下面是一個例子：
``` rust
let a = [10, 20, 30, 40, 50];

for element in a.iter() {
    println!("the value is: {}", element);
}
```
當執行這段程式時，得到的結果與上面用`while`做的一樣。更重要的是，我們增加了程式碼的安全性，也消滅了產出bug的機會。在上面`while`的程式碼中，如果你將一個元素給移除掉之後忘記更新條件的範圍，程式會panic。用了`for`迴圈，這是個不用擔心的問題。`for`迴圈的簡潔與安全讓它成為了Rust裡最常用的迴圈。就算是想跑確定次數的迴圈，也能用`Range`來達成，`Range`由標準函式庫提供，它能產生出一個數字到另一個數字的所有數字。上面的倒數可以用`for`迴圈、`Range`與一個還沒提的方法`rev`，用來反轉範圍：
``` rust
for number in (1..4).rev() {
    println!("{}!", number);
}
println!("LIFTOFF!!!");
```
注意範圍是以兩個點連接，而且並不會碰到上界。也就是說，`a..b`的實際範圍是從`a`到`b - 1`。

## Summary 總結
你做到了！這是一個很有分量的章節：你學到了有關變數、純量與複合的資料型態、函式、註解、`if`運算式與迴圈。如果你想要練習這個章節講到的概念，試著建構幾個能做到下面要求的程式：
- 轉換華氏與攝氏溫度。
- 產生第n個費氏數。
- 印出聖誕歌The Twelve Days of Christmas的歌詞。


# Links
- 前一章節 - 3.4 [Comments](./comment.md)
- 回到[目錄](./../README.md)

## References
- *match* - 6.2 [The match Control Flow Operator]

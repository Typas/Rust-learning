# 3.2 Date Types 資料型態

## Scalar Types 純量型態
一個純量的資料型態代表的是一個值。Rust有四種主要的純量型態：整數、浮點數、布林數與字元。

### Integer Types 整數型態
整數是一個沒有任何小數的數值。在第二章用的`u32`型態，代表的是這是一個無正負號且使用了32位元空間的整數。表格3.1列出了Rust的內建整數型態。

> 表格3.1: 內建整數型態

| Length  | Signed  | Unsigned |
|---------|---------|----------|
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| arch    | `isize` | `usize`  |

每個長度都有兩種宣告，帶正負號的與不帶的。*Signed* 與 *Unsigned* 取決於這個變數有沒有可能會是負的，也就是說，當這個變數必定不會是負值時，可以使用無正負號的整數。每個使用了 n 個位元的有號整數表示數值的範圍為從 -2^(n-1) 到 2^(n-1) - 1 ；無號整數表示的範圍是自 0 到 2^n - 1 。

最後一行的`isize`與`usize`代表變數的長度取決於電腦的架構，在一台32位元電腦上執行時變數的長度會是32-bit。

#### 整數的表示法
你可以以表格3.2中的任何形式來表達整數。除了Byte以外的形式都可以加上型態當成後綴，例如`57u8`，而底線`_`可以作為分隔符號使用，例如`1_000`與`1000`表示的是同一個數值。

> 表格3.2: 整數表示形式

| 表示法   | Number literals | Example       |
|----------|-----------------|---------------|
| 十進制   | Decimal         | `98_222`      |
| 十六進制 | Hex             | `0xff`        |
| 八進制   | Octal           | `0o77`        |
| 二進制   | Binary          | `0b1111_0000` |
| 位元組   | Byte(`u8` only) | `b'A'`        |

如果不知道該選擇哪種，交給編譯器就好。一般來說，`i32`在Rust中會是最快的，就算是64位元的系統也是。

#### Integer Overflow 整數溢位
假設你有一個`u8`的變數，這可以表示`0`到`255`之間的任何整數。當你試著讓它變成`256`時會怎樣？這被稱為「整數溢位」，Rust對這行為有些有趣的規則。

當在debug模式編譯時，Rust會檢查這情況，而且會造成你的程式 *panic* ，這代表程式執行錯誤並退出。

當在release模式時，Rust不會檢查，而會做二補數包裝(two's complement wrapping)。簡言之，`256`會變`0`，`257`會變`1`等等。通常溢位會被當成錯誤，就算是「二補數包裝」後也一樣。如果確定要使用這方法，標準函式庫有一個型態`Wrapping`可以標示這行為而不會被認定為錯誤。

### Floating-Point Types 浮點數型態
Rust有兩種浮點數的基本型態，`f32`與`f64`，分別佔用了32 bits與64 bits。預設的浮點數型態為`f64`。

以下是浮點數的宣告例子：
``` rust
let x = 2.0;      // f64
let y : f32 = 3.0; // f32
```

浮點數的基本型態是根據IEEE-754所訂。`f32`代表單精度浮點數，`f64`代表雙精度浮點數。

### Numberic Operations 數值運算
Rust分別支援整數與浮點數的基本運算：加法、減法、乘法、除法與求餘。
以下是數值運算的例子：
``` rust
// addition
let sum = 5 + 10;

// subtraction
let difference = 95.5 - 4.3;

// multiplication
let product = 4 * 30;

// division
let quotient = 56.7 / 32.2;

// remainder
let remainder = 43 % 5;
```
每個表達式都用了一個運算子求出一個值，再綁到一個變數上。附錄B包含了一個Rust提供的所有運算子的表格。

### The Boolean Type 布林型態
一如其他程式語言，Rust中的布林值有兩種：`true`與`false`。在Rust中以`bool`指定布林型態。例如：
``` rust
let t = true;
let f : bool = false; // with explicit type annotation
```
布林值的主要用途是條件判斷上，像是`if`敘述。在Control Flow章節會講到`if`是怎麼在Rust裡運作的。

### The Character Type 字元型態
Rust的`char`型態是整個語言中最基本的文字型態，下面的程式碼示範如何使用這種型態：
``` rust
let c = 'z';
let z = 'Z';
let ch_heart = '心';
```
Rust的`char`型態代表了一個Unicode Scalar Value，比ASCII能表示的文字還多。不過字元並不是Unicode有的概念，所以Rust中的`char`不代表其他程式語言中的字元。這在第八章的String中會再討論到。

## Compound Types 複合型態
複合型態可以將多個變數組成一個型態。Rust有兩個原生的複合型態：tuple與array。

### The Tuple Type 數值組型態
數值組是一種將多種型態的數值組合進一個型態的常見方式。數值組有固定的長度：當宣告後便不可以再增加或減少內容物。

數值組的宣告方式是在一個小括號內寫入多個數值，以逗點分隔位置。數值組的每個位置都對應一個型態，而不同位置的型態不需要相同。如下例：
``` rust
let tup : (i32, f64, u8) = (500, 6.4, 1);
```
變數`tup`綁在了整個數值組上，因為數值組被認為是個數值的複合型態。要取得其中的一個數值，可以用類型比對來解構數值組，例如：
``` rust
let tup = (500, 6.4, 1);

let (x, y, z) = tup;

println!("The value of y is: {}", y);
```
這段程式先將數值組綁在變數`tup`上，再用組合三個類型的`let`將`tup`分別給了三個變數`x`、`y`和`z`。這稱為**解構**，因為它將一個數值組分解成三個部分。最後，程式把`y`的值印出。

除了用類型比對解構以外，還能直接用`.`加索引值來取得數值組的單一元素的值。例如：
``` rust
let x : (i32, f64, u8) = (500, 6.4, 1);
let five_hundread = x.0;
let one = x.2;
```
這個程式先創造了一個數值組`x`，再分別以數值組的索引值來製造新的變數。與大多的程式語言相同，索引值從0開始。

### The Array Type 陣列型態
另一個將多個數值整合進一個型態的方法叫作**陣列**。與數值組不同，陣列的每一個元素都必須是同一個型態。Rust的陣列是固定長度的，與數值組相同。

Rust中陣列的宣告方式是在一個中括號內寫入數值，以逗點隔開。例如：
``` rust
let a = [1, 2, 3, 4, 5];
```
當想把資料放在stack而不是heap中，或是確定數量固定時，陣列很好用。陣列與第八章會提到的vector很像，但是vector可以伸縮長度。當你不知道要用陣列還是vector時，vector是你的好朋友。

一個會使用陣列的好例子是一年中的所有月份，這幾乎不會變，所以你可以放心使用陣列：
``` rust
let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
```
陣列還有個有趣的型態，長得像`[型態;數量]`。舉例來說：
``` rust
let a : [i32 ; 5] = [1, 2, 3, 4, 5];
```

#### Accessing Array Elements 存取陣列元素
陣列是一個在stack上的單一記憶體區塊。你能以索引值存取陣列元素，例如：
``` rust
let a = [1, 2, 3, 4, 5];
let first = a[0];
let second = a[1];
```
在這個例子中，變數`first`會綁在數值`1`上面，因為這是陣列上索引值`[0]`所代表的值。變數`second`則會得到數值`2`。

#### Invalid Array Element Access 陣列元素的無效存取
當試著用索引值存取陣列範圍外的元素會如何？把上面的範例改一下之後，
``` rust
let a = [1, 2, 3, 4, 5];
let index = 10;

let element = a[index];

println!("The value of element is: {}", element);
```
程式碼可以編譯成功，但是執行時會報錯退出，產生runtime error。當用索引值來存取一個元素時，Rust會檢查索引值是否超過了陣列的長度。當索引值超過長度時，Rust will panic.

這是第一個Rust安全原則的例子。在許多低階語言如C，這種檢查並不存在，當你提供了一個錯誤的索引值時，不應被存取的記憶體位置也能存取得到。

# Links
- 前一章節 - 3.1 [Variables and Mutability](./variable.md)
- 下一章節 - 3.3 [How Functions Work](./function.md)
- 回到[目錄](./../README.md)

## References
- *if* - 3.5 [Control Flow](./flow.md)
- 8.1 [Vectors] 尚未完工
- 8.2 [Strings] 尚未完工
- *panic* - 9.1 [Unrecoverable Errors with panic!] 尚未完工
- 21.2 [Appendix B] 尚未完工

# Extra notes
以下`<argument>`代表必須的參數，`[optional]`代表選填的參數，`, ...`表示前面參數的列舉。
`expr`代表運算式，`...`代表敘述，`type`代表型態，`value`代表值，`name`代表變數名稱。
Rust宣告可以無初始值，但同時須指定資料型態。

## 變數
- 宣告：`let [mut] <name> [: type] = <expr> ;`或是`let [mut] <name> : <type> ;`。
``` rust
let a:i32;
let mut b = 60;
```

## 常數
- 宣告：`const <name> : <type> = <expr> ;`。
``` rust
const pi : f64 = 3.14159;
```

## Tuple
- 宣告：`let <name> [: (type1, type2, ...)] = <(var1, var2, ...)> ;`或`let <name> : <(type1, type2, ...)> ;`。
- 解構： `let (<name1>, <name2>, ...) = <tuple_name> ;`。其中的型態可以底線`_`代表pattern matching的任意型態，由編譯器推導出；解構的變數也能以`_`來取代不需要的變數。
- 索引： `<name>.n`，其中`n`代表index value，從0開始。
``` rust
let tup = (20, 30.6, '早');
let top : (i32, u64, f32);
let (x, y, _) = tup;
let z = tup.2;
```

## 陣列
- 宣告：`let <name> [: [type ; number]] = <[value1, value2, ...]> ;` 或是`let <name> : [<type> ; <number>]`，其中`number`代表元素個數。
- 存取： `<name>[n]`，其中`n`代表index value。

## 函式
- 宣告：`fn <name>([name1 : type1, name2 : type2, ...]) [-> type] {...}`
- 回傳值有`return`接的運算式與最後的運算式兩種。

## 註解
只有以`//`開始的單行註解。

## 條件判斷
- 形式：`if <expr> {...} [else <expr> {...}]`
- 以`else if`處理多重條件
- `if`本身可以是個運算式。

## 迴圈
- 形式有三種：`loop`、`while`與`for`。
- `loop`可以是個運算式，接在`break`後，`;`前的運算式是`loop`的回傳值。
- 形式：
  - `loop {...}`，以`break`離開或陷入無窮迴圈中
  - `while condition {...}`
  - `for <name> in <Collection> {...}`

回到[目錄](./../README.md)

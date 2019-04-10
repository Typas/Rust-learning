# Extra notes
以下`<argument>`代表必須的參數，`[optional]`代表選填的參數，`, ...`表示前面參數的列舉。
Rust宣告可以無初始值，但同時須指定資料型態。

## 變數
宣告形式：`let [mut] <name> [: type] = <expression> ;`或是`let [mut] <name> : <type> ;`。

## 常數
宣告形式：`const <name> : <type> = <expression> ;`。

## Tuple
宣告形式：`let <name> [: (type1, type2, ...)] = <(var1, var2, ...)> ;`或`let <name> : <(type1, type2, ...)> ;`。
解構： `let (<var1>, <var2>, ...) = <tuple_name> ;`。其中的型態可以底線`_`代表pattern matching的任意型態，由編譯器推導出；解構的變數也能以`_`來取代不需要的變數。
索引： `<name>.n`，其中`n`代表index value。

## 陣列
宣告形式：`let <name> [: [type ; number]] = <[value1, value2, ...]> ;` 或是`let <name> : <[type ; number]>`
存取： `<name>[n]`，其中`n`代表index value。

back to [category](./../README.md)

# Notes

## 結構的定義
- 以關鍵字`struct`開始，大括號內定義每個欄位。
- 欄位形式：`<name> : <type>,`
- 最後一欄一樣需加`,`結尾
- 定義不需加`;`在最後

## 宣告結構變數
- 與一般宣告相同
- `=`後面接`<structname> { <name1> : <value1>, ... }`
- 也要以`;`表示語句完成

### 結構的更新語法
- 能在建立新變數時複製舊變數的部份欄位
- 以`..<variable>`複製未指定值的欄位，最後沒有`,`
``` rust
let var2 = Mystruct {
    field1: String::from("hello"),
    field2: 0,
    ..var1
}
```

### Tuple Struct
- 欄位沒有名稱
- 使用上與tuple相同
``` rust
struct Tup(i32, i32);

let var = Tup(1, 2);
```

### 結構的限制
- 每個欄位皆需要完整儲存在結構內
- 結構定義不能使用參照

## 結構的除錯資訊
- 檔頭需加上`#[derive(Debug)]`
- 能在`println!()`以`{:?}`輸出每個欄位的資訊
- `{:#?}`輸出的除錯資訊有更好的排版

## 方法的定義
- 與一般函式定義相同
- 需包含在`impl <structname> {}`的大括號之中
- 第一個參數必為`&self`
- 以`var.method()`呼叫
- `impl`可以有多個同時存在

## 相關函式
- 包含在`impl <structname> {}`的大括號之中
- 參數沒有`&self`
- 以`structname::asso_func()`呼叫

回到[目錄](./README.md)

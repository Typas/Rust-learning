# Notes

## 列舉的定義
- 以關鍵字`enum`開始，大括號內定義每個列舉
- 列舉形式：`<name>[(type1, type2...)]`
- 每個列舉視同一個`struct`
- 最後一欄一樣需加`,`作結尾
- 定義不需在最後加`;`

### 選項
- Rust以`Option<T>`來代替null
- 定義如下：
``` rust
enum Option<T> {
    Some(T),
    None,
}
```
- 使用方法如下：
``` rust
let some_n = Some(5);
let absent_n: Option<i32> = None;
```
- `Option<T>`與`T`型態不同，需要經過額外轉換，所以可以抓到null的情況。
- Rust用`None`來代替null

## 配對
- 使用：
``` rust
match var_enum {
    T::n1 => 1,
    T::n2 => 2,
    T::n3 => {exp1; value}
    _ => (),
}
```
- 在配對上，需要列出所有模式的配對。
- 對於只需處理一種的情況，`if let`更好用。

### 占位符
- 配對中所有非特殊情況可以用一個`_`代表所有未列出的情況。
- 對應的值可以用`()`代表空值

## `if let`
- 簡化流程控制
- 缺點是無法額外處理其他狀況
- 使用方法如下：
``` rust
if let match_value = variable {
    // do something;
} else {
    // do another;
}
```
- 與以下`match`語句相同：
``` rust
match variable {
    match_value => println!("do something"),
    _ => println!("do another"),
}
```

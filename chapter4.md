# Notes

## 記憶體配置
- 區塊以`{}`包覆
- 結束區塊時自動呼叫`drop`函式清理動態配置的記憶體

## `=` 對資料的操作
- 純配置在stack上的資料直接複製
- 配置在heap上的資料轉移所有權，舊變數無效

## 對變數進行深度複製
- 使用`clone`函式複製heap資料
``` rust
let s1 = String::from("hello");
let s2 = s1.clone();
```

## 函數
- 參數等同於使用`=`對資料的操作，轉移所有權到參數上
- 回傳會轉移所有權

### 借用
- 函數的參數使用參照，不轉移所有權
``` rust
fn calculate_length(s: &String) -> usize
{
  // calculation here
}
```
- 使用函數時一樣要寫上參照
``` rust
let s1 = String::from("hello");
let len = calculate_length(&s1);
```

#### 可變參照
- 函數的參數與使用時皆加上`mut`，傳入的變數必須為`mut`
``` rust
let mut s = String::from("hello");
change(&mut s);

fn change(some_string: &mut String) {}
```

### 多重參照
- 同時有多個不變參照可行
- 可變參照只能有一個，且不能有不變參照
- 可用區段做出生命期區別，避免資料競爭

## 切片型態
- 切片是一段資料的參照
- 語法為`&variable[start..end]`
- 開頭預設為整段資料的開頭(0)
- 結尾預設為整段資料的結尾
``` rust
let another = &s[..]; // equiv &s
```

### 字串切片
``` rust
let s = "Hello, world!";
```
- `s`的型態為`&str`，是字串切片的型態。
- 函數的參數型態為`&str`時，傳入`String`型態也能正常運作

### 其他型態的切片
``` rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
```
- `slice`的型態為`&[i32]`
- `a`的型態為`[i32]`

回到[目錄](./README.md)

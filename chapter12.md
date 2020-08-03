# Notes

這章講輸入輸出

## 接受程式參數

需要`std::env`。其中`args[0]`必為程式自身的路徑。`env::args()`在接到非法Unicode字元時會panic，如果有必要則以`env::args_os()`替代。
``` rust
use std::env;

let args: Vec<String> = env::args().collect();
```

## 讀取檔案
需要`std::fs`。
常用函式有`fs::write`、`fs::read`、`fs::read_to_string`。

``` rust
use std::fs;

let contents = fs::read_to_string(filename)
    .expect("Something went wrong reading the file");
```

## 輸出錯誤訊息

以`eprintln!()` macro取代`println!()`。

## 取得環境變數

以`env::var()`取得環境變數。以`is_err()`確認變數是否存在，不存在為true。

``` rust
use std::env;

let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
```

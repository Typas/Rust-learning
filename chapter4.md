# Notes

## 記憶體配置
- 區塊以`{}`包覆
- 結束區塊時自動呼叫`drop`函式清理動態配置的記憶體

## = 對資料的操作
- 純配置在stack上的資料直接複製
- 配置在heap上的資料轉移所有權，舊變數無效

## 對變數進行深度複製
- 使用`clone`函式複製heap資料
``` rust
let s1 = String::from("hello");
let s2 = s1.clone();
```

回到[目錄](./../README.md)

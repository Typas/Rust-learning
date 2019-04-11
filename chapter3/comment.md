# 3.4 Comments 註解

## Comments 註解
大多的程式設計師致力於讓他們的程式碼更好懂，但有時額外的解釋是必要的。這時，程式設計師們會在程式碼中留下一些筆記，或者「註解」。編譯器會忽略掉註解，但是對於想讀原始碼的人們來說，這很有幫助。

這裡有個簡單的註解：
``` rust
// hello, world
```
Rust中，註解以兩個斜線開始，直到一行的結尾。如果想要寫多行的註解，每一行的開頭都需要`//`，像是這樣：
``` rust
// 我們在做一個很複雜的東西，註解長到需要好幾行
// 才能解釋完。希望這堆註解能解釋這坨在幹嘛。
```
註解也能被放在一行程式碼的後面：
``` rust
let minimum_pay = 23_100; // 基本工資
```

不過更常見的是放在程式碼的上面來解釋：
``` rust
// 基本工資
let minimum_pay = 23100;
```

Rust還有另一種註解叫文件註解，會在第14章討論到。

# Links
- Previous section - 3.3 [How Functions Work](./function.md)
- Next section - 3.5 [Control Flow](./flow.md)
- back to [category](./../README.md)

## References
- 14.2 [Publishing a Crate to Crates.io] 尚未完工

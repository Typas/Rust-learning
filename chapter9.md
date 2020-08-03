# Notes

這章主要講錯誤的處理。

## 無法回復的錯誤
`panic!` 是在程式出現無法回復的錯誤時使用的巨集。它會印出一個錯誤訊息，回溯並清除stack，然後結束。通常是用在設計者想不到要怎麼處理錯誤時使用。

``` rust
fn main() {
    panic!("crash and burn");
}
```

### 回溯或中斷的選擇
預設是在panic出現時，程式會回溯到每個正在用的函式。不過這很吃效能，有另一種方式是「中斷」，不清除而直接結束程式，此時程式所使用的記憶體會需要作業系統來清。如果想讓編出來的程式盡可能小的話，可以在 *Cargo.toml* 中改寫panic的行為。

``` toml
[profile.release]
panic = 'abort'
```

### 回溯
如果有設定環境變數`RUST_BACKTRACE`的話，在`panic!`作用後就可以看到一長串的回溯，會有包含標準函式庫，內建的，與你所使用的crate中所有正在使用stack的函式。

## 可回復的錯誤
`Result`是一個有兩種可能性的列舉，分別是`Ok`跟`Err`：

``` rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`跟`E`各是泛用型別參數，各對應到一個型別，藉此來分別處理成功與失敗的情況。

### 範例

開檔時所回傳的即為`Result`，結合`match`可處理開檔錯誤時的情況。

``` rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    }
}
```

### 不同錯誤的對應處理

上面那個例子是只要出錯就用`panic!`處理，而實際情況常需要更細微的處理，如檔案覆寫等等。

``` rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            }
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

如此巢狀`match`就能處理無舊檔時直接建立新檔的情況，請記得後面是`,`還是`;`。
第二行的`std::io::ErrorKind`是因為`File::open`回傳的是`std::io::ErrorKind`這個列舉，所以需要引入。

因為`match`看起來很囉嗦，所以後面還有closure來簡化所寫的函式，如上面可以改寫如下：

``` rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

這樣就可以少寫一些正常狀況的對應，而專注在錯誤處理上，雖然還是很囉嗦。


### 錯誤處理的捷徑：`unwrap`與`expect`

錯誤處理上，`match`夠用，但是很囉嗦，而且看起來不是很明瞭。`Result <T, E>`型別提供了一些輔助函式來協助。

#### `unwrap`

其中一個叫`unwrap`的，是像上面那個囉嗦`match`的簡化版：如果`Result`回傳值是`Ok`那邊，那`unwrap`就會直接給`Ok`那邊的值；如果`Result`回傳`Err`的話，`unwrap`會改呼叫`panic!`巨集。

``` rust
use std:fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

#### `expect`

另一個是`expect`，不同於`unwrap`的是你要提供錯誤訊息當參數，以方便debug。

``` rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### 傳遞錯誤

如果函式發生錯誤，常常會需要在呼叫函式的地方才處理，這時就該丟回去。

``` rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");
    
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

#### 語法糖：`?`

上面那個一堆`match`的東西可以改寫成下面這樣：

``` rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

請注意`?`只在出錯時回傳，所以最後的回傳值需要補上。而`?`會呼叫`from`函式，需要有實作的`From`特性才能用。
也有更懶的寫法，把上面的`open`與`read_to_string`串起來，而`?`一樣需要在對應的位置。

##### `?` 的侷限性

這個語法糖的限制條件為函式的回傳值必須是`Result`或`Option`型別。


## 錯誤可否回復的抉擇

在寫雛型時，panic是一個很好的錯誤處理預留方式。而要不要用`Result`則取決於你是否有比編譯器更多的資訊，也就是程式邏輯上需要考慮的事。而有下面三種情況之一，就該考慮使用`panic!`：

- 錯誤狀況不是預期下發生的事
- 再來的程式碼需要不是在這錯誤情況下執行
- 沒有一個好的方式來用目前所用的型別記錄這個資訊

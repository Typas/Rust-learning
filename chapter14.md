# Notes

這章主要重講Cargo跟Crates.io。

## 自訂build

Cargo有兩種profile，一個是執行`cargo build`時使用的`dev`，另一個是執行`cargo build --release`時使用的`release`。
在*Cargo.toml*中加入`[profile.*]`可以自訂任何你想要的profile，也能蓋過預設值。

舉例來說，`dev`與`release` profile的預設`opt-level`值為：
``` toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

這個值範圍為0到3，值越小做的最佳化也越少，編譯的時間也會越短。

## 發佈到Crates.io

### 文件註解

常見的註解是以`//`開頭，能轉譯成HTML文件的則是以`///`開頭的註解，稱為文件註解。
這種註解能夠支援Markdown語法，程式區塊的預設語言為Rust。
這些文件能以`cargo doc`產生出來，而在後加上參數`--open`則可以順便在瀏覽器中打開。

#### 常用標題

除了最常見的`# Examples`外，還有幾個也很常用的：

- **Panics:** 會標出函數在什麼時候會觸發panic機制。
- **Errors:** 當函數回傳`Result`時，在哪些情況會回傳哪些錯誤。
- **Safety:** 如果函數本身為`unsafe`時，需要有一段來解釋為什麼函數是`unsafe`的，還有函數所預期的環境。

#### 文件測試

所有在文件中的Rust程式碼在測試時也會一併編譯測試，以確保範例本身有跟上程式碼的更動。

#### 包含物件的註解

`//!`常用在整個crate的根檔案，常為*src/lib.rs*，主要用途是描述整個module或crate是在做什麼的。

### 以`pub use`來提供方便的對外API

之前已經提過用`pub`公開函數與模組。
不過因為程式碼的結構可能很多層，如果引用時要用到`use my_crate::some_module::another_module::UsefulType`這麼長又這麼難記的路徑時，對使用者來說很不方便。
`pub use`可以讓已經公開過的函數或模組能直接在表層就被引用，而不改變程式的結構。
如果想用`use my_crate::UsefulType`來替代前面的長路徑時，可以在更上層的地方如*src/lib.rs*使用`pub use`如下：

``` rust
pub use self::some_module::another_module::UsefulType;

pub mod some_module {
    // --snip--
}
```

### 設定Crates.io的帳號

在公開發布你的crate前，必須在[crates.io](https://crates.io/)先建立一個帳號並拿到API token。目前的crates.io是綁定GitHub帳號。
當登入後，到[crates.io/me/](https://crates.io/me/)獲得你的API key。再執行`cargo login`綁定API key如下：

```
$ cargo login myapikey12345
```

這個API key會存在*~/.cargo/credentials*，請注意這是個密鑰，不要把這個密鑰交給任何人。
當不小心洩露時，請儘快重新取得一次API key。

### 在Crate增加Metadata

Crates.io上一個名稱只會對應到一個crate，所以沒辦法發布撞名的crate。在正式發布前，請先搜尋有沒有重複到名稱。
如果有，你需要更改在*Cargo.toml*裡`[package]`的名字，如下所示：

``` toml
[package]
name = "guessing_game"
```

而`license`欄位則可以用[SPDX](https://spdx.org/licenses/)裡的*license identifier value*來直接指定授權，使用多個授權時中間則以`OR`隔開。
最後你的*Cargo.toml*應該會正式發布前會長成這樣：

``` toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["0.1.0"]
edition = "2018"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```


### 發布到Crates.io

使用`cargo publish`把整個crate給公開發表出去。每有一個新版本時，更改完`version`的值後再用一次`cargo publish`就會更新版本。

#### 移除在Crates.io的版本

雖然你不能移除之前的crate版本，但是你可以預防未來的專案使用這些版本作為相依版本。
`cargo yank`可以讓一個版本在已使用的專案仍相依於該版本，而防止新專案使用該版本。
使用例如下所示：
```
$ cargo yank --vers 1.0.1
```

如果要再發放此版本的使用，則可在後面再加上參數`--undo`來回復：

```
$ cargo yank --vers 1.0.1 --undo
```

## Cargo Workspaces

Workspace是一組分享同一個*Cargo.lock*與輸出目錄的package。
在workspace根目錄的*Cargo.toml*不會有其他*Cargo.toml*的`[package]`與其他詮釋資料。
而會以`[workspace]`開頭，描述整個workspace的組成。

``` toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

在之後用`cargo new adder`會作成一個共用*target*目錄的package。`cargo new add-one --lib`則會建立一個新的library。
Cargo並不會預設其內部的相依性，所以仍需手動指定，如在*adder/Cargo.toml*：

``` toml
[dependencies]

add-one = { path = "../add-one" }
```

如果要執行特定的package，則需要以`-p`來指定：

```
$ cargo run -p adder
```

## 從Crates.io安裝程式

`cargo install`能讓你從crates.io上安裝程式，如安裝`ripgrep`：

```
$ cargo install ripgrep
```

一般而言使用rustup安裝時都會在`$PATH`新增*$HOME/.cargo/bin*，如果是從系統安裝的可能要自行加上。

## 自訂Cargo指令

只要在`$PATH`裡有執行檔的命名是`cargo-something`，就能用`cargo something`的指令執行。想知道有哪些指令可以用`cargo --list`查詢。

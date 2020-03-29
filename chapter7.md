# Notes

這章主要在講Cargo的專案管理。

## Package

### 定義
Cargo的一個功能，能讓你build、測試與分享crate。
Package包含了 *Cargo.toml* 描述如何build這些crate。

### 限制
一個package只能有少於一個函式庫crate、但能有無限多執行檔crate，合計至少要包含一個crate。

### 使用
`cargo new`出來的就是一個package。

### External Package
如果要在專案中使用一個外部的套件，要在 *Cargo.toml* 加入這個套件：

``` toml
[dependencies]
rand = "0.5.5"
```

標準函式庫也是外部套件，但是預設會加入。而要簡化path一樣要用`use`，絕對路徑以`std`開頭。

## Crate

### 定義
一個提供函式庫或執行檔的module樹。

### 使用
以`cargo new --lib`建立函式庫crate。
看起來好像只有 *src/lib.rs* 是函式庫crate，其他都會被要求放入 *src/bin*，每個檔案被視為是分別的binary crate。

## Module

### 定義
與`use`連用，讓你控制path的組成、範圍與可見性。

### 使用
關鍵字為`mod`，以`{}`包圍與一般函式相同。
使用上與C++的namespace很相似。

#### 檔案規則
若要分開module宣告與實作，則需要在 *src/lib.rs* 中宣告`mod mod_name;`，再建立 *mod_name.rs* 檔案與 *src/mod_name* 目錄。而在 *mod_name.rs* 中只放其child module的宣告。為一遞迴關係。

### `use`
類似C++的`using`，或是檔案系統中的symbolic link，能簡化path的長度。

``` rust
use path::to::module::mod_name;
```

#### `as`
為path取上別名，能避免重覆名稱的衝突。

``` rust
use path::to::module::mod_name as alias;
```
#### `pub use`
預設的`use`是private的，這個link一樣只有範圍內的能用，若要讓外部code也能用這個link，則需要寫成`pub use`。

#### Nested Path
合併多個`use`，減少行數。如以下的`use`敘述：

``` rust
use std::io;
use std::io::Write;
use std::cmp::Ordering;
```

可以縮為：

``` rust
use std::{cmp::Ordering, io::{self, Write}};
```

#### Glob Operator
如果想把所有path定義的public item帶進來，可以用glob operator `*`：

``` rust
use std::collections::*;
```

但是因為這個引用並不明確，可能會造成debug困難。

## Path

### 定義
一個命名像是struct、function或module等item的方式。

### 使用
分為絕對路徑與相對路徑。絕對路徑從crate名或`crate`開始；相對路徑從所在的module開始，並用module內的`self`、`super`或任何識別字來描述路徑。多個識別字之間以雙冒號`::`分隔，同C++。

#### `super`
概念類似檔案系統中的`..`，能存取外面一層的item。

### 限制
路徑中的任何module都不能是private。

#### Privacy Boundary
Rust的所有item(函式、method、結構、enum、module與常數)預設都是private。外層module的item不能用其內module的private item，但內層的可以用外層的private module。item前加上關鍵字`pub`可以把item轉換成public。

##### Struct與Enum的Public
在struct前加上`pub`後，其內的任何欄位與函式仍然維持private，需自行在要公開的欄位前加上`pub`。enum則是加上`pub`後內部所有欄位都是public。


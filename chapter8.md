# Notes

這章主要講常見的資料結構，被包含在 *collection* 裡面。

## Vector

### 建立
有兩種方法
#### 空的Vector
可用`Vec::new`，但需要指定型別，因為未儲存任何元素所以無法推導。

``` rust
let v: Vec<i32> = Vec::new();
```

#### 有數個值的Vector
`vec!`是個方便的巨集，裡面的元素會讓編譯器自行推導出型別。

``` rust
let v = vec![1, 2, 3];
```

### 更新
`push`函式可以用來更新vector，記得需要`mut`屬性。

``` rust
let mut v = Vec::new();

v.push(5);
v.push(6);
```

這邊的vector因為可推導所以不用指定型別。

### 移除
跟其他struct一樣，vector在生命期結束時也會drop所有它的內容。

### 讀取
有兩種方法，一個用index取值，一個用`get`函式。indexing需合用`&`與`[]`，會給出參考；`get`函式則會回傳一個`Option<&T>`的參考。immutable reference會與`push`衝突，請注意其之間的生命期。

#### indexing syntax
``` rust
let v = vec![1, 2, 3];

let third: &i32 = &v[2];
println!("The third element is {}", third);
```

#### `get` method
``` rust
let v = vec![1, 2, 3];

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

### Iteration
分為兩種，不變式跟可變式。

#### immutable way
``` rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

#### mutable way
``` rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```
這邊需要解參考運算子`*`取值。

### Enum 與 `vec!` 結合使用
概念是`vec!`只吃同一個型別，而`enum`可以存不同的型別。結合`enum`與`match`可以做到個別處理內部每個型別。

``` rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

## String

### 定義
標準函式庫提供的可成長、可操作、有所有權的UTF-8編碼字串型別。slice型別為`&str`。

### 建立

- 與`Vec<T>`相同的`new`函式

``` rust
let mut s = String::new();
```

- `to_string`函式，只要有實作`Display`特性的都可以。
``` rust
let data = "initial contents";

let s = data.to_string();

let s = "initial contents".to_string();
```

- `String::from`函式
``` rust
let s = String::from("initial contents");
```
### 更新

#### Append
需要被操作的字串有`mut`，被「複製」的字串或字元則不需。

- `push_str`：追加字串切片

``` rust
let mut s = String::from("foo");
s.push_str("bar");
```

- `push`：追加字元

``` rust
let mut s = String::from("lo");
s.push('l');
```

#### Concatenation

- `+` 運算子：注意所有權的轉移關係與型別

``` rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

- `format!` 巨集：不轉移所有權
概念類似C的`sprintf()`。

``` rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

### Indexing
不支援。`String`雖然是包裝後的`Vec<u8>`，但因為UTF-8字元編碼的關係所以index難以直接對應字元。

### Slice
這邊的index是照byte算的，所以下面的slice實際是兩個中文字。

``` rust
let hello = "你好";

let s = &hello[0..4];
```

### Iteration
能用`chars`函式取得字串內的UTF-8字元，但是部分語言會有獨立聲調的字元存在。

``` rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

用`bytes`則是取得每個byte的數值，但有長於一個byte的UTF-8字元。

## Hash Map
因為使用頻率相對低所以不被包在預載的函式庫中。

``` rust
use std::collections::HashMap;
```

### 建立

- 先用`new`創建空的hash map，再用`insert`加入元素。

``` rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

- 以`collect`將vector of tuples轉成hash map。
而vector of tuples則以`iter`結合`zip`組合。

``` rust
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

### 所有權

在使用了`insert`後，所有權會轉移到hash map上，所以原有的變數會無法使用。

### 存取

- `get`的參數為key，會回傳Option<&value>。

``` rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

- 也可以用`for`迴圈

``` rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

### 更新

#### 覆寫值

因為一個key只會對應到一個value，所以同個key再做`insert`就會覆寫原有的value。

``` rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);
```

#### 無值才新增

因為有些hash map會需要避免覆寫，可以用`entry`函式回傳的Entry這個列舉來確認。
`or_insert`會在Entry的key不存在於hash map時新增一個tuple進hash map。

``` rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);
```

#### 僅更新現有值

與上面的情景完全相反的情況也很常見, 可根據回傳值來操作達成。
回傳的型別為`&mut V`，所以需要解參考。

``` rust
let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

### Hashing Functions

預設是較高安全度的，也可以指定不同的hasher以達成最佳化。

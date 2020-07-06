# Notes

這章講Smart Pointer智慧型指標。

## 先備知識

指標是一種指向記憶體位址的變數。在Rust中最常見的是以參考型式`&`存在，不過沒有overhead。
而智慧型指標則是用在有更間接的情況，如指向metadata詮釋資料的情形。Rust的智慧型指標是自C++11來的，能提供比參考更強力的使用性。
通常智慧型指標會與結構連用，而其中的`Deref`與`Drop`兩種特性能讓你更便利的使用智慧型指標。
標準函式庫中常見的智慧型指標有3類：
- `Box<T>`能提供在heap上的定址。
- `Rc<T>`能允許資料的多重所有權，並在無擁有者時釋放資料。
- `Ref<T>`、`RefMut<T>`，以`RefCell<T>`存取，能讓借用規則從編譯時檢查轉向在執行時檢查，有可能造成memory leak。

## Box

## Rc

## Ref, RefMut, RefCell

## `Deref` Trait

## `Drop` Trait


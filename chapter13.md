# Notes

這章講iterator跟closure。

## Closure

語法結構為`|var: Type| -> ReturnType { statements; expression }`，其中的`var`可以單個`_`表示不需傳入參數，參數型態有時可省略，單行表達或敘述可省略大括號。

## `Fn` Trait
這會結合到泛型的使用。其中`Fn`表示函數只使用immutable方式借用外部變數；`FnMut`表示函數使用mutable方式借用外部變數；`FnOnce`表示函數會得到外部變數的所有權。

``` rust 
struct Cacher<T>
where T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}
```

## Iterator

### `Iterator` Trait
需要實作的method為`next()`。
``` rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

之後便可以`var.iter()`產出iterator。

### 拿走 Iterator 的 Methods

在文件中第一個參數是`self`的就是了，常見的有`count()`、`last()`、`collect()`、`max()`、`min()`、`sum()`。

### 生成其他的 Iterator

必定會轉移iterator的所有權，常見的有`map()`、`filter()`、`zip()`。

### 效能

使用iterator的效能常比使用loop還高。

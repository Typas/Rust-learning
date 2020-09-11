# Option

[Source Code Location](https://github.com/rust-lang/rust/blob/master/library/core/src/option.rs)

# Use
``` rust
use core::iter::{FromIterator, FusedIterator, TrustedLen};
use core::pin::Pin;
use core::{
    convert, fmt, hint, mem,
    ops::{self, Deref, DerefMut},
};
```

下方到時看看有什麼有使用到的。

# Definition
``` rust
#[derive(Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
pub enum Option<T> {
    /// No value
    None,
    /// Some value `T`
    Some(T),
}
```

在去除一些給編譯器看的屬性巨集後，是非常簡單的定義。

# Associated Functions

# Const Methods

## Stable method, but unstable const, issue 67441

### is_some
``` rust
#[inline]
pub const fn is_some(&self) -> bool {
    matches!(*self, Some(_))
}
```

用到了 [matches](https://doc.rust-lang.org/std/macro.matches.html) 巨集，在對應到任何有值的東西時回傳 `true` ，反之則回傳 `false` 。

### is_none
``` rust
#[inline]
pub const fn is_none(&self) -> bool {
    !self.is_some()
}
```

上面的 `is_some` 的反轉，只是我不太能理解為什麼是用 `Some` 而不是 `None` 去對應。

### unwrap
``` rust
#[inline]
#[track_caller]
pub const fn unwrap(self) -> T {
    match self {
        Some(val) => val,
        None => panic!("called `Option::unwrap()` on a `None` value"),
    }
}
```

就是一個在 `None` 會噴死你的函數，會拿走所有權。

### iter
``` rust
#[inline]
pub const fn iter(&self) -> Iter<'_, T> {
    Iter { inner: Item { opt: self.as_ref() } }
}
```

回傳的型態 [Iter](iter-1) 是模組內定義的一個結構。



# Safe Methods

## Stable

### as_ref
``` rust
#[inline]
pub const fn as_ref(&self) -> Option<&T> {
    match *self {
        Some(ref x) => Some(x),
        None => None,
    }
}
```

用到了 `match` 裡的 `ref` 關鍵字，配對的值本身型態是 =U= 時，加上 =ref= 則在分支內的變數 =x= 會是 =&U= 型態。

### as_mut
``` rust
#[inline]
pub fn as_mut(&mut self) -> Option<&mut T> {
    match *self {
        Some(ref mut x) => Some(x),
        None => None,
    }
}
```

同上，多用了一個 =mut= 關鍵字，型態改變成 =&mut U= 。

### as_pin_ref
``` rust
#[inline]
pub fn as_pin_ref(self: Pin<&Self>) -> Option<Pin<&T>> {
    // SAFETY: `x` is guaranteed to be pinned because it comes from `self`
    // which is pinned.
    unsafe { Pin::get_ref(self).as_ref().map(|x| Pin::new_unchecked(x)) }
}
```

TODO: usage of `core::pin::Pin`

### as_pin_mut
``` rust
#[inline]
pub fn as_pin_mut(self: Pin<&mut Self>) -> Option<Pin<&mut T>> {
    // SAFETY: `get_unchecked_mut` is never used to move the `Option` inside `self`.
    // `x` is guaranteed to be pinned because it comes from `self` which is pinned.
    unsafe { Pin::get_unchecked_mut(self).as_mut().map(|x| Pin::new_unchecked(x)) }
}
```

TODO: usage of `core::pin::Pin`

### expect
``` rust
#[inline]
#[track_caller]
pub fn expect(self, msg: &str) -> T {
    match self {
        Some(val) => val,
        None => expect_failed(msg),
    }
}
```

裡面的 `expect_failed()` 是私有方法，在對應到 `None` 時會觸發。會拿走所有權。

### expect_failed
``` rust
#[inline(never)]
#[cold]
#[track_caller]
fn expect_failed(msg: &str) -> ! {
    panic!("{}", msg)
}
```

屬性巨集 `cold` 是對編譯器提示這個函數不太可能被呼叫到。問題：為何會需要獨立成一個方法？

### unwrap_or
``` rust
#[inline]
pub fn unwrap_or(self, default: T) -> T {
    match self {
        Some(x) => x,
        None => default,
    }
}
```

請注意這裡會拿走自身與 `default` 的所有權。

### unwrap_or_else
``` rust
#[inline]
pub fn unwrap_or_else<F: FnOnce() -> T>(self, f: F) -> T {
    match self {
        Some(x) => x,
        None => f(),
    }
}
```

裡面的函數 `f` 會拿走所有內部變數的所有權，自身的所有權也會被此方法拿走。

### map
``` rust
#[inline]
#[stable(feature = "rust1", since = "1.0.0")]
pub fn map<U, F: FnOnce(T) -> U>(self, f: F) -> Option<U> {
    match self {
        Some(x) => Some(f(x)),
        None => None,
    }
}
```

利用 `f` 將自身映射到 `f(x)` 上，此方法會拿走自身的所有權。

### map_or
``` rust
#[inline]
pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
    match self {
        Some(t) => f(t),
        None => default,
    }
}
```

與 `unwrap_or` 相似，但 `default` 的型態為映射後的 `U` 而非原來的 `T` 。

### map_or_else
``` rust
#[inline]
pub fn map_or_else<U, D: FnOnce() -> U, F: FnOnce(T) -> U>(self, default: D, f: F) -> U {
    match self {
        Some(t) => f(t),
        None => default(),
    }
}
```

與 `unwrap_or_else` 相似，但型態為映射後的 `U` 。

### ok_or
``` rust
#[inline]
pub fn ok_or<E>(self, err: E) -> Result<T, E> {
    match self {
        Some(v) => Ok(v),
        None => Err(err),
    }
}
```

把 `Option<T>` 型態轉成 `Result<T, E>` 型態的神方法，需要自行加上錯誤型態，兩個參數都會被拿走所有權。

### ok_or_else
``` rust
#[inline]
pub fn ok_or_else<E, F: FnOnce() -> E>(self, err: F) -> Result<T, E> {
    match self {
        Some(v) => Ok(v),
        None => Err(err()),
    }
}
```

`ok_or` 的錯誤參數改成使用一個回傳錯誤的函數。會拿走自身的所有權。

### iter_mut

## Unstable

### contains
``` rust
#[inline]
#[unstable(feature = "option_result_contains", issue = "62358")]
pub fn contains<U>(&self, x: &U) -> bool
where
    U: PartialEq<T>,
{
    match self {
        Some(y) => x == y,
        None => false,
    }
}
```

從定義看出，這段程式碼在使用了特徵綁定後不需限於同型態，只需要單向的 `PartialEq` 特徵即可。

# Unsafe Methods

# Trait Implementations

# Structs

## Item

### Definition
``` rust
#[derive(Clone, Debug)]
struct Item<A> {
    opt: Option<A>,
}
```



### Trait Implementations

#### Iterator
``` rust
impl<A> Iterator for Item<A> {
    type Item = A;

    #[inline]
    fn next(&mut self) -> Option<A> {
        self.opt.take()
    }

    #[inline]
    fn size_hint(&self) -> (usize, Option<usize>) {
        match self.opt {
            Some(_) => (1, Some(1)),
            None => (0, Some(0)),
        }
    }
}
```

#### DoubleEndedIterator
``` rust
impl<A> DoubleEndedIterator for Item<A> {
    #[inline]
    fn next_back(&mut self) -> Option<A> {
        self.opt.take()
    }
}
```

#### ExactSizeIterator
``` rust
impl<A> ExactSizeIterator for Item<A> {}
```

#### FusedIterator
``` rust
impl<A> FusedIterator for Item<A> {}
```

#### TrustedLen
``` rust
unsafe impl<A> TrustedLen for Item<A> {}
```

## Iter

[iter](#iter) 所回傳的結構。

### Definition
``` rust
#[derive(Debug)]
pub struct Iter<'a, A: 'a> {
    inner: Item<&'a A>,
}
```


### Trait Implementations
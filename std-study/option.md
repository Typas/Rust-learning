
# Table of Contents

1.  [Use](#org15bbbb1)
2.  [Definition](#org7a72245)
3.  [Associated Functions](#orge0d9f94)
4.  [Const Methods](#org7656a25)
    1.  [Stable, but unstable const, issue 67441](#orgeb41097)
        1.  [is<sub>some</sub>](#org0981f42)
        2.  [is<sub>none</sub>](#org674d564)
        3.  [unwrap](#orgc389672)
        4.  [iter](#orgc06fea7)
5.  [Safe Methods](#org09a7560)
    1.  [Stable](#org9ab7e52)
        1.  [as<sub>ref</sub>](#org814af82)
        2.  [as<sub>mut</sub>](#orgecf5645)
        3.  [as<sub>pin</sub><sub>ref</sub>](#org882c641)
        4.  [as<sub>pin</sub><sub>mut</sub>](#org6264c49)
        5.  [expect](#orgd81ed21)
        6.  [expect<sub>failed</sub>](#orgbd9cf3d)
        7.  [unwrap<sub>or</sub>](#org8ec2401)
        8.  [unwrap<sub>or</sub><sub>else</sub>](#orgf5ae483)
        9.  [map](#orga06be3a)
        10. [map<sub>or</sub>](#orgdfeb8da)
        11. [map<sub>or</sub><sub>else</sub>](#org4af344d)
        12. [ok<sub>or</sub>](#orgeac5d28)
        13. [ok<sub>or</sub><sub>else</sub>](#orgb7018f5)
        14. [iter<sub>mut</sub>](#orgb3779ee)
    2.  [Unstable](#org7d8e4e1)
        1.  [contains](#orgad42200)
6.  [Unsafe Methods](#orgaa4322f)
7.  [Trait Implementations](#org932ac50)
8.  [Structs](#orgb9128ad)
    1.  [Item](#org2a6e3c4)
        1.  [Definition](#org7a02e82)
        2.  [Trait Implementations](#org8c4ca26)
    2.  [Iter](#org99e9ddd)
        1.  [Definition](#orgcb1d9ce)
        2.  [Trait Implementations](#org0744690)

[Source Code Location](https://github.com/rust-lang/rust/blob/master/library/core/src/option.rs)


<a id="org15bbbb1"></a>

# Use

    use core::iter::{FromIterator, FusedIterator, TrustedLen};
    use core::pin::Pin;
    use core::{
        convert, fmt, hint, mem,
        ops::{self, Deref, DerefMut},
    };

下方到時看看有什麼有使用到的。


<a id="org7a72245"></a>

# Definition

    #[derive(Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
    pub enum Option<T> {
        /// No value
        None,
        /// Some value `T`
        Some(T),
    }

在去除一些給編譯器看的屬性巨集後，是非常簡單的定義。


<a id="orge0d9f94"></a>

# Associated Functions


<a id="org7656a25"></a>

# Const Methods


<a id="orgeb41097"></a>

## Stable, but unstable const, issue 67441


<a id="org0981f42"></a>

### is<sub>some</sub>

    #[inline]
    pub const fn is_some(&self) -> bool {
        matches!(*self, Some(_))
    }

用到了 [matches](https://doc.rust-lang.org/std/macro.matches.html) 巨集，在對應到任何有值的東西時回傳 `true` ，反之則回傳 `false` 。


<a id="org674d564"></a>

### is<sub>none</sub>

    #[inline]
    pub const fn is_none(&self) -> bool {
        !self.is_some()
    }

上面的 `is_some` [is<sub>some</sub>](#org0981f42) 的反轉，只是我不太能理解為什麼是用 `Some` 而不是 `None` 去對應。


<a id="orgc389672"></a>

### unwrap

    #[inline]
    #[track_caller]
    pub const fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }

就是一個在 `None` 會噴死你的函數，會拿走所有權。


<a id="orgc06fea7"></a>

### iter

<a id="orgac670d2"></a>

    #[inline]
    pub const fn iter(&self) -> Iter<'_, T> {
        Iter { inner: Item { opt: self.as_ref() } }
    }

回傳的型態 [Iter](#orgc8ef16b) 是模組內定義的一個結構。


<a id="org09a7560"></a>

# Safe Methods


<a id="org9ab7e52"></a>

## Stable


<a id="org814af82"></a>

### as<sub>ref</sub>

    #[inline]
    pub const fn as_ref(&self) -> Option<&T> {
        match *self {
            Some(ref x) => Some(x),
            None => None,
        }
    }

用到了 `match` 裡的 `ref` 關鍵字，配對的值本身型態是 `U` 時，加上 `ref` 則在分支內的變數 `x` 會是 `&U` 型態。


<a id="orgecf5645"></a>

### as<sub>mut</sub>

    #[inline]
    pub fn as_mut(&mut self) -> Option<&mut T> {
        match *self {
            Some(ref mut x) => Some(x),
            None => None,
        }
    }

同上，多用了一個 `mut` 關鍵字，型態改變成 `&mut U` 。


<a id="org882c641"></a>

### as<sub>pin</sub><sub>ref</sub>

    #[inline]
    pub fn as_pin_ref(self: Pin<&Self>) -> Option<Pin<&T>> {
        // SAFETY: `x` is guaranteed to be pinned because it comes from `self`
        // which is pinned.
        unsafe { Pin::get_ref(self).as_ref().map(|x| Pin::new_unchecked(x)) }
    }

TODO: usage of `core::pin::Pin`


<a id="org6264c49"></a>

### as<sub>pin</sub><sub>mut</sub>

    #[inline]
    pub fn as_pin_mut(self: Pin<&mut Self>) -> Option<Pin<&mut T>> {
        // SAFETY: `get_unchecked_mut` is never used to move the `Option` inside `self`.
        // `x` is guaranteed to be pinned because it comes from `self` which is pinned.
        unsafe { Pin::get_unchecked_mut(self).as_mut().map(|x| Pin::new_unchecked(x)) }
    }

TODO: usage of `core::pin::Pin`


<a id="orgd81ed21"></a>

### expect

    #[inline]
    #[track_caller]
    pub fn expect(self, msg: &str) -> T {
        match self {
            Some(val) => val,
            None => expect_failed(msg),
        }
    }

裡面的 `expect_failed()` 是私有方法，在對應到 `None` 時會觸發。會拿走所有權。


<a id="orgbd9cf3d"></a>

### expect<sub>failed</sub>

    #[inline(never)]
    #[cold]
    #[track_caller]
    fn expect_failed(msg: &str) -> ! {
        panic!("{}", msg)
    }

屬性巨集 `cold` 是對編譯器提示這個函數不太可能被呼叫到。問題：為何會需要獨立成一個方法？


<a id="org8ec2401"></a>

### unwrap<sub>or</sub>

    #[inline]
    pub fn unwrap_or(self, default: T) -> T {
        match self {
            Some(x) => x,
            None => default,
        }
    }

請注意這裡會拿走自身與 `default` 的所有權。


<a id="orgf5ae483"></a>

### unwrap<sub>or</sub><sub>else</sub>

    #[inline]
    pub fn unwrap_or_else<F: FnOnce() -> T>(self, f: F) -> T {
        match self {
            Some(x) => x,
            None => f(),
        }
    }

裡面的函數 f 會拿走所有內部變數的所有權，自身的所有權也會被此方法拿走。


<a id="orga06be3a"></a>

### map

    #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn map<U, F: FnOnce(T) -> U>(self, f: F) -> Option<U> {
        match self {
            Some(x) => Some(f(x)),
            None => None,
        }
    }

利用 f 將自身映射到 f(x) 上，此方法會拿走自身的所有權。


<a id="orgdfeb8da"></a>

### map<sub>or</sub>

    #[inline]
    pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default,
        }
    }

與 `unwrap_or` 相似，但 `default` 的型態為映射後的 `U` 而非原來的 `T` 。


<a id="org4af344d"></a>

### map<sub>or</sub><sub>else</sub>

    #[inline]
    pub fn map_or_else<U, D: FnOnce() -> U, F: FnOnce(T) -> U>(self, default: D, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default(),
        }
    }

與 `unwrap_or_else` 相似，但型態為映射後的 `U` 。


<a id="orgeac5d28"></a>

### ok<sub>or</sub>

    #[inline]
    pub fn ok_or<E>(self, err: E) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err),
        }
    }

把 `Option<T>` 型態轉成 `Result<T, E>` 型態的神方法，需要自行加上錯誤型態，兩個參數都會被拿走所有權。


<a id="orgb7018f5"></a>

### ok<sub>or</sub><sub>else</sub>

    #[inline]
    pub fn ok_or_else<E, F: FnOnce() -> E>(self, err: F) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err()),
        }
    }

`ok_or` 的錯誤參數改成使用一個回傳錯誤的函數。會拿走自身的所有權。


<a id="orgb3779ee"></a>

### iter<sub>mut</sub>


<a id="org7d8e4e1"></a>

## Unstable


<a id="orgad42200"></a>

### contains

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

從定義看出，這段程式碼在使用了特徵綁定後不需限於同型態，只需要單向的PartialEq特徵即可。


<a id="orgaa4322f"></a>

# Unsafe Methods


<a id="org932ac50"></a>

# Trait Implementations


<a id="orgb9128ad"></a>

# Structs


<a id="org2a6e3c4"></a>

## Item


<a id="org7a02e82"></a>

### Definition

    #[derive(Clone, Debug)]
    struct Item<A> {
        opt: Option<A>,
    }


<a id="org8c4ca26"></a>

### Trait Implementations

1.  Iterator

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

2.  DoubleEndedIterator

        impl<A> DoubleEndedIterator for Item<A> {
            #[inline]
            fn next_back(&mut self) -> Option<A> {
                self.opt.take()
            }
        }

3.  ExactSizeIterator

        impl<A> ExactSizeIterator for Item<A> {}

4.  FusedIterator

        impl<A> FusedIterator for Item<A> {}

5.  TrustedLen

        unsafe impl<A> TrustedLen for Item<A> {}


<a id="org99e9ddd"></a>

## Iter

<a id="orgc8ef16b"></a>

[iter()](#orgac670d2) 所回傳的結構。


<a id="orgcb1d9ce"></a>

### Definition

    #[derive(Debug)]
    pub struct Iter<'a, A: 'a> {
        inner: Item<&'a A>,
    }


<a id="org0744690"></a>

### Trait Implementations


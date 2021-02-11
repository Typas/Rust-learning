
# Table of Contents

1.  [Use](#org36b97a1)
2.  [Definition](#org1a470a7)
3.  [Associated Functions](#org21b1be7)
4.  [Const Methods](#orge839726)
    1.  [Stable](#org1240e89)
        1.  [`is_some`](#orgeb4a89f)
        2.  [`is_none`](#org949f5df)
        3.  [`as_ref`](#orgb653f65)
    2.  [Stable, but unstable const](#org7edbec6)
        1.  [`unwrap`](#orgaaa4ceb)
        2.  [`iter`](#orga4f9f25)
5.  [Safe Methods](#orgdaac50f)
    1.  [Stable](#org18f2d9d)
        1.  [`as_mut`](#org44bbb08)
        2.  [`as_pin_ref`](#orgde39291)
        3.  [`as_pin_mut`](#org696c40a)
        4.  [`expect`](#org2faad9a)
        5.  [`expect_failed`](#orge0613f5)
        6.  [`unwrap_or`](#orge778f19)
        7.  [`unwrap_or_else`](#org7e7c1fb)
        8.  [`map`](#org183765c)
        9.  [`map_or`](#orge0d7dbd)
        10. [`map_or_else`](#org7ff12e4)
        11. [`ok_or`](#orgc3d7409)
        12. [`ok_or_else`](#org24c446d)
        13. [`insert`](#orgfef7c9b)
        14. [`iter_mut`](#org53d8389)
        15. [`and`](#orgf8e1056)
        16. [`and_then`](#org657ee93)
        17. [`filter`](#orga190982)
        18. [`or`](#orga7d1390)
        19. [`or_else`](#orgddaf2af)
        20. [`xor`](#orgcef6e1a)
        21. [`get_or_insert`](#org6ecf145)
        22. [`get_or_insert_with`](#org46ca9ee)
        23. [`take`](#orgfc024ac)
        24. [`replace`](#org8f1d7f1)
        25. [`zip`](#orgad9116a)
        26. [`copied`](#org59ec8cb)
        27. [`cloned`](#org5dd65a3)
        28. [`unwrap_or_default`](#orgca006ef)
        29. [`as_deref`](#org634b2f3)
        30. [`as_deref_mut`](#org372f263)
        31. [`transpose`](#orga2885cb)
    2.  [Unstable](#orgd66f45d)
        1.  [`contains`](#org2025453)
        2.  [`zip_with`](#org82eec6b)
        3.  [`expect_none`](#org80290e2)
        4.  [`expect_none_failed`](#org4bff605)
        5.  [`unwrap_none`](#orgfd98607)
        6.  [`flatten`](#org2d44ece)
6.  [Unsafe Methods](#org39356e6)
    1.  [Unstable](#org90a1f59)
        1.  [`unwrap_unchecked`](#org2341902)
7.  [Trait Implementations](#orga5e2b3d)
    1.  [Clone](#org04967e6)
    2.  [Default](#org9fa2e79)
    3.  [IntoIterator](#orgb1a01c9)
    4.  [From](#orgaf91253)
    5.  [FromIterator](#orgd232f4d)
    6.  [ops::Try (Unstable)](#orgca43045)
8.  [Structs](#org7adc0de)
    1.  [Item](#org36ea248)
        1.  [Definition](#org53d8193)
        2.  [Trait Implementations](#orgd176b6a)
    2.  [Iter](#org8b017c8)
        1.  [Definition](#org68b2e04)
        2.  [Trait Implementations](#orgfa524d8)
    3.  [IterMut](#org07e8dc1)
        1.  [Definition](#org4a3a821)
        2.  [Trait Implementations](#org0eb1850)
    4.  [IntoIter](#orgc09ea42)
        1.  [Definition](#org4ecede6)
        2.  [Trait Implementations](#org61b5ff4)
    5.  [NoneError (Unstable)](#orgf8b1d1a)

[Source Code Location](https://github.com/rust-lang/rust/blob/master/library/core/src/option.rs)


<a id="org36b97a1"></a>

# <a id="org57b7606"></a> Use

    use core::iter::{FromIterator, FusedIterator, TrustedLen};
    use core::pin::Pin;
    use core::{
        convert, fmt, hint, mem,
        ops::{self, Deref, DerefMut},
    };


<a id="org1a470a7"></a>

# Definition

    #[derive(Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
    pub enum Option<T> {
        /// No value
        None,
        /// Some value `T`
        Some(T),
    }

在去除一些給編譯器看的屬性巨集後，是非常簡單的定義。


<a id="org21b1be7"></a>

# Associated Functions


<a id="orge839726"></a>

# Const Methods


<a id="org1240e89"></a>

## Stable


<a id="orgeb4a89f"></a>

### <a id="org0f3f733"></a> `is_some`

    #[inline]
    pub const fn is_some(&self) -> bool {
        matches!(*self, Some(_))
    }

用到了 [matches](https://doc.rust-lang.org/std/macro.matches.html) 巨集，在對應到任何有值的東西時回傳 `true` ，反之則回傳 `false` 。


<a id="org949f5df"></a>

### <a id="org0a6c7ed"></a> `is_none`

    #[inline]
    pub const fn is_none(&self) -> bool {
        !self.is_some()
    }

上面的 `is_some` 的反轉，只是我不太能理解為什麼是用 `Some` 而不是 `None` 去對應。


<a id="orgb653f65"></a>

### <a id="orgbd44e20"></a> `as_ref`

    #[inline]
    pub const fn as_ref(&self) -> Option<&T> {
        match *self {
            Some(ref x) => Some(x),
            None => None,
        }
    }

用到了 `match` 裡的 `ref` 關鍵字，配對的值本身型態是 `U` 時，加上 `ref` 則在分支內的變數 `x` 會是 `&U` 型態。


<a id="org7edbec6"></a>

## Stable, but unstable const


<a id="orgaaa4ceb"></a>

### <a id="org058f235"></a> `unwrap`

    #[inline]
    #[track_caller]
    pub const fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }

就是一個在 `None` 會噴死你的函數，會拿走所有權。


<a id="orga4f9f25"></a>

### <a id="orgb3294f3"></a> `iter`

    #[inline]
    pub const fn iter(&self) -> Iter<'_, T> {
        Iter { inner: Item { opt: self.as_ref() } }
    }

回傳的型態 [Iter](#org8e324a6) 是模組內定義的一個結構。


<a id="orgdaac50f"></a>

# Safe Methods


<a id="org18f2d9d"></a>

## Stable


<a id="org44bbb08"></a>

### <a id="org1fbeaf4"></a> `as_mut`

    #[inline]
    pub fn as_mut(&mut self) -> Option<&mut T> {
        match *self {
            Some(ref mut x) => Some(x),
            None => None,
        }
    }

同上，多用了一個 `mut` 關鍵字，型態改變成 `&mut U` 。


<a id="orgde39291"></a>

### <a id="orga3ea116"></a> `as_pin_ref`

    #[inline]
    pub fn as_pin_ref(self: Pin<&Self>) -> Option<Pin<&T>> {
        // SAFETY: `x` is guaranteed to be pinned because it comes from `self`
        // which is pinned.
        unsafe { Pin::get_ref(self).as_ref().map(|x| Pin::new_unchecked(x)) }
    }

[use](#org57b7606)
TODO: usage of `core::pin::Pin`


<a id="org696c40a"></a>

### <a id="org144ee13"></a> `as_pin_mut`

    #[inline]
    pub fn as_pin_mut(self: Pin<&mut Self>) -> Option<Pin<&mut T>> {
        // SAFETY: `get_unchecked_mut` is never used to move the `Option` inside `self`.
        // `x` is guaranteed to be pinned because it comes from `self` which is pinned.
        unsafe { Pin::get_unchecked_mut(self).as_mut().map(|x| Pin::new_unchecked(x)) }
    }

[use](#org57b7606)
TODO: usage of `core::pin::Pin`


<a id="org2faad9a"></a>

### <a id="org7238b21"></a> `expect`

    #[inline]
    #[track_caller]
    pub fn expect(self, msg: &str) -> T {
        match self {
            Some(val) => val,
            None => expect_failed(msg),
        }
    }

裡面的 `expect_failed()` 是私有方法，在對應到 `None` 時會觸發。會拿走所有權。


<a id="orge0613f5"></a>

### `expect_failed`

    #[inline(never)]
    #[cold]
    #[track_caller]
    fn expect_failed(msg: &str) -> ! {
        panic!("{}", msg)
    }

屬性巨集 `cold` 是對編譯器提示這個函數不太可能被呼叫到。問題：為何會需要獨立成一個方法？


<a id="orge778f19"></a>

### <a id="org41b047c"></a> `unwrap_or`

    #[inline]
    pub fn unwrap_or(self, default: T) -> T {
        match self {
            Some(x) => x,
            None => default,
        }
    }

請注意這裡會拿走自身與 `default` 的所有權。


<a id="org7e7c1fb"></a>

### <a id="org65e9d2d"></a> `unwrap_or_else`

    #[inline]
    pub fn unwrap_or_else<F: FnOnce() -> T>(self, f: F) -> T {
        match self {
            Some(x) => x,
            None => f(),
        }
    }

裡面的函數 `f` 會拿走所有內部變數的所有權，自身的所有權也會被此方法拿走。


<a id="org183765c"></a>

### <a id="org17f15d4"></a> `map`

    #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn map<U, F: FnOnce(T) -> U>(self, f: F) -> Option<U> {
        match self {
            Some(x) => Some(f(x)),
            None => None,
        }
    }

利用 `f` 將自身映射到 `f(x)` 上，此方法會拿走自身的所有權。


<a id="orge0d7dbd"></a>

### <a id="org9d02474"></a> `map_or`

    #[inline]
    pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default,
        }
    }

與 [unwrap<sub>or</sub>](#org41b047c) 相似，但 `default` 的型態為映射後的 `U` 而非原來的 `T` 。


<a id="org7ff12e4"></a>

### <a id="orge8fa7ef"></a> `map_or_else`

    #[inline]
    pub fn map_or_else<U, D: FnOnce() -> U, F: FnOnce(T) -> U>(self, default: D, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default(),
        }
    }

與 [unwrap<sub>or</sub><sub>else</sub>](#org65e9d2d) 相似，但型態為映射後的 `U` 。


<a id="orgc3d7409"></a>

### <a id="org7bde697"></a> `ok_or`

    #[inline]
    pub fn ok_or<E>(self, err: E) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err),
        }
    }

把 `Option<T>` 型態轉成 `Result<T, E>` 型態的神方法，需要自行加上錯誤型態，兩個參數都會被拿走所有權。


<a id="org24c446d"></a>

### <a id="org9199867"></a> `ok_or_else`

    #[inline]
    pub fn ok_or_else<E, F: FnOnce() -> E>(self, err: F) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err()),
        }
    }

從 `ok_or` 的錯誤參數改成使用一個回傳錯誤的函數。會拿走自身的所有權。


<a id="orgfef7c9b"></a>

### <a id="org35bd432"></a> `insert`

    #[inline]
    #[unstable(feature = "option_insert", reason = "newly added", issue = "78271")]
    pub fn insert(&mut self, value: T) -> &mut T {
        *self = Some(value);
    
        match self {
            Some(v) => v,
            // SAFETY: the code above just filled the option
            None => unsafe { hint::unreachable_unchecked() },
        }
    }

以 `value` 取代原有的值，原值會被丟掉。 `value` 的所有權會被轉移，回傳的為可變參照。


<a id="org53d8389"></a>

### <a id="org66f4815"></a> `iter_mut`

    #[inline]
    pub fn iter_mut(&mut self) -> IterMut<'_, T> {
        IterMut { inner: Item { opt: self.as_mut() } }
    }

利用新結構 [IterMut](#org42ed16c) 來達成特徵實作的隔離。


<a id="orgf8e1056"></a>

### <a id="orge02e82f"></a> `and`

    #[inline]
    pub fn and<U>(self, optb: Option<U>) -> Option<U> {
        match self {
            Some(_) => optb,
            None => None,
        }
    }

與不同型態做邏輯的「且」運算，注意是回傳 `optb` 的結果。


<a id="org657ee93"></a>

### <a id="orgc376fee"></a> `and_then`

    #[inline]
    pub fn and_then<U, F: FnOnce(T) -> Option<U>>(self, f: F) -> Option<U> {
        match self {
            Some(x) => f(x),
            None => None,
        }
    }

上方邏輯「且」運算中，將參數 `optb` 改為函數 `f` 的方法。非常類似於 [map()](#org17f15d4) ，但是在函數的定義上不同。 `map()` 使用的函數為回傳型態 `U` 的一次性函數，而 `and_then()` 則是使用回傳型態為 `Option<U>` 的一次性函數。


<a id="orga190982"></a>

### <a id="org5330024"></a> `filter`

    #[inline]
    pub fn filter<P: FnOnce(&T) -> bool>(self, predicate: P) -> Self {
        if let Some(x) = self {
            if predicate(&x) {
                return Some(x);
            }
        }
        None
    }

`P` 為接受一個參數為 `&T` 的一次性函數，為何不用一般的 `Fn` ？這段程式碼是用 `if let` 而不是用 `match` ，不確定這樣寫的原因為何，或許與 `match` 對 `predicate(&x)` 的解析有關係。

我不確定這樣會不會爆炸的寫法：

    match self {
        Some(x) => match predicate(&x) {
            true => Some(x),
            false => None,
        },
        None,
    }


<a id="orga7d1390"></a>

### <a id="org8bbdde4"></a> `or`

    #[inline]
    pub fn or(self, optb: Option<T>) -> Option<T> {
        match self {
            Some(_) => self,
            None => optb,
        }
    }

與同型態做邏輯的「或」運算。在自身有值時會回傳自身，反之回傳 `optb` 。


<a id="orgddaf2af"></a>

### <a id="org3fd37ac"></a> `or_else`

    #[inline]
    pub fn or_else<F: FnOnce() -> Option<T>>(self, f: F) -> Option<T> {
        match self {
            Some(_) => self,
            None => f(),
        }
    }

上方邏輯「或」運算中，把 `optb` 以 `f` 這個函數取代掉的方法。注意「且」能回傳不同型態，而「或」只能回傳同一型態。


<a id="orgcef6e1a"></a>

### <a id="orgab47ec5"></a> `xor`

    #[inline]
    pub fn xor(self, optb: Option<T>) -> Option<T> {
        match (self, optb) {
            (Some(a), None) => Some(a),
            (None, Some(b)) => Some(b),
            _ => None,
        }
    }

與同型態做邏輯的「互斥或」運算。兩者皆有或皆無值時回傳 `None` ，其一有值時回傳有值的一邊。


<a id="org6ecf145"></a>

### <a id="org312072a"></a> `get_or_insert`

    #[inline]
    pub fn get_or_insert(&mut self, value: T) -> &mut T {
        self.get_or_insert_with(|| value)
    }

以一個簡單的閉包直接把 `v` 轉移進去 [get<sub>or</sub><sub>insert</sub><sub>with</sub>()](#orge08f689) ，避免了重複的程式碼。


<a id="org46ca9ee"></a>

### <a id="orge08f689"></a> `get_or_insert_with`

    #[inline]
    pub fn get_or_insert_with<F: FnOnce() -> T>(&mut self, f: F) -> &mut T {
        if let None = *self {
            *self = Some(f());
        }
    
        match self {
            Some(v) => v,
            // SAFETY: a `None` variant for `self` would have been replaced by a `Some`
            // variant in the code above.
            None => unsafe { hint::unreachable_unchecked() },
        }
    }

第一段程式碼將所有的 `None` 轉換成 `Some(f())` 。第二段程式碼中利用強解參照把 `Option<T>` 轉成 `&mut T` 後回傳，其中 `None` 分支由第一段保證不會被執行到，因此用一段 SAFETY 註解標示為何使用到 `unsafe` 。
此處有用到 [core::hint](#org57b7606) 。


<a id="orgfc024ac"></a>

### <a id="orgad04df5"></a> `take`

    #[inline]
    pub fn take(&mut self) -> Option<T> {
        mem::take(self)
    }

此處用到 [core::mem](#org57b7606) 的 [take()](https://doc.rust-lang.org/core/mem/fn.take.html) 。這個函數需要 `Option` 自身的 [Default](#org39a056f) 特徵實作，會將內部型態為 `T` 的值以預設值取代，並回傳被取代的值。若這個型態沒有標註為 `Copy` 特徵，則回傳會拿走所有權。


<a id="org8f1d7f1"></a>

### <a id="org45da4d0"></a> `replace`

    #[inline]
    pub fn replace(&mut self, value: T) -> Option<T> {
        mem::replace(self, Some(value))
    }

此處用到 [core::mem](#org57b7606) 的 [replace()](https://doc.rust-lang.org/core/mem/fn.replace.html) 。這個函數會將內部的值以 `value` 取代，並回傳原本的值。


<a id="orgad9116a"></a>

### <a id="orgbb7774a"></a> `zip`

    pub fn zip<U>(self, other: Option<U>) -> Option<(T, U)> {
        match (self, other) {
            (Some(a), Some(b)) => Some((a, b)),
            _ => None,
        }
    }

這個方法會在兩個值都是 `Some` 的時候綁成一個元組，其餘都會回傳 `None` 。概念上是「且」運算。


<a id="org59ec8cb"></a>

### <a id="orgd4b1fe7"></a> `copied`

需要型態 `T` 有 `Copy` 特徵。

    impl<T: Copy> Option<&T> {
        pub fn copied(self) -> Option<T> {
            self.map(|&t| t)
        }
    }

    impl<T: Copy> Option<&mut T> {
        pub fn copied(self) -> Option<T> {
            self.map(|&mut t| t)
        }
    }

兩個實作十分相近，都是利用閉包特性進行複製。


<a id="org5dd65a3"></a>

### <a id="org06f3a6a"></a> `cloned`

需要型態 `T` 有 `Clone` 特徵。

    impl<T: Clone> Option<&T> {
        pub fn cloned(self) -> Option<T> {
            self.map(|t| t.clone())
        }
    }

    impl<T: Clone> Option<&mut T> {
        pub fn cloned(self) -> Option<T> {
            self.map(|t| t.clone())
        }
    }

由於有強解參照，不用像上面的 [copied()](#orgd4b1fe7) 一樣特別去寫閉包參數。


<a id="orgca006ef"></a>

### <a id="org3a50e46"></a> `unwrap_or_default`

需要型態 `T` 有 `Default` 特徵。

    impl<T: Default> Option<T> {
        #[inline]
        pub fn unwrap_or_default(self) -> T {
            match self {
                Some(x) => x,
                None => Default::default(),
            }
        }
    }

非常簡單的一個 `match` 解決。


<a id="org634b2f3"></a>

### <a id="orgff3319c"></a> `as_deref`

需要型態 `T` 有 [Deref](https://doc.rust-lang.org/core/ops/trait.Deref.html) 特徵。

    impl<T: Deref> Option<T> {
        pub fn as_deref(&self) -> Option<&T::Target> {
            self.as_ref().map(|t| t.deref())
        }
    }

用 `as_ref()` 取得 `&T` 後，再用 `map()` 裡的 `deref()` 與強解參照，得到 `T` 型態的解參照型態 `&T::Target` 。


<a id="org372f263"></a>

### <a id="orgb48c42c"></a> `as_deref_mut`

需要型態 `T` 有 [DerefMut](https://doc.rust-lang.org/core/ops/trait.DerefMut.html) 特徵。

    impl<T: DerefMut> Option<T> {
        pub fn as_deref_mut(&mut self) -> Option<&mut T::Target> {
            self.as_mut().map(|t| t.deref_mut())
        }
    }

與上面接近，回傳型態不同。


<a id="orga2885cb"></a>

### <a id="org2dea7c4"></a> `transpose`

    impl<T, E> Option<Result<T, E>> {
        #[inline]
        pub fn transpose(self) -> Result<Option<T>, E> {
            match self {
                Some(Ok(x)) => Ok(Some(x)),
                Some(Err(e)) => Err(e),
                None => Ok(None),
            }
        }
    }

將 `Option<Result<T, E>>` 轉成 `Result<Option<T>, E>` ，用 `match` 做簡單的對應就解決了。


<a id="orgd66f45d"></a>

## Unstable


<a id="org2025453"></a>

### <a id="org50367a0"></a> `contains`

    #[must_use]
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

從定義看出，這段程式碼在使用了特徵綁定後不需限於同型態，只需要單向的 `PartialEq` 特徵即可。


<a id="org82eec6b"></a>

### <a id="org9d8d3b7"></a> `zip_with`

    #[unstable(feature = "option_zip", issue = "70086")]
    pub fn zip_with<U, F, R>(self, other: Option<U>, f: F) -> Option<R>
    where
        F: FnOnce(T, U) -> R,
    {
        Some(f(self?, other?))
    }

其中的 `?` 保證當自身或 `other` 其一是 `None` 時會直接回傳 `None` ，只有兩個都是 `Some` 時才會利用 `f` 將型態 `T` 與 `U` 映射到 `R` 上。


<a id="org80290e2"></a>

### <a id="org94e96ea"></a> `expect_none`

需要型態 `T` 有 [fmt::Debug](https://doc.rust-lang.org/core/fmt/trait.Debug.html) 特徵。

    impl<T: fmt::Debug> Option<T> {
        #[inline]
        #[track_caller]
        #[unstable(feature = "option_expect_none", reason = "newly added", issue = "62633")]
        pub fn expect_none(self, msg: &str) {
            if let Some(val) = self {
                expect_none_failed(msg, &val);
            }
        }
    }


<a id="org4bff605"></a>

### `expect_none_failed`

    impl<T: fmt::Debug> Option<T> {
        #[inline(never)]
        #[cold]
        #[track_caller]
        fn expect_none_failed(msg: &str, value: &dyn fmt::Debug) -> ! {
            panic!("{}: {:?}", msg, value)
        }
    }


<a id="orgfd98607"></a>

### <a id="orgc5f4eb1"></a> `unwrap_none`

需要型態 `T` 有 [fmt::Debug](https://doc.rust-lang.org/core/fmt/trait.Debug.html) 特徵。

    impl<T: fmt::Debug> Option<T> {
        #[inline]
        #[track_caller]
        #[unstable(feature = "option_unwrap_none", reason = "newly added", issue = "62633")]
        pub fn unwrap_none(self) {
            if let Some(val) = self {
                expect_none_failed("called `Option::unwrap_none()` on a `Some` value", &val);
            }
        }
    }


<a id="org2d44ece"></a>

### <a id="org14b1013"></a> `flatten`

需要為型態 `Option<Option<T>>` 。

    impl<T> Option<Option<T>> {
        #[inline]
        pub fn flatten(self) -> Option<T> {
            self.and_then(convert::identity)
        }
    }

[Use](#org57b7606) 用到了 [core::convert::identity](https://doc.rust-lang.org/core/convert/fn.identity.html) 這個函數，與 [and<sub>then</sub>()](#orgc376fee) 這個方法，讓 `Some(opt)` 用 `convert::identity()` 轉成 `opt` ，而 `None` 則因為 `and_then` 轉成 `None` ，最後型態是 `Option<T>` 。


<a id="org39356e6"></a>

# Unsafe Methods


<a id="org90a1f59"></a>

## Unstable


<a id="org2341902"></a>

### <a id="orga788d95"></a> `unwrap_unchecked`

    #[inline]
    #[track_caller]
    #[unstable(feature = "option_result_unwrap_unchecked", reason = "newly added", issue = "81383")]
    pub unsafe fn unwrap_unchecked(self) -> T {
        debug_assert!(self.is_some());
        match self {
            Some(val) => val,
            // SAFETY: the safety contract must be upheld by the caller.
            None => unsafe { hint::unreachable_unchecked() },
        }
    }

不檢查就直接把 `Option<T>` 轉換成 `T` 型態，如果裡面為 `None` 時為未定義行為。


<a id="orga5e2b3d"></a>

# Trait Implementations


<a id="org04967e6"></a>

## <a id="org351d2fc"></a> Clone

    impl<T: Clone> Clone for Option<T> {
        #[inline]
        fn clone(&self) -> Self {
            match self {
                Some(x) => Some(x.clone()),
                None => None,
            }
        }
    
        #[inline]
        fn clone_from(&mut self, source: &Self) {
            match (self, source) {
                (Some(to), Some(from)) => to.clone_from(from),
                (to, from) => *to = from.clone(),
            }
        }
    }

`clone()` 很簡單，就是用內容的 `clone()` 再在外面包一層 `Some` 。當 `clone_from()` 的兩個都是 `Some` 時也很簡單； `(Some, None)` 時用了 `Option` 內的 `clone()` 所以保證會複製到 `None` ； `(None, Some)` 時自身的值被 `Some` 裡面的值更新，所以也保證會複製到 `Some` ； `(None, None)` 時則保證會複製到 `None` 。


<a id="org9fa2e79"></a>

## <a id="org39a056f"></a> Default

    impl<T> Default for Option<T> {
        #[inline]
        fn default() -> Option<T> {
            None
        }
    }

預設值是 `None` 。所以不需要型態 `T` 有任何的 `Default` 實作。


<a id="orgb1a01c9"></a>

## <a id="orgd88eff4"></a> IntoIterator

    impl<T> IntoIterator for Option<T> {
        type Item = T;
        type IntoIter = IntoIter<T>;
    
        #[inline]
        fn into_iter(self) -> IntoIter<T> {
            IntoIter { inner: Item { opt: self } }
        }
    }

建立一個 [IntoIter](#org34b3010) 結構。

    impl<'a, T> IntoIterator for &'a Option<T> {
        type Item = &'a T;
        type IntoIter = Iter<'a, T>;
    
        fn into_iter(self) -> Iter<'a, T> {
            self.iter()
        }
    }

這邊直接使用了 [Iter](#org8e324a6) 結構。

    impl<'a, T> IntoIterator for &'a mut Option<T> {
        type Item = &'a mut T;
        type IntoIter = IterMut<'a, T>;
    
        fn into_iter(self) -> IterMut<'a, T> {
            self.iter_mut()
        }
    }

這邊直接使用了 [IterMut](#org42ed16c) 結構。


<a id="orgaf91253"></a>

## <a id="orgc6bc1ad"></a> From

    impl<T> From<T> for Option<T> {
        fn from(val: T) -> Option<T> {
            Some(val)
        }
    }

這邊會直接把 `val` 的所有權吃掉，必定轉成 `Some` 。

    impl<'a, T> From<&'a Option<T>> for Option<&'a T> {
        fn from(o: &'a Option<T>) -> Option<&'a T> {
            o.as_ref()
        }
    }

這邊會複製的是參照，生命期為 `o` 的參照來源。使用了 [as<sub>ref</sub>()](#orgbd44e20) 方法，所以可以將型態從 `&Option<T>` 轉為 `Option<&T>` 而且不複製到內容物本身。

    impl<'a, T> From<&'a mut Option<T>> for Option<&'a mut T> {
        fn from(o: &'a mut Option<T>) -> Option<&'a mut T> {
            o.as_mut()
        }
    }

這邊複製一個可變參照，生命期為 `o` 的參照來源。使用了 [as<sub>mut</sub>()](#org1fbeaf4) 方法，所以可以將型態從 `&mut Option<T>` 轉為 `Option<&mut T>` 而不複製內容易本身。


<a id="orgd232f4d"></a>

## <a id="org4658cef"></a> FromIterator

    
    impl<A, V: FromIterator<A>> FromIterator<Option<A>> for Option<V> {
        #[inline]
        fn from_iter<I: IntoIterator<Item = Option<A>>>(iter: I) -> Option<V> {
            // FIXME(#11084): This could be replaced with Iterator::scan when this
            // performance bug is closed.
    
            iter.into_iter().map(|x| x.ok_or(())).collect::<Result<_, _>>().ok()
        }
    }

註解裡提到的 [Iterator::scan](https://doc.rust-lang.org/core/iter/trait.Iterator.html#method.scan) 是創出新的迭代器的方法。參數 `iter` 的型態 `I` 必須要有 `IntoIterator` 中 `Item = Option<A>` 的實作；最後的回傳型態 `V` 則必須要有 `FromIterator` 中型態 `A` 的實作。首先是把參數 `iter` 用 `into_iter()` 轉成 `IntoIterator` ，再用 `map()` 將每個型態為 `Option<A>` 的元素 `x` 用 `ok_or()` 轉成型態 `Result<A, ()>` ，此時外面型態是 `Map<Self, F>` ，然後用 `IntoIterator` 的方法 `collect()` 做成 `Result<V, ()>` ，最後再用 `Result` 的方法 `ok()` 轉回 `Option<V>` 。


<a id="orgca43045"></a>

## <a id="org2c6b718"></a> ops::Try (Unstable)

    #[unstable(feature = "try_trait", issue = "42327")]
    impl<T> ops::Try for Option<T> {
        type Ok = T;
        type Error = NoneError;
    
        #[inline]
        fn into_result(self) -> Result<T, NoneError> {
            self.ok_or(NoneError)
        }
    
        #[inline]
        fn from_ok(v: T) -> Self {
            Some(v)
        }
    
        #[inline]
        fn from_error(_: NoneError) -> Self {
            None
        }
    }

這個以後可能用來取代 [ok<sub>or</sub>()](#org7bde697) 等等方法，以一個 `?` 就回傳 `Result` 型態。


<a id="org7adc0de"></a>

# Structs


<a id="org36ea248"></a>

## <a id="orga6c3fde"></a> Item


<a id="org53d8193"></a>

### Definition

    #[derive(Clone, Debug)]
    struct Item<A> {
        opt: Option<A>,
    }

[Iter](#org8e324a6) 內的欄位結構。


<a id="orgd176b6a"></a>

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
    
    實作上使用 [take()](#orgad04df5) 這個方法達成。 其中的 `size_hint()` 因為非常簡單所以用 `match` 來加速。

2.  DoubleEndedIterator

        impl<A> DoubleEndedIterator for Item<A> {
            #[inline]
            fn next_back(&mut self) -> Option<A> {
                self.opt.take()
            }
        }
    
    因為在內有東西時從前面往後看與從後往前是一樣的，所以與上面的 `next()` 寫法一樣即可。

3.  ExactSizeIterator

        impl<A> ExactSizeIterator for Item<A> {}

4.  FusedIterator

        impl<A> FusedIterator for Item<A> {}

5.  TrustedLen

        unsafe impl<A> TrustedLen for Item<A> {}


<a id="org8b017c8"></a>

## <a id="org8e324a6"></a> Iter

[iter()](#orgb3294f3) 所回傳的結構。


<a id="org68b2e04"></a>

### Definition

    #[derive(Debug)]
    pub struct Iter<'a, A: 'a> {
        inner: Item<&'a A>,
    }

借用的生命期為 `a` ，而結構本身的生命期也為 `a` 。


<a id="orgfa524d8"></a>

### Trait Implementations

1.  Iterator

        impl<'a, A> Iterator for Iter<'a, A> {
            type Item = &'a A;
        
            #[inline]
            fn next(&mut self) -> Option<&'a A> {
                self.inner.next()
            }
            #[inline]
            fn size_hint(&self) -> (usize, Option<usize>) {
                self.inner.size_hint()
            }
        }
    
    利用了另一個結構 [Item](#orga6c3fde) 簡化了麻煩的生命期標註，這邊只做呼叫內部的方法完成。

2.  DoubleEndedIterator

        impl<'a, A> DoubleEndedIterator for Iter<'a, A> {
            #[inline]
            fn next_back(&mut self) -> Option<&'a A> {
                self.inner.next_back()
            }
        }

3.  ExactSizeIterator

        impl<A> ExactSizeIterator for Iter<'_, A> {}

4.  FusedIterator

        impl<A> FusedIterator for Iter<'_, A> {}

5.  Clone

        impl<A> Clone for Iter<'_, A> {
            #[inline]
            fn clone(&self) -> Self {
                Iter { inner: self.inner.clone() }
            }
        }
    
    不使用 `derive` 巨集，而是明確呼叫 [Item](#orga6c3fde) 的 `clone()` 。

6.  TrustedLen (Unstable)

        #[unstable(feature = "trusted_len", issue = "37572")]
        unsafe impl<A> TrustedLen for Iter<'_, A> {}


<a id="org07e8dc1"></a>

## <a id="org42ed16c"></a> IterMut

[iter<sub>mut</sub>()](#org66f4815) 所回傳的結構。


<a id="org4a3a821"></a>

### Definition

    #[derive(Debug)]
    pub struct IterMut<'a, A: 'a> {
        inner: Item<&'a mut A>,
    }

與上方的[Iter](#org8e324a6) 不同的是多了一個 `mut` 借用。


<a id="org0eb1850"></a>

### Trait Implementations

1.  Iterator

        impl<'a, A> Iterator for IterMut<'a, A> {
            type Item = &'a mut A;
        
            #[inline]
            fn next(&mut self) -> Option<&'a mut A> {
                self.inner.next()
            }
            #[inline]
            fn size_hint(&self) -> (usize, Option<usize>) {
                self.inner.size_hint()
            }
        }
    
    所有的實作都交由 [Item](#orga6c3fde) 完成， [Iter](#org8e324a6) 與 [IterMut](#org42ed16c) 只負責做出泛型的不同借用，非常高的抽象程度。

2.  DoubleEndedIterator

        impl<'a, A> DoubleEndedIterator for IterMut<'a, A> {
            #[inline]
            fn next_back(&mut self) -> Option<&'a mut A> {
                self.inner.next_back()
            }
        }

3.  ExactSizeIterator

        impl<A> ExactSizeIterator for IterMut<'_, A> {}

4.  FusedIterator

        impl<A> FusedIterator for IterMut<'_, A> {}

5.  TrustedLen (Unstable)

        #[unstable(feature = "trusted_len", issue = "37572")]
        unsafe impl<A> TrustedLen for IterMut<'_, A> {}


<a id="orgc09ea42"></a>

## <a id="org34b3010"></a> IntoIter

[IntoIterator](#orgd88eff4) 所回傳的結構。


<a id="org4ecede6"></a>

### Definition

    #[derive(Clone, Debug)]
    pub struct IntoIter<A> {
        inner: Item<A>,
    }


<a id="org61b5ff4"></a>

### Trait Implementations

1.  Iterator

        impl<A> Iterator for IntoIter<A> {
            type Item = A;
        
            #[inline]
            fn next(&mut self) -> Option<A> {
                self.inner.next()
            }
            #[inline]
            fn size_hint(&self) -> (usize, Option<usize>) {
                self.inner.size_hint()
            }
        }
    
    與前面都一樣，利用了共同的內容結構 [Item](#orga6c3fde) 的實作完成特徵的實作。

2.  DoubleEndedIterator

        impl<A> DoubleEndedIterator for IntoIter<A> {
            #[inline]
            fn next_back(&mut self) -> Option<A> {
                self.inner.next_back()
            }
        }

3.  ExactSizeIterator

        impl<A> ExactSizeIterator for IntoIter<A> {}

4.  FusedIterator

        impl<A> FusedIterator for IntoIter<A> {}

5.  TrustedLen (Unstable)

        #[unstable(feature = "trusted_len", issue = "37572")]
        unsafe impl<A> TrustedLen for IntoIter<A> {}


<a id="orgf8b1d1a"></a>

## <a id="orgb0f6e5e"></a> NoneError (Unstable)

    #[unstable(feature = "try_trait", issue = "42327")]
    #[derive(Clone, Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
    pub struct NoneError;


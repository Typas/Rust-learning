
# Table of Contents

1.  [Use](#orgebb7d02)
2.  [Definition](#orgd407e53)
3.  [Associated Functions](#org92561f6)
4.  [Const Methods](#org2e8165f)
    1.  [Stable, but unstable const, issue 67441](#org0720fc9)
        1.  [`is_some`](#orgf7b5b70)
        2.  [`is_none`](#org4d06357)
        3.  [`unwrap`](#org828978c)
        4.  [`iter`](#org2f8c16c)
5.  [Safe Methods](#org0679317)
    1.  [Stable](#orgdb445e0)
        1.  [`as_ref`](#org9f5e8d6)
        2.  [`as_mut`](#orge2f9381)
        3.  [`as_pin_ref`](#org60f59de)
        4.  [`as_pin_mut`](#orga2e1f49)
        5.  [`expect`](#orgf612e70)
        6.  [`expect_failed`](#orgc88fca0)
        7.  [`unwrap_or`](#org7c3677d)
        8.  [`unwrap_or_else`](#orge722dc8)
        9.  [`map`](#org2aeb92c)
        10. [`map_or`](#org3d9be07)
        11. [`map_or_else`](#orgd408fca)
        12. [`ok_or`](#orga5c2ab9)
        13. [`ok_or_else`](#org1cd1fe9)
        14. [`iter_mut`](#orgdf6b4f8)
        15. [`and`](#org6905454)
        16. [`and_then`](#orga85fae4)
        17. [`filter`](#orgb9b3c4c)
        18. [`or`](#orgfc33591)
        19. [`or_else`](#orga6f34f5)
        20. [`xor`](#org2360269)
        21. [`get_or_insert`](#org3c2c455)
        22. [`get_or_insert_with`](#org3a37fd0)
        23. [`take`](#org6926e76)
        24. [`replace`](#orgd19ec31)
        25. [`zip`](#org0a49fff)
        26. [`copied`](#org034771b)
        27. [`cloned`](#org6338b9b)
        28. [`unwrap_or_default`](#orgc64124f)
        29. [`as_deref`](#org6641f18)
        30. [`as_deref_mut`](#org14072a8)
        31. [`transpose`](#orgc80a95f)
    2.  [Unstable](#org06eb209)
        1.  [`contains`](#org4c72238)
        2.  [`zip_with`](#org5db8cc4)
        3.  [`expect_none`](#org59452bf)
        4.  [`expect_none_failed`](#org9fbde6d)
        5.  [`unwrap_none`](#orgc54cfc1)
        6.  [`flatten`](#org8399be7)
6.  [Unsafe Methods](#orgc4b10ac)
7.  [Trait Implementations](#org20a0349)
    1.  [Clone](#org20b5099)
    2.  [Default](#orgdb6d882)
    3.  [IntoIterator](#orgba5baa3)
    4.  [From](#orge1a9cbf)
    5.  [FromIterator](#org5326fdd)
    6.  [ops::Try (Unstable)](#orgb772959)
8.  [Structs](#orgb88569e)
    1.  [Item](#org25125d5)
        1.  [Definition](#orgeb3401c)
        2.  [Trait Implementations](#org840259e)
    2.  [Iter](#orgb127045)
        1.  [Definition](#org6022e0e)
        2.  [Trait Implementations](#orgd9103a9)
    3.  [IterMut](#orgcfb31b8)
        1.  [Definition](#org18ba0bf)
        2.  [Trait Implementations](#orgfc24096)
    4.  [IntoIter](#org209937a)
        1.  [Definition](#org7ca4286)
        2.  [Trait Implementations](#orga99761a)
    5.  [NoneError (Unstable)](#orgbe8dad7)

[Source Code Location](https://github.com/rust-lang/rust/blob/master/library/core/src/option.rs)


<a id="orgebb7d02"></a>

# <a id="org1db1c36"></a> Use

    use core::iter::{FromIterator, FusedIterator, TrustedLen};
    use core::pin::Pin;
    use core::{
        convert, fmt, hint, mem,
        ops::{self, Deref, DerefMut},
    };


<a id="orgd407e53"></a>

# Definition

    #[derive(Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
    pub enum Option<T> {
        /// No value
        None,
        /// Some value `T`
        Some(T),
    }

在去除一些給編譯器看的屬性巨集後，是非常簡單的定義。


<a id="org92561f6"></a>

# Associated Functions


<a id="org2e8165f"></a>

# Const Methods


<a id="org0720fc9"></a>

## Stable, but unstable const, issue 67441


<a id="orgf7b5b70"></a>

### <a id="org49682a4"></a> `is_some`

    #[inline]
    pub const fn is_some(&self) -> bool {
        matches!(*self, Some(_))
    }

用到了 [matches](https://doc.rust-lang.org/std/macro.matches.html) 巨集，在對應到任何有值的東西時回傳 `true` ，反之則回傳 `false` 。


<a id="org4d06357"></a>

### <a id="orge49c4be"></a> `is_none`

    #[inline]
    pub const fn is_none(&self) -> bool {
        !self.is_some()
    }

上面的 `is_some` 的反轉，只是我不太能理解為什麼是用 `Some` 而不是 `None` 去對應。


<a id="org828978c"></a>

### <a id="org08c9961"></a> `unwrap`

    #[inline]
    #[track_caller]
    pub const fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }

就是一個在 `None` 會噴死你的函數，會拿走所有權。


<a id="org2f8c16c"></a>

### <a id="org498d766"></a> `iter`

    #[inline]
    pub const fn iter(&self) -> Iter<'_, T> {
        Iter { inner: Item { opt: self.as_ref() } }
    }

回傳的型態 [Iter](#orgb1fd13a) 是模組內定義的一個結構。


<a id="org0679317"></a>

# Safe Methods


<a id="orgdb445e0"></a>

## Stable


<a id="org9f5e8d6"></a>

### <a id="org372dd0d"></a> `as_ref`

    #[inline]
    pub const fn as_ref(&self) -> Option<&T> {
        match *self {
            Some(ref x) => Some(x),
            None => None,
        }
    }

用到了 `match` 裡的 `ref` 關鍵字，配對的值本身型態是 `U` 時，加上 `ref` 則在分支內的變數 `x` 會是 `&U` 型態。


<a id="orge2f9381"></a>

### <a id="orga8f6f30"></a> `as_mut`

    #[inline]
    pub fn as_mut(&mut self) -> Option<&mut T> {
        match *self {
            Some(ref mut x) => Some(x),
            None => None,
        }
    }

同上，多用了一個 `mut` 關鍵字，型態改變成 `&mut U` 。


<a id="org60f59de"></a>

### <a id="org94f2e98"></a> `as_pin_ref`

    #[inline]
    pub fn as_pin_ref(self: Pin<&Self>) -> Option<Pin<&T>> {
        // SAFETY: `x` is guaranteed to be pinned because it comes from `self`
        // which is pinned.
        unsafe { Pin::get_ref(self).as_ref().map(|x| Pin::new_unchecked(x)) }
    }

[use](#org1db1c36)
TODO: usage of `core::pin::Pin`


<a id="orga2e1f49"></a>

### <a id="org61117c5"></a> `as_pin_mut`

    #[inline]
    pub fn as_pin_mut(self: Pin<&mut Self>) -> Option<Pin<&mut T>> {
        // SAFETY: `get_unchecked_mut` is never used to move the `Option` inside `self`.
        // `x` is guaranteed to be pinned because it comes from `self` which is pinned.
        unsafe { Pin::get_unchecked_mut(self).as_mut().map(|x| Pin::new_unchecked(x)) }
    }

[use](#org1db1c36)
TODO: usage of `core::pin::Pin`


<a id="orgf612e70"></a>

### <a id="org88d10ea"></a> `expect`

    #[inline]
    #[track_caller]
    pub fn expect(self, msg: &str) -> T {
        match self {
            Some(val) => val,
            None => expect_failed(msg),
        }
    }

裡面的 `expect_failed()` 是私有方法，在對應到 `None` 時會觸發。會拿走所有權。


<a id="orgc88fca0"></a>

### `expect_failed`

    #[inline(never)]
    #[cold]
    #[track_caller]
    fn expect_failed(msg: &str) -> ! {
        panic!("{}", msg)
    }

屬性巨集 `cold` 是對編譯器提示這個函數不太可能被呼叫到。問題：為何會需要獨立成一個方法？


<a id="org7c3677d"></a>

### <a id="org036c942"></a> `unwrap_or`

    #[inline]
    pub fn unwrap_or(self, default: T) -> T {
        match self {
            Some(x) => x,
            None => default,
        }
    }

請注意這裡會拿走自身與 `default` 的所有權。


<a id="orge722dc8"></a>

### <a id="org6b348a1"></a> `unwrap_or_else`

    #[inline]
    pub fn unwrap_or_else<F: FnOnce() -> T>(self, f: F) -> T {
        match self {
            Some(x) => x,
            None => f(),
        }
    }

裡面的函數 `f` 會拿走所有內部變數的所有權，自身的所有權也會被此方法拿走。


<a id="org2aeb92c"></a>

### <a id="org8f55d55"></a> `map`

    #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn map<U, F: FnOnce(T) -> U>(self, f: F) -> Option<U> {
        match self {
            Some(x) => Some(f(x)),
            None => None,
        }
    }

利用 `f` 將自身映射到 `f(x)` 上，此方法會拿走自身的所有權。


<a id="org3d9be07"></a>

### <a id="org70ae627"></a> `map_or`

    #[inline]
    pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default,
        }
    }

與 [unwrap<sub>or</sub>](#org036c942) 相似，但 `default` 的型態為映射後的 `U` 而非原來的 `T` 。


<a id="orgd408fca"></a>

### <a id="org9e1fdba"></a> `map_or_else`

    #[inline]
    pub fn map_or_else<U, D: FnOnce() -> U, F: FnOnce(T) -> U>(self, default: D, f: F) -> U {
        match self {
            Some(t) => f(t),
            None => default(),
        }
    }

與 [unwrap<sub>or</sub><sub>else</sub>](#org6b348a1) 相似，但型態為映射後的 `U` 。


<a id="orga5c2ab9"></a>

### <a id="orgcb6b289"></a> `ok_or`

    #[inline]
    pub fn ok_or<E>(self, err: E) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err),
        }
    }

把 `Option<T>` 型態轉成 `Result<T, E>` 型態的神方法，需要自行加上錯誤型態，兩個參數都會被拿走所有權。


<a id="org1cd1fe9"></a>

### <a id="org4913080"></a> `ok_or_else`

    #[inline]
    pub fn ok_or_else<E, F: FnOnce() -> E>(self, err: F) -> Result<T, E> {
        match self {
            Some(v) => Ok(v),
            None => Err(err()),
        }
    }

從 `ok_or` 的錯誤參數改成使用一個回傳錯誤的函數。會拿走自身的所有權。


<a id="orgdf6b4f8"></a>

### <a id="org4a034db"></a> `iter_mut`

    #[inline]
    pub fn iter_mut(&mut self) -> IterMut<'_, T> {
        IterMut { inner: Item { opt: self.as_mut() } }
    }

利用新結構 [IterMut](#org1d187d1) 來達成特徵實作的隔離。


<a id="org6905454"></a>

### <a id="org03b7b96"></a> `and`

    #[inline]
    pub fn and<U>(self, optb: Option<U>) -> Option<U> {
        match self {
            Some(_) => optb,
            None => None,
        }
    }

與不同型態做邏輯的「且」運算，注意是回傳 `optb` 的結果，兩方的所有權都會被轉移。


<a id="orga85fae4"></a>

### <a id="org3153524"></a> `and_then`

    #[inline]
    #[stable(feature = "rust1", since = "1.0.0")]
    pub fn and_then<U, F: FnOnce(T) -> Option<U>>(self, f: F) -> Option<U> {
        match self {
            Some(x) => f(x),
            None => None,
        }
    }

上方邏輯「且」運算中，將參數 `optb` 改為函數 `f` 的方法。非常類似於 [map()](#org8f55d55) ，但是在函數的定義上不同。 `map()` 使用的函數為回傳型態 `U` 的一次性函數，而 `and_then()` 則是使用回傳型態為 `Option<U>` 的一次性函數。


<a id="orgb9b3c4c"></a>

### <a id="org709077c"></a> `filter`

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


<a id="orgfc33591"></a>

### <a id="org59f32e7"></a> `or`

    #[inline]
    pub fn or(self, optb: Option<T>) -> Option<T> {
        match self {
            Some(_) => self,
            None => optb,
        }
    }

與同型態做邏輯的「或」運算，兩個參數都會被拿走所有權。在自身有值時會回傳自身，反之回傳 `optb` 。


<a id="orga6f34f5"></a>

### <a id="orge57e1dc"></a> `or_else`

    #[inline]
    pub fn or_else<F: FnOnce() -> Option<T>>(self, f: F) -> Option<T> {
        match self {
            Some(_) => self,
            None => f(),
        }
    }

上方邏輯「或」運算中，把 `optb` 以 `f` 這個函數取代掉的方法。注意「且」能回傳不同型態，而「或」只能回傳同一型態。


<a id="org2360269"></a>

### <a id="org5de4ed9"></a> `xor`

    #[inline]
    pub fn xor(self, optb: Option<T>) -> Option<T> {
        match (self, optb) {
            (Some(a), None) => Some(a),
            (None, Some(b)) => Some(b),
            _ => None,
        }
    }

與同型態做邏輯的「互斥或」運算，兩個參數都會被拿走所有權。兩者皆有或皆無值時回傳 `None` ，其一有值時回傳有值的一邊。


<a id="org3c2c455"></a>

### <a id="org84caffb"></a> `get_or_insert`

    #[inline]
    pub fn get_or_insert(&mut self, v: T) -> &mut T {
        self.get_or_insert_with(|| v)
    }

以一個簡單的閉包直接把 `v` 轉移進去 [get<sub>or</sub><sub>insert</sub><sub>with</sub>()](#org4e06267) ，避免了重複的程式碼。


<a id="org3a37fd0"></a>

### <a id="org4e06267"></a> `get_or_insert_with`

    #[inline]
    pub fn get_or_insert_with<F: FnOnce() -> T>(&mut self, f: F) -> &mut T {
        if let None = *self {
            *self = Some(f());
        }
    
        match *self {
            Some(ref mut v) => v,
            // SAFETY: a `None` variant for `self` would have been replaced by a `Some`
            // variant in the code above.
            None => unsafe { hint::unreachable_unchecked() },
        }
    }

第一段程式碼將所有的 `None` 轉換成 `Some(f())` 。第二段程式碼中利用 `match` 的 `ref mut` 把 `T` 轉成 `&mut T` 後回傳，其中 `None` 分支由第一段保證不會被執行到，因此用一段 SAFETY 註解標示為何使用到 `unsafe` 。
此處有用到 [core::hint](#org1db1c36) 。


<a id="org6926e76"></a>

### <a id="orgf18ffd9"></a> `take`

    #[inline]
    pub fn take(&mut self) -> Option<T> {
        mem::take(self)
    }

此處用到 [core::mem](#org1db1c36) 的 [take()](https://doc.rust-lang.org/core/mem/fn.take.html) 。這個函數需要 `Option` 自身的 [Default](#org8218958) 特徵實作，會將內部型態為 `T` 的值以預設值取代，並回傳被取代的值。若這個型態沒有標註為 `Copy` 特徵，則回傳會拿走所有權。


<a id="orgd19ec31"></a>

### <a id="org8a69aa4"></a> `replace`

    #[inline]
    pub fn replace(&mut self, value: T) -> Option<T> {
        mem::replace(self, Some(value))
    }

此處用到 [core::mem](#org1db1c36) 的 [replace()](https://doc.rust-lang.org/core/mem/fn.replace.html) 。這個函數會將內部的值以 `value` 取代，並回傳原本的值。


<a id="org0a49fff"></a>

### <a id="org0645e76"></a> `zip`

    pub fn zip<U>(self, other: Option<U>) -> Option<(T, U)> {
        match (self, other) {
            (Some(a), Some(b)) => Some((a, b)),
            _ => None,
        }
    }

這個方法會在兩個值都是 `Some` 的時候綁成一個元組，其餘都會回傳 `None` 。概念上是「且」運算，注意兩個所有權都會被吃掉。


<a id="org034771b"></a>

### <a id="orgd059251"></a> `copied`

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


<a id="org6338b9b"></a>

### <a id="org657aefd"></a> `cloned`

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

由於有強解參照，不用像上面的 [copied()](#orgd059251) 一樣特別去寫閉包參數。


<a id="orgc64124f"></a>

### <a id="org98bf402"></a> `unwrap_or_default`

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


<a id="org6641f18"></a>

### <a id="org493dbf4"></a> `as_deref`

需要型態 `T` 有 [Deref](https://doc.rust-lang.org/core/ops/trait.Deref.html) 特徵。

    impl<T: Deref> Option<T> {
        pub fn as_deref(&self) -> Option<&T::Target> {
            self.as_ref().map(|t| t.deref())
        }
    }

用 `as_ref()` 取得 `&T` 後，再用 `map()` 裡的 `deref()` 與強制解參，得到 `T` 型態的解參照型態 `&T::Target` 。


<a id="org14072a8"></a>

### <a id="org5f353bc"></a> `as_deref_mut`

需要型態 `T` 有 [DerefMut](https://doc.rust-lang.org/core/ops/trait.DerefMut.html) 特徵。

    impl<T: DerefMut> Option<T> {
        pub fn as_deref_mut(&mut self) -> Option<&mut T::Target> {
            self.as_mut().map(|t| t.deref_mut())
        }
    }

與上面接近，回傳型態不同。


<a id="orgc80a95f"></a>

### <a id="orged477a5"></a> `transpose`

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


<a id="org06eb209"></a>

## Unstable


<a id="org4c72238"></a>

### <a id="orgb198927"></a> `contains`

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


<a id="org5db8cc4"></a>

### <a id="orge22cd1c"></a> `zip_with`

    #[unstable(feature = "option_zip", issue = "70086")]
    pub fn zip_with<U, F, R>(self, other: Option<U>, f: F) -> Option<R>
    where
        F: FnOnce(T, U) -> R,
    {
        Some(f(self?, other?))
    }

其中的 `?` 保證當自身或 `other` 其一是 `None` 時會直接回傳 `None` ，只有兩個都是 `Some` 時才會利用 `f` 將型態 `T` 與 `U` 映射到 `R` 上。


<a id="org59452bf"></a>

### <a id="org9e33e0e"></a> `expect_none`

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


<a id="org9fbde6d"></a>

### `expect_none_failed`

    impl<T: fmt::Debug> Option<T> {
        #[inline(never)]
        #[cold]
        #[track_caller]
        fn expect_none_failed(msg: &str, value: &dyn fmt::Debug) -> ! {
            panic!("{}: {:?}", msg, value)
        }
    }


<a id="orgc54cfc1"></a>

### <a id="org1641baf"></a> `unwrap_none`

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


<a id="org8399be7"></a>

### <a id="org7358bb1"></a> `flatten`

需要為型態 `Option<Option<T>>` 。

    impl<T> Option<Option<T>> {
        #[inline]
        pub fn flatten(self) -> Option<T> {
            self.and_then(convert::identity)
        }
    }

[Use](#org1db1c36) 用到了 [core::convert::identity](https://doc.rust-lang.org/core/convert/fn.identity.html) 這個函數，與 [and<sub>then</sub>()](#org3153524) 這個方法，讓 `Some(opt)` 用 `convert::identity()` 轉成 `opt` ，而 `None` 則因為 `and_then` 轉成 `None` ，最後型態是 `Option<T>` 。


<a id="orgc4b10ac"></a>

# Unsafe Methods

沒有這種東西。


<a id="org20a0349"></a>

# Trait Implementations


<a id="org20b5099"></a>

## <a id="orge6ea26b"></a> Clone

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


<a id="orgdb6d882"></a>

## <a id="org8218958"></a> Default

    impl<T> Default for Option<T> {
        #[inline]
        fn default() -> Option<T> {
            None
        }
    }

預設值是 `None` 。所以不需要型態 `T` 有任何的 `Default` 實作。


<a id="orgba5baa3"></a>

## <a id="org1896a78"></a> IntoIterator

    impl<T> IntoIterator for Option<T> {
        type Item = T;
        type IntoIter = IntoIter<T>;
    
        #[inline]
        fn into_iter(self) -> IntoIter<T> {
            IntoIter { inner: Item { opt: self } }
        }
    }

建立一個 [IntoIter](#org5792f25) 結構，會把所有權拿走。

    impl<'a, T> IntoIterator for &'a Option<T> {
        type Item = &'a T;
        type IntoIter = Iter<'a, T>;
    
        fn into_iter(self) -> Iter<'a, T> {
            self.iter()
        }
    }

這邊直接使用了 [Iter](#orgb1fd13a) 結構。

    impl<'a, T> IntoIterator for &'a mut Option<T> {
        type Item = &'a mut T;
        type IntoIter = IterMut<'a, T>;
    
        fn into_iter(self) -> IterMut<'a, T> {
            self.iter_mut()
        }
    }

這邊直接使用了 [IterMut](#org1d187d1) 結構。


<a id="orge1a9cbf"></a>

## <a id="orga0ddeeb"></a> From

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

這邊會複製的是參照，生命期為 `o` 的參照來源。使用了 [as<sub>ref</sub>()](#org372dd0d) 方法，所以可以將型態從 `&Option<T>` 轉為 `Option<&T>` 而且不複製到內容物本身。

    impl<'a, T> From<&'a mut Option<T>> for Option<&'a mut T> {
        fn from(o: &'a mut Option<T>) -> Option<&'a mut T> {
            o.as_mut()
        }
    }

這邊複製一個可變參照，生命期為 `o` 的參照來源。使用了 [as<sub>mut</sub>()](#orga8f6f30) 方法，所以可以將型態從 `&mut Option<T>` 轉為 `Option<&mut T>` 而不複製內容易本身。


<a id="org5326fdd"></a>

## <a id="orgd1cd6d1"></a> FromIterator

    
    impl<A, V: FromIterator<A>> FromIterator<Option<A>> for Option<V> {
        #[inline]
        fn from_iter<I: IntoIterator<Item = Option<A>>>(iter: I) -> Option<V> {
            // FIXME(#11084): This could be replaced with Iterator::scan when this
            // performance bug is closed.
    
            iter.into_iter().map(|x| x.ok_or(())).collect::<Result<_, _>>().ok()
        }
    }

註解裡提到的 [Iterator::scan](https://doc.rust-lang.org/core/iter/trait.Iterator.html#method.scan) 是創出新的迭代器的方法。參數 `iter` 的型態 `I` 必須要有 `IntoIterator` 中 `Item = Option<A>` 的實作；最後的回傳型態 `V` 則必須要有 `FromIterator` 中型態 `A` 的實作。首先是把參數 `iter` 用 `into_iter()` 轉成 `IntoIterator` ，再用 `map()` 將每個型態為 `Option<A>` 的元素 `x` 用 `ok_or()` 轉成型態 `Result<A, ()>` ，此時外面型態是 `Map<Self, F>` ，然後用 `IntoIterator` 的方法 `collect()` 做成 `Result<V, ()>` ，最後再用 `Result` 的方法 `ok()` 轉回 `Option<V>` 。


<a id="orgb772959"></a>

## <a id="org745c02a"></a> ops::Try (Unstable)

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

這個以後可能用來取代 [ok<sub>or</sub>()](#orgcb6b289) 等等方法，以一個 `?` 就回傳 `Result` 型態。


<a id="orgb88569e"></a>

# Structs


<a id="org25125d5"></a>

## <a id="org346801e"></a> Item


<a id="orgeb3401c"></a>

### Definition

    #[derive(Clone, Debug)]
    struct Item<A> {
        opt: Option<A>,
    }

[Iter](#orgb1fd13a) 內的欄位結構。


<a id="org840259e"></a>

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
    
    實作上使用 [take()](#orgf18ffd9) 這個方法達成。 其中的 `size_hint()` 因為非常簡單所以用 `match` 來加速。

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


<a id="orgb127045"></a>

## <a id="orgb1fd13a"></a> Iter

[iter()](#org498d766) 所回傳的結構。


<a id="org6022e0e"></a>

### Definition

    #[derive(Debug)]
    pub struct Iter<'a, A: 'a> {
        inner: Item<&'a A>,
    }

借用的生命期為 `a` ，而結構本身的生命期也為 `a` 。


<a id="orgd9103a9"></a>

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
    
    利用了另一個結構 [Item](#org346801e) 簡化了麻煩的生命期標註，這邊只做呼叫內部的方法完成。

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
    
    不使用 `derive` 巨集，而是明確呼叫 [Item](#org346801e) 的 `clone()` 。

6.  TrustedLen (Unstable)

        #[unstable(feature = "trusted_len", issue = "37572")]
        unsafe impl<A> TrustedLen for Iter<'_, A> {}


<a id="orgcfb31b8"></a>

## <a id="org1d187d1"></a> IterMut

[iter<sub>mut</sub>()](#org4a034db) 所回傳的結構。


<a id="org18ba0bf"></a>

### Definition

    #[derive(Debug)]
    pub struct IterMut<'a, A: 'a> {
        inner: Item<&'a mut A>,
    }

與上方的[Iter](#orgb1fd13a) 不同的是多了一個 `mut` 借用。


<a id="orgfc24096"></a>

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
    
    所有的實作都交由 [Item](#org346801e) 完成， [Iter](#orgb1fd13a) 與 [IterMut](#org1d187d1) 只負責做出泛型的不同借用，非常高的抽象程度。

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

        unsafe impl<A> TrustedLen for IterMut<'_, A> {}


<a id="org209937a"></a>

## <a id="org5792f25"></a> IntoIter

[IntoIterator](#org1896a78) 所回傳的結構。


<a id="org7ca4286"></a>

### Definition

    #[derive(Clone, Debug)]
    pub struct IntoIter<A> {
        inner: Item<A>,
    }


<a id="orga99761a"></a>

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
    
    與前面都一樣，利用了共同的內容結構 [Item](#org346801e) 的實作完成特徵的實作。

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

5.  TrustedLen

        #[unstable(feature = "trusted_len", issue = "37572")]
        unsafe impl<A> TrustedLen for IntoIter<A> {}


<a id="orgbe8dad7"></a>

## <a id="org0999581"></a> NoneError (Unstable)

    #[unstable(feature = "try_trait", issue = "42327")]
    #[derive(Clone, Copy, PartialEq, PartialOrd, Eq, Ord, Debug, Hash)]
    pub struct NoneError;


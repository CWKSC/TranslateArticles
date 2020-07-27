---
date: 2020-07-27 19:00:00
layout: post
title: "CIS 194 10 Applicative functors, Part I 適用函子，第一部分"
subtitle: 
description: 
image: 
optimized_image: 
category: CIS 194 spring 2013
tags:
  - CIS 194 spring 2013
  - Haskell
author: CWKSC
paginate: false
math: true
---

Source: [10-applicative](https://www.seas.upenn.edu/~cis194/spring13/lectures/10-applicative.html)

## ▌Applicative functors, Part I 適用函子，第一部分

CIS 194 第 10 週
2012 年 3 月 25 日

建議閱讀：

- [Applicative Functors](http://learnyouahaskell.com/functors-applicative-functors-and-monoids#applicative-functors) from Learn You a Haskell
- [The Typeclassopedia](http://www.haskell.org/haskellwiki/Typeclassopedia)

## ▌Motivation 動機

考慮以下 `Employee` 類型：

```haskell
type Name = String

data Employee = Employee { name    :: Name
                           , phone   :: String }
                  deriving Show
```

`Employee` 構造函數具有類型

```haskell
Employee :: Name -> String -> Employee
```

也就是說，如果我們具有 `Name` 和 `String`，則可以應用 `Employee` 構造函數來構建 `Employee` 對象

但是，假設我們沒有 `Name` 和 `String` 。我們實際上擁有的可能是 `Maybe Name` 和 `Maybe String` 。也許它們來自解析充滿錯誤的某些文件，或者來自某些字段可能留為空白的形式，或者來自某種形式。 我們不一定能做出 `Employee`。 但可以肯定的是，我們可以做出 `Maybe Employee`。 也就是說，我們想使用 `(Name -> String -> Employee)` 函數並將其轉換為 `(Maybe Name -> Maybe String -> Maybe Employee)` 函數。 我們可以用這種類型寫某些東西嗎？

```haskell
(Name -> String -> Employee) ->
(Maybe Name -> Maybe String -> Maybe Employee)
```

當然可以，我完全相信您現在就可以在睡眠中寫下它。我們可以想像它是如何工作的：如果名稱或字符串是 `Nothing` ，我們得到 `Nothing`；如果兩者都為 `Just` ，我們得到一個使用 `Employee` 構造函數（包裝在 `Just` 中）構建的 `Employee`。 但是，讓我們繼續前進 ...

考慮一下：現在有了 `[Name]` 和 `[String]`，而不是 `Name` 和 `String`。 也許我們可以從中獲得一名 `[Employee]`？ 現在我們要

```haskell
(Name -> String -> Employee) ->
([Name] -> [String] -> [Employee])
```

我們可以想像兩種不同的工作方式：我們可以將相應的 `Name`s 和 `String`s 匹配起來組成 `Employee`s ； 或者我們可以通過所有可能的方式將 `Name` 和 `String` 配對

或如何處理：對於某些類型 `e`，我們有一個 `(e -> Name)` 和 `(e -> String)`。 例如，也許 `e` 是一些巨大的數據結構，並且我們有函數告訴我們如何從中提取 `Name` 和 `String`。 我們能否將其變成 `(e -> Employee)`，即從相同結構中提取 `Employee` 的配方？

```haskell
(Name -> String -> Employee) ->
((e -> Name) -> (e -> String) -> (e -> Employee))
```

沒問題，這一次實際上只有一種編寫此函數的方法

## ▌Generalizing 泛化

現在，我們已經看到了這種模式的實用性，讓我們概括一下。 我們想要的函數類型實際上看起來像這樣：

```haskell
(a -> b -> c) -> (f a -> f b -> f c)
```

嗯，這看起來很熟悉 …… 與 `fmap` 的類型非常相似！

```haskell
fmap :: (a -> b) -> (f a -> f b)
```

唯一的區別是一個額外的參數。我們可以調用所需的函數 `fmap2`，因為它帶有兩個參數的函數。 也許我們可以用 `fmap` 來寫 `fmap2` ，所以我們只需要對 `f` 施加 `Functor` 約束：

```haskell
fmap2 :: Functor f => (a -> b -> c) -> (f a -> f b -> f c)
fmap2 h fa fb = undefined
```

盡我們所能，但是 `Functor` 並沒有給我們足夠的力量來實現 `fmap2`。 怎麼了？ 我們有

```haskell
h  :: a -> b -> c
fa :: f a
fb :: f b
```

注意，我們也可以將 `h` 的類型寫為 `a -> (b -> c)` 。 因此，我們有一個採用 `a` 的函數，並且我們有一個類型為 `f a` 的值 … 我們唯一能做的就是使用 `fmap` 將函數移到 `f` 之上，從而得到類型為結果：

```haskell
h         :: a -> (b -> c)
fmap h    :: f a -> f (b -> c)
fmap h fa :: f (b -> c)
```

好的，現在我們有了 `f (b -> c)` 類型的東西和 `f b` 類型的東西 …… 這就是我們被困住的地方！ `fmap` 不再有幫助。 它為我們提供了一種將函數應用於 `Functor` 上下文中的值的方法，但是我們現在需要的是將本身在 `Functor` 上下文中的函數應用於 `Functor` 上下文中的值

## ▌Applicative 

可能進行這種 “上下文應用” 的函子稱為 *applicative*，並且 `Applicative` 類（在 [`Control.Applicative`](http://haskell.org/ghc/docs/latest/html/libraries/base/Control-Applicative.html) 中定義）捕獲了這種模式

```haskell
class Functor f => Applicative f where
  pure  :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
```

`(<*>)` 運算符（通常發音為“ ap”，是“ apply”的縮寫）完全封裝了 “上下文應用程序” 這一原理。 還要注意， `Applicative` 類也要求其實例也是 `Functor` 的實例，因此我們始終可以將 `fmap` 與 `Applicative` 的實例一起使用。 最後，請注意，`Applicative` 還具有另一種方法 `pure` ，該方法使我們可以將類型 `a` 的值注入到容器中。 現在，有趣的是，`fmap0` 是 `pure` 的另一個合理名稱：

```haskell
pure  :: a             -> f a
fmap  :: (a -> b)      -> f a -> f b
fmap2 :: (a -> b -> c) -> f a -> f b -> f c
```

有了 `(<*>)` 之後，我們就可以實現 `fmap2` 了，它在標準庫中實際上稱為 `liftA2`：

```haskell
liftA2 :: Applicative f => (a -> b -> c) -> f a -> f b -> f c
liftA2 h fa fb = (h `fmap` fa) <*> fb
```

實際上，這種模式非常普遍，因此 `Control.Applicative` 定義 `(<$>)` 為 `fmap` 的同義詞

```haskell
(<$>) :: Functor f => (a -> b) -> f a -> f b
(<$>) = fmap
```

這樣我們就可以寫

```haskell
liftA2 h fa fb = h <$> fa <*> fb
```

那 `liftA3` 呢

```haskell
liftA3 :: Applicative f => (a -> b -> c -> d) -> f a -> f b -> f c -> f d
liftA3 h fa fb fc = ((h <$> fa) <*> fb) <*> fc
```

（注意，`(<$>)` 和 `(<*>)` 的優先級和結合性實際上以這樣的方式，上述的所有的括號是不必要的定義）

好漂亮！與從 `fmap` 到 `liftA2` （需要從 `Functor` 到泛化 `Applicative`）的跳轉不同，從 `liftA2` 到 `liftA3` （然後從那裡到 `liftA4` …）跳轉不需要任何額外的功能 —— `Applicative` 就足夠了

實際上，當我們擁有所有這樣的參數時，我們通常不必費心調用 `liftA2`, `liftA3` 等，而是直接使用 `f <$> x <*> y <*> z <*> ...` 模式。（但是，`liftA2` 和朋友確實在部分應用會派上用場）

但是 `pure` 呢？`pure` 用於需要在某個函子 `f` 的上下文中將某些函數應用於自變量，但其中一個或多個自變量不在 `f` 中的情況 —— 可以說，這些自變量是“純”的。 在應用之前，我們可以先使用 `pure` 將它們提升到 `f`。 像這樣：

```haskell
liftX :: Applicative f => (a -> b -> c -> d) -> f a -> b -> f c -> f d
liftX h fa b fc = h <$> fa <*> pure b <*> fc
```

## ▌Applicative laws / Applicative 定律

對於 `Applicative`，只有一個真正“有趣”的定律：

```haskell
f `fmap` x === pure f <*> x
```

將函數 `f` 映射到容器 `x` 上應獲得與首先將函數注入容器中，然後使用 `(<*>)` 將其應用於 `x` 相同的結果

還有其他定律，但它們沒有啟發性。如果您確實需要，可以自己閱讀

## ▌Applicative examples / Applicative 例子

`Maybe`

讓我們嘗試從 `Maybe` 開始編寫一些 `Applicative` 實例。 通過將值注入到 `Just` 包裝器中來進行純工作； `(<*>)` 是可能存在故障的功能應用程序。 如果函數或其參數為空，則結果為 `Nothing`

```haskell
instance Applicative Maybe where
  pure              = Just
  Nothing <*> _     = Nothing
  _ <*> Nothing     = Nothing
  Just f <*> Just x = Just (f x)
```

讓我們來看一個例子：

```haskell
m_name1, m_name2 :: Maybe Name
m_name1 = Nothing
m_name2 = Just "Brent"

m_phone1, m_phone2 :: Maybe String
m_phone1 = Nothing
m_phone2 = Just "555-1234"

ex01 = Employee <$> m_name1 <*> m_phone1
ex02 = Employee <$> m_name1 <*> m_phone2
ex03 = Employee <$> m_name2 <*> m_phone1
ex04 = Employee <$> m_name2 <*> m_phone2
```
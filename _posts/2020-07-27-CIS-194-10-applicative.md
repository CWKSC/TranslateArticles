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

> Translation not finish

## ▌Applicative functors, Part I 適用函子，第一部分

CIS 194 第 10 週
2012 年 3 月 25 日

建議閱讀：

- [Applicative Functors](http://learnyouahaskell.com/functors-applicative-functors-and-monoids#applicative-functors) from Learn You a Haskell
- [The Typeclassopedia](http://www.haskell.org/haskellwiki/Typeclassopedia)

## ▌動機

考慮以下`Employee`類型：

```
type Name = String

data Employee = Employee { name    :: Name
                         , phone   :: String }
                deriving Show
```

當然，`Employee`構造函數具有類型

```
Employee :: Name -> String -> Employee
```

也就是說，如果我們具有`Name`和`String`，則可以應用`Employee`構造函數來構建`Employee`對象。

但是，假設我們沒有a `Name`和a `String`；我們實際擁有的是一個`Maybe Name`和一個`Maybe String`。也許它們來自解析充滿錯誤的某些文件，或者來自某些字段可能留為空白的形式，或者來自某種形式。我們不一定可以製作一個`Employee`。但是當然我們可以做一個`Maybe Employee`。也就是說，我們想使用我們的`(Name -> String -> Employee)`功能並將其轉換為`(Maybe Name -> Maybe String -> Maybe Employee)`功能。我們可以用這種類型寫東西嗎？

```
(Name -> String -> Employee) ->
(Maybe Name -> Maybe String -> Maybe Employee)
```

當然可以，我完全相信您現在就可以在睡眠中寫下它。我們可以想像它是如何工作的：如果名稱或字符串是`Nothing`，我們就走`Nothing`了；如果兩者都為`Just`，則`Employee`使用`Employee`構造函數（封裝在中`Just`）得出一個構建體。但是，讓我們繼續前進...

考慮一下：現在有a `Name`和a 而不是`String`a `[Name]`和a `[String]`。也許我們可以擺脫`[Employee]`困境？現在我們要

```
(Name -> String -> Employee) ->
([Name] -> [String] -> [Employee])
```

我們可以想像兩種不同的工作方式：我們可以將對應的`Name`s和`String`s 匹配成`Employee`s；或者我們可以通過所有可能的方式將`Name`s和`String`s 配對。

還是這樣：我們有一個`(e -> Name)`and `(e -> String)`用於某種類型`e`。例如，也許`e`是一些巨大的數據結構，並且我們有一些函數告訴我們如何從中提取a `Name`和a `String`。我們可以將其製成一個`(e -> Employee)`，即`Employee`從相同結構中提取的方法嗎？

```
(Name -> String -> Employee) ->
((e -> Name) -> (e -> String) -> (e -> Employee))
```

沒問題，這一次實際上只有一種編寫此函數的方法。

## ▌泛化

既然我們已經看到了這種模式的用處，讓我們概括一下。我們想要的函數類型實際上看起來像這樣：

```
(a -> b -> c) -> (f a -> f b -> f c)
```

嗯，這看起來很熟悉……與的類型非常相似`fmap`！

```
fmap :: (a -> b) -> (f a -> f b)
```

唯一的區別是額外的論點。我們可以調用所需的函數`fmap2`，因為它需要兩個參數的函數。也許我們可以`fmap2`根據來寫`fmap`，所以我們只需要`Functor`限制以下內容`f`：

```
fmap2 :: Functor f => (a -> b -> c) -> (f a -> f b -> f c)
fmap2 h fa fb = undefined
```

盡力而為，但是`Functor`並不能給我們足夠的執行力`fmap2`。怎麼了？我們有

```
h  :: a -> b -> c
fa :: f a
fb :: f b
```

請注意，我們也可以編寫`h`as 的類型`a -> (b -> c)`。因此，我們有一個採用的函數`a`，並且我們有一個類型的值`f a`…我們唯一可以做的就是使用`fmap`將該函數移到上`f`，從而得到一個類型的結果：

```
h         :: a -> (b -> c)
fmap h    :: f a -> f (b -> c)
fmap h fa :: f (b -> c)
```

好，現在我們有了某種類型的`f (b -> c)`東西和某種類型的東西`f b`……這就是我們所困的地方！`fmap`不再有幫助。它為我們提供了一種將函數應用於`Functor`上下文中的值的方法，但是我們現在需要的是將*自身在`Functor`上下文中*的函數應用於*上下文*中的值`Functor`。

## ▌適用性

可能進行這種“上下文應用”的函子稱為*applicative*，並且`Applicative`類（在中定義[`Control.Applicative`](http://haskell.org/ghc/docs/latest/html/libraries/base/Control-Applicative.html)）捕獲了這種模式。

```
class Functor f => Applicative f where
  pure  :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
```

該`(<*>)`運營商（通常發音為“AP”，簡稱“申請”）封裝“上下文應用程序”的正是這個道理。還要注意的是，`Applicative`類需要它的實例是實例`Functor`為好，這樣我們就可以隨時使用`fmap`與實例`Applicative`。最後，請注意，`Applicative`還有另一個方法，`pure`它允許我們將類型的值注入`a`到容器中。現在，有趣的是，這`fmap0`將是`pure`：

```
pure  :: a             -> f a
fmap  :: (a -> b)      -> f a -> f b
fmap2 :: (a -> b -> c) -> f a -> f b -> f c
```

現在`(<*>)`，我們可以實現了`fmap2`，它在標準庫中實際上稱為`liftA2`：

```
liftA2 :: Applicative f => (a -> b -> c) -> f a -> f b -> f c
liftA2 h fa fb = (h `fmap` fa) <*> fb
```

實際上，這種模式非常普遍，因此`Control.Applicative`定義`(<$>)`為的同義詞`fmap`，

```
(<$>) :: Functor f => (a -> b) -> f a -> f b
(<$>) = fmap
```

這樣我們就可以寫

```
liftA2 h fa fb = h <$> fa <*> fb
```

那`liftA3`呢

```
liftA3 :: Applicative f => (a -> b -> c -> d) -> f a -> f b -> f c -> f d
liftA3 h fa fb fc = ((h <$> fa) <*> fb) <*> fc
```

（注意，優先級和的結合性`(<$>)`和`(<*>)`實際上以這樣的方式，上述的所有的括號是不必要的定義）。

好漂亮！與從`fmap`到`liftA2`（需要從`Functor`到泛化`Applicative`）的跳轉不同，從`liftA2`到`liftA3`（從那裡到`liftA4`…）跳轉不需要任何額外的功能— `Applicative`足夠。

其實，當我們擁有所有的參數，像這樣我們平時也懶得打電話`liftA2`，`liftA3`等，但只使用`f <$> x <*> y <*> z <*> ...`直接模式。（不過`liftA2`，朋友確實會派上用場進行部分應用。）

但是呢`pure`？`pure`是我們想要的一些功能應用到論據一些仿函數的背景情況`f`，但是一個或多個自變量是*不是*在`f`-those參數是“純粹的”，可以這麼說。在申請之前，我們可以先將`pure`它們提升到`f`第一位。像這樣：

```
liftX :: Applicative f => (a -> b -> c -> d) -> f a -> b -> f c -> f d
liftX h fa b fc = h <$> fa <*> pure b <*> fc
```

## ▌適用法律

只有以下一項真正“有趣”的法則`Applicative`：

```
f `fmap` x === pure f <*> x
```

將功能映射到`f`容器上`x`應該產生與首先將功能注入到容器中，然後將其應用於`x`with 相同的結果`(<*>)`。

還有其他法律，但它們沒有啟發性。如果您確實需要，可以自己閱讀。

## ▌適用實例

**也許**

讓我們嘗試以`Applicative`開頭的一些實例`Maybe`。`pure`通過將值注入`Just`包裝器來工作；`(<*>)`是可能失敗的功能應用程序。結果是`Nothing`函數或其參數是否為。

```
instance Applicative Maybe where
  pure              = Just
  Nothing <*> _     = Nothing
  _ <*> Nothing     = Nothing
  Just f <*> Just x = Just (f x)
```

讓我們來看一個例子：

```
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
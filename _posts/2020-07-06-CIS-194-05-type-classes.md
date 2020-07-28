---
date: 2020-07-06 19:00:00
layout: post
title: "CIS 194 05 More polymorphism and type classes 更多多態和類型類"
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

Source: [05-type-classes](https://www.seas.upenn.edu/~cis194/spring13/lectures/05-type-classes.html)

## ▌More polymorphism and type classes

## 更多多態和類型類

CIS 194 第 5 週
2013 年 2 月 11 日

Haskell 特殊的多態被稱為 *參數* 多態。從本質上講，這意味著多態函數必須對任何輸入類型 *均一地* 工作。事實證明，多態函數對於程序員和用戶都具有一些有趣的含義

## ▌Parametricity 參數化

考慮類型

```haskell
a -> a -> a
```

要記得 `a` 是一個 *類型變量* ，可以代表任何類型。這種類型的函數有哪些？

這個如何：

```haskell
f :: a -> a -> a
f x y = x && y
```

事實證明這是行不通。語法至少有效，但沒有類型檢查。我們收到以下錯誤消息：

```haskell
2012-02-09.lhs:37:16:
    Couldn't match type `a' with `Bool'
      `a' is a rigid type variable bound by
          the type signature for f :: a -> a -> a at 2012-02-09.lhs:37:3
    In the second argument of `(&&)', namely `y'
    In the expression: x && y
    In an equation for `f': f x y = x && y
```

這是因爲多態函數的 *調用者 (caller)* 可以選擇類型。在這裡，我們（*實現者 (implementors)*）試圖選擇一種特定的類型（即 `Bool`），但我們有機會給 `String` 或 `Int`，或者甚至是某人使用定義的某種類型 `f`，而我們可能無法事先知道。換句話說，您可以閱讀類型

```haskell
a -> a -> a
```

*承諾* 無論調用者選擇哪種類型，此類型的函數都將起作用

我們可以 *想像* 的另一個實現是

```haskell
f a1 a2 = case (typeOf a1) of
            Int  -> a1 + a2
            Bool -> a1 && a2
            _    -> a1
```

其中 `f` 某些類型的行為以某些特定方式表現。畢竟，我們當然可以在 Java 中實現它：

```java
class AdHoc {

    public static Object f(Object a1, Object a2) {
        if (a1 instanceof Integer && a2 instanceof Integer) {
            return (Integer)a1 + (Integer)a2;
        } else if (a1 instanceof Boolean && a2 instanceof Boolean) {
            return (Boolean)a1 && (Boolean)a2;
        } else {
            return a1;
        }
    }

    public static void main (String[] args) {
        System.out.println(f(1,3));
        System.out.println(f(true, false));
        System.out.println(f("hello", "there"));
    }

}

[byorgey@LVN513-9:~/tmp]$ javac Adhoc.java && java AdHoc
4
false
hello
```

但事實證明，無法用 Haskell 編寫此代碼。Haskell 沒有 Java `instanceof` 運算符之類的東西：不可能問什麼是什麼類型，並不能根據答案決定要做什麼。原因之一是檢查後，編譯器將*擦除* Haskell 類型：在運行時，沒有類型信息可供查詢！但是，正如我們將看到的，還有其他充分的理由

這種類型的多態被稱為 *參數多態* 。我們說像這樣的函數在類型中 `f :: a -> a -> a` 是 *參數化* 的 `a`。在這裡，“參數”只是“對於呼叫者選擇的任何類型都可以統一工作”的幻想術語。在 Java 中，這種類型的多態性是由 *泛型* 提供的（您猜對了，它受 Haskell 的啟發：Haskell 的原始設計師之一 [Philip Wadler](http://homepages.inf.ed.ac.uk/wadler/) 後來成為 Java 泛型開發的主要參與者之一）

那麼，實際上什麼函數 *可以* 具有這種類型？其實只有兩個！

```haskell
f1 :: a -> a -> a
f1 x y = x

f2 :: a -> a -> a
f2 x y = y
```

因此，事實證明類型 `a -> a -> a` 確實告訴了我們很多東西。

讓我們玩參數化遊戲！考慮以下每種多態類型。對於每種類型，確定該類型的功能可能具有的行為。

```haskell
a -> a
a -> b
a -> b -> a
[a] -> [a]
(b -> c) -> (a -> b) -> (a -> c)
(a -> a) -> a -> a
```

## ▌Two views on parametricity 關於參數化的兩種觀點

作為多態函數的 *實現* 者，尤其是如果您習慣於使用 Java 之類的結構的語言時 `instanceof`，您可能會發現這些限制很煩人。“你是什麼意思，我不允許做 X？”

但是，存在雙重觀點。作為多態函數的 *用戶* ，參數性不對應於 *限制，* 而是對應於 *保證* 。通常，當這些工具為您提供有關其行為方式的有力保證時，它們的使用和推理就容易得多。參數化是僅查看 Haskell 函數類型可以告訴您有關函數的太多原因的一部分。

好的，很好，但是有時候根據類型決定要做什麼真的很有用！例如，加法呢？我們已經看到加法是多態的（例如，可在 `Int`，`Integer` 和 `Double` 上使用），但顯然必須知道要添加的數字類型才能決定要做什麼：加兩個 `Integer`s 的方式與加法兩個 `Double`s 完全不同。那麼它實際上是如何工作的呢？只是魔術嗎？

其實不是！實際上，我們 *可以* 使用 Haskell 根據類型決定要做什麼，只是不像我們以前想像的那樣。讓我們先來看一下 `(+)` 的類型：

```haskell
Prelude> :t (+)
(+) :: Num a => a -> a -> a
```

嗯，`Num a =>` 那邊在做什麼？實際上，`(+)` 它並不是唯一一個帶有有趣的雙箭頭類型的標準函數。這裡還有其他一些：

```haskell
(==) :: Eq a   => a -> a -> Bool
(<)  :: Ord a  => a -> a -> Bool
show :: Show a => a -> String
```

這是怎麼回事？

## ▌Type classes 類型類別

`Num`，`Eq`，`Ord`, 和 `Show` 是*類型類 (Type classes)*，我們說 `(==)`，`(<)` 和 `(+)` 有 “類型類多態(type-class polymorphic)”。直觀地講，類型類對應於為其定義了某些操作 *的類型集*，並且類型類多態函數僅適用於所討論的類型類實例。作為示例，讓我們詳細研究 `Eq` 類型類

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
```

我們可以這樣閱讀：`Eq` 聲明為帶有單個參數 `a` 的類型類。任何類型的 `a` 要成為 `Eq` 的 *實例* 的都必須定義兩個函數，`(==)` 和 `(/=)` ，並使用指定的類型簽名。例如，要創建 `Int` 一個實例，`Eq` 我們必須定義 `(==) :: Int -> Int -> Bool` 和 `(/=) :: Int -> Int -> Bool`。（當然，沒有必要，因為標準的 Prelude 已經為我們定義了一個 `Int` 實例 `Eq`）

讓我們再次看看 `(==)` 的類型：

```haskell
(==) :: Eq a => a -> a -> Bool
```

`=>` 之前的 `Eq a` 是 *類型類約束* 。我們可以這樣理解：對於任何類型的 `a`，只要 `a` 是 `Eq` 的實例，`(==)` 都可以採用兩個類型 `a` 的值並返回 `Bool`。在不是 Eq 實例的某種類型上調用函數 `(==)` 是類型錯誤。如果一個普通多態類型是承諾了該函數將適用於調用者選擇的任何類型，則類型類多態函數是一個受限的承諾，該函數將適用於調用者選擇的任何類型 —— 只要選擇的類型是所需類型類的實例即可

需要注意的重要一點是，當使用 `(==)`（或任何類型類方法）時，編譯器 *會* 根據其參數的推斷類型使用類型推斷來確定 *應選擇哪種實現 `(==)`*。換句話說，這類似於在 Java 之類的語言中使用重載方法

為了更好地了解它在實際中的工作方式，讓我們創建自己的類型並為其聲明一個 `Eq` 實例。

```haskell
data Foo = F Int | G Char

instance Eq Foo where
  (F i1) == (F i2) = i1 == i2
  (G c1) == (G c2) = c1 == c2
  _ == _ = False

  foo1 /= foo2 = not (foo1 == foo2)
```

我們必須同時定義 `(==)` 和 `(/=)`，這有點令人討厭 。實際上，類型類可以根據其他方法提供方法的 *默認實現* ，只要實例不使用其自身覆蓋默認定義，就應使用該方法。因此我們可以想像這樣聲明 `Eq`：

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  x /= y = not (x == y)
```

現在，任何聲明 `Eq` 實例的人都只需實現 `(==)`，他們將免費獲得 `(/=)` 。但是，如果由於某些原因他們想 `(/=)` 用自己的方法覆蓋默認實現，則也可以這樣做。

實際上，`Eq` 該類是這樣聲明的：

```haskell
class Eq a where
  (==), (/=) :: a -> a -> Bool
  x == y = not (x /= y)
  x /= y = not (x == y)
```

這意味著，當我們做的一個實例 `Eq`，我們可以定義 *兩種*  `(==)` 或 `(/=)`，取其更方便; 另一項將根據我們指定的一項自動定義。（但是，我們必須要小心：如果不指定任何一個，我們將得到無限遞歸！）

事實證明，`Eq` （以及其他一些標準類型類）很特殊：GHC 能夠自動 `Eq` 為我們生成實例。像這樣：

```haskell
data Foo' = F' Int | G' Char
  deriving (Eq, Ord, Show)
```

這告訴 GHC 的自動派生實例 `Eq`，`Ord` 以及 `Show` 類型類為我們的數據類型 `Foo`。

### ▌Type classes and Java interfaces 類型類和 Java 接口

類型類與 Java 接口非常相似。兩者都定義了一組實現特定操作列表的類型 / 類。但是，有幾種重要的方法可以使類型類比 Java 接口更通用：

1. 定義 Java 類時，必須聲明其實現的任何接口。另一方面，類型類實例是與相應類型的聲明分開聲明的，甚至可以放在單獨的模塊中

2. 可以為類型類方法指定的類型比為 Java 接口方法可以指定的簽名更通用、更靈活，尤其是當 *多參數類型類*  輸入圖片（？）時。例如，考慮一個假設的類型類

   > 不知道 enter the picture 如何翻譯，原文在此：
>
   > The types that can be specified for type class methods are more general and flexible than the signatures that can be given for Java interface methods, especially when *multi-parameter type classes* enter the picture. For example, consider a hypothetical type class

   ```haskell
   class Blerg a b where
  blerg :: a -> b -> Bool
   ```

   使用 `blerg` 執行多次調度：編譯器取決於類型 `a` 和 `b` 選擇哪種 `blerg` 實現。 Java 沒有簡單的方法可以做到這一點

   Haskell 類型類也可以輕鬆地處理二進制（或三進制或 …）方法，如

   ```haskell
   class Num a where
     (+) :: a -> a -> a
  ...
   ```

   在 Java 中，沒有很好的方法來執行此操作：一方面，兩個參數之一必須是 “特權(privileged)” 參數，實際上是要 `(+)` 在其上調用方法，並且這種不對稱性很尷尬。此外，由於 Java 的子類型化，獲取某個接口類型的兩個參數並 *不能* 保證它們實際上是同一類型，這使得實現二進制運算符（如 `(+)`）很尷尬（通常需要進行一些運行時類型檢查）

   > There is no nice way to do this in Java: for one thing, one of the two arguments would have to be the “privileged” one which is actually getting the `(+)` method invoked on it, and this asymmetry is awkward. Furthermore, because of Java’s subtyping, getting two arguments of a certain interface type does *not* guarantee that they are actually the same type, which makes implementing binary operators such as `(+)` awkward (usually requiring some runtime type checks).

### ▌Standard type classes 標準類型類別

這是您應該了解的一些其他標準類型類：

- [Ord](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AOrd) 用於其元素可以 *完全排序的類型* ，即可以比較任何兩個元素以查看哪個元素小於另一個元素。它提供了類似的比較操作 `(<)` 和 `(<=)`，也是 `compare` 函數

- [Num](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3ANum) 用於“數字”類型，它支持加法，減法和乘法運算。需要注意的一件事是整數實際上是類型類多態：

  ```haskell
  Prelude> :t 5
  5 :: Num a => a
  ```

  這意味著像 `5` 這樣的文字可以用作 `Int`s，`Integer`s，`Double`s 或作為 `Num` 實例的任何其他類型（`Rational`，`Complex Double` 甚至是您定義的類型 ...）

- [Show](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AShow) 定義了方法 `show`，該方法用於將值轉換為 `String`s

- [Read](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#v:Eq/Read) 是的雙重性(dual of) `Show`     PS:（ dual of 或者是重複的意思？）

- [Integral](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AIntegral) 表示整數類型，例如 `Int` 和 `Integer`

### ▌A type class example 類型類示例

作為製作自己的類型類的示例，請考慮以下內容：

```haskell
class Listable a where
  toList :: a -> [Int]
```

我們可以將其 `Listable` 視為可以轉換為 `Int`s 列表的事物類。看一下類型 `toList`：

```haskell
toList :: Listable a => a -> [Int]
```

讓我們為做一些實例 `Listable`。首先，僅通過創建一個單列表就可以將一個 `Int` 可以轉換為 `[Int]` ，並且 `Bool` 同樣可以轉換，也就是說，通過翻譯 `True` 來 `1` 和 `False` 到 `0`：

```haskell
instance Listable Int where
  -- toList :: Int -> [Int]
  toList x = [x]

instance Listable Bool where
  toList True  = [1]
  toList False = [0]
```

我們無需做任何工作即可將 `Int` 列表轉換為 `Int` 列表：

```haskell
instance Listable [Int] where
  toList = id
```

最後，這是一個二叉樹類型，我們可以通過展平將其轉換為列表：

```haskell
data Tree a = Empty | Node a (Tree a) (Tree a)

instance Listable (Tree Int) where
  toList Empty        = []
  toList (Node x l r) = toList l ++ [x] ++ toList r
```

如果我們根據 `toList` 實現其他函數，則它們也會得到 `Listable` 約束。例如：

```haskell
-- to compute sumL, first convert to a list of Ints, then sum
sumL x = sum (toList x)
```

`ghci` 告知我們類型 `sumL` 為

```haskell
sumL :: Listable a => a -> Int
```

這很有意義：`sumL` 僅適用於 `Listable` 實例的類型，因為它使用 `toList`。 這個如何？

```haskell
foo x y = sum (toList x) == sum (toList y) || x < y
```

`ghci` 告知我們的類型 `foo` 是

```haskell
foo :: (Listable a, Ord a) => a -> a -> Bool
```

也就是說，`foo` 可以處理同時是 `toList` 和 `Ord` 實例的類型，因為它同時使用了 `Listable` 和 `Ord` 

作為最後一個更複雜的示例，請考慮以下實例：

```haskell
instance (Listable a, Listable b) => Listable (a,b) where
  toList (x,y) = toList x ++ toList y
```

注意，我們如何將類型類約束放在實例以及函數類型上。 這就是說，只要 `a` 和 `b` 都存在，對類型 `(a，b)` 就是 `Listable` 的實例。 然後，在對 `toList` 的定義中，對成對的 `a` 和 `b` 類型的值使用 `toList`。 請注意，此定義不是遞歸的！ 我們正在定義的 `toList` 版本正在調用 `toList` 的其他版本，而不是本身
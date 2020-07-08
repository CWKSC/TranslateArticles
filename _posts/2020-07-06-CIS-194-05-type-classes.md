---
date: 2020-07-04 19:00:00
layout: post
title: "CIS 194 05 More polymorphism and type classes"
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

## ▌More polymorphism and type classes 更多多態和類型類

CIS 194 第 5 週
2013 年 2 月 11 日

Haskell 特殊的多態品牌被稱為*參數*多態。從本質上講，這意味著多態函數必須對任何輸入類型*均一地*工作。事實證明，這對於程序員和多態函數用戶都具有一些有趣的含義。

## 參數化

考慮類型

```
a -> a -> a
```

請記住，這`a`是一個*類型變量*，可以代表任何類型。這種類型的功能有哪些？

那這個呢：

```
f :: a -> a -> a
f x y = x && y
```

事實證明這是行不通的。該語法至少有效，但不鍵入check。特別是，我們收到以下錯誤消息：

```
2012-02-09.lhs:37:16:
    Couldn't match type `a' with `Bool'
      `a' is a rigid type variable bound by
          the type signature for f :: a -> a -> a at 2012-02-09.lhs:37:3
    In the second argument of `(&&)', namely `y'
    In the expression: x && y
    In an equation for `f': f x y = x && y
```

這不起作用的原因是多態函數的*調用者*可以選擇類型。在這裡，我們（*實現*者）試圖選擇一種特定的類型（即`Bool`），但可能會給我們`String`，或`Int`，或者甚至是某人使用定義的某種類型`f`，而我們可能無法事先知道。換句話說，您可以閱讀類型

```
a -> a -> a
```

作為*承諾*，與這種類型的功能將工作不管是什麼類型的調用者進行選擇。

我們可以想像的另一個實現是

```
f a1 a2 = case (typeOf a1) of
            Int  -> a1 + a2
            Bool -> a1 && a2
            _    -> a1
```

其中`f`某些類型的行為以某些特定方式表現。畢竟，我們當然可以在Java中實現它：

```
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

但事實證明，無法用Haskell編寫此代碼。Haskell沒有Java `instanceof`運算符之類的東西：不可能問什麼是什麼類型，並不能根據答案決定要做什麼。原因之一是檢查後，編譯器將*擦除* Haskell類型：在運行時，沒有類型信息可供查詢！但是，正如我們將看到的，還有其他充分的理由。

這種類型的多態被稱為*參數多態*。我們說像這樣的函數在類型中`f :: a -> a -> a`是*參數化*的`a`。在這裡，“參數”只是“對於呼叫者選擇的任何類型都可以統一工作”的幻想術語。在Java中，這種類型的多態性是由*泛型*提供的（您猜對了，它受Haskell的啟發：Haskell的原始設計師之一[Philip Wadler](http://homepages.inf.ed.ac.uk/wadler/)後來成為Java泛型開發的主要參與者之一）。

那麼，實際上什麼功能*可以*具有這種類型？其實只有兩個！

```
f1 :: a -> a -> a
f1 x y = x

f2 :: a -> a -> a
f2 x y = y
```

因此，事實證明類型`a -> a -> a`確實告訴了我們很多東西。

讓我們玩參數化遊戲！考慮以下每種多態類型。對於每種類型，確定該類型的功能可能具有的行為。

- `a -> a`
- `a -> b`
- `a -> b -> a`
- `[a] -> [a]`
- `(b -> c) -> (a -> b) -> (a -> c)`
- `(a -> a) -> a -> a`

## 關於參數化的兩種觀點

作為多態函數的*實現*者，尤其是如果您習慣於使用Java之類的結構的語言時`instanceof`，您可能會發現這些限制很煩人。“你是什麼意思，我不允許做X？”

但是，存在雙重觀點。作為多態函數的*用戶*，參數性不對應於*限制，*而是對應於*保證*。通常，當這些工具為您提供有關其行為方式的有力保證時，它們的使用和推理就容易得多。參數化是僅查看Haskell函數類型可以告訴您有關函數的太多原因的一部分。

好的，很好，但是有時候根據類型決定要做什麼真的很有用！例如，加法呢？我們已經看到加法是多態的（例如，可在`Int`，`Integer`和上`Double`使用），但顯然必須知道要添加的數字類型才能決定要做什麼：加兩個`Integer`s的方式與加法完全不同2 `Double`秒。那麼它實際上是如何工作的呢？只是魔術嗎？

其實不是！實際上，我們*可以*使用Haskell根據類型決定要做什麼，只是不像我們以前想像的那樣。讓我們先來看一下的類型`(+)`：

```
Prelude> :t (+)
(+) :: Num a => a -> a -> a
```

嗯，`Num a =>`那邊在做什麼？實際上，`(+)`它並不是唯一一個帶有有趣的雙箭頭類型的標準函數。這裡還有其他一些：

```
(==) :: Eq a   => a -> a -> Bool
(<)  :: Ord a  => a -> a -> Bool
show :: Show a => a -> String
```

那麼這是怎麼回事？

## 類型類別

`Num`，`Eq`，`Ord`，和`Show`是*類型類*，和我們說`(==)`，`(<)`和`(+)`有“型級多態”。直觀地講，類型類對應於為其定義了某些操作*的類型集*，並且類型類多態函數僅適用於作為所討論類型類實例的類型。作為示例，讓我們詳細研究`Eq`類型類。

```
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
```

我們可以這樣閱讀：`Eq`聲明為帶有單個參數的類型類`a`。任何`a`要成為的*實例*的類型都`Eq`必須定義兩個函數，`(==)`並`(/=)`使用指定的類型簽名。例如，要創建`Int`一個實例，`Eq`我們必須定義`(==) :: Int -> Int -> Bool`和`(/=) :: Int -> Int -> Bool`。（當然，沒有必要，因為標準的Prelude已經為我們定義了一個`Int`實例`Eq`。）

讓我們`(==)`再次看看類型：

```
(==) :: Eq a => a -> a -> Bool
```

在`Eq a`這到來之前的`=>`是一個*類型類的約束*。我們可以這樣理解：對於任何類型`a`，*只要`a`是`Eq`* type *的實例*，`(==)`都可以採用type的兩個值`a`並返回a `Bool`。`(==)`在不是的實例的某種類型上調用函數是類型錯誤`Eq`。如果一個普通的多態類型是一個保證該函數將適用於調用者選擇的任何類型的承諾，則類型類多態函數是一個*受限的*保證，即該函數將適用於調用者選擇的任何類型，*只要*所選的類型是一個實例即可。所需類型的類別。

需要注意的重要一點是，當使用`(==)`（或任何類型類方法）時，編譯器*會*根據其參數的推斷類型使用類型推斷來確定*應選擇**哪種實現`(==)`*。換句話說，這類似於在Java之類的語言中使用重載方法。

為了更好地了解它在實際中的工作方式，讓我們創建自己的類型並`Eq`為其聲明一個實例。

```
data Foo = F Int | G Char

instance Eq Foo where
  (F i1) == (F i2) = i1 == i2
  (G c1) == (G c2) = c1 == c2
  _ == _ = False

  foo1 /= foo2 = not (foo1 == foo2)
```

我們必須同時定義`(==)`和，這有點令人討厭`(/=)`。實際上，類型類可以根據其他方法提供方法的*默認實現*，只要實例不使用其自身覆蓋默認定義，就應使用該方法。因此我們可以想像這樣聲明`Eq`：

```
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  x /= y = not (x == y)
```

現在，任何聲明`Eq`only 實例的人都必須指定的實現`(==)`，他們將`(/=)`免費獲得。但是，如果由於某些原因他們想`(/=)`用自己的方法覆蓋默認實現，則也可以這樣做。

實際上，`Eq`該類實際上是這樣聲明的：

```
class Eq a where
  (==), (/=) :: a -> a -> Bool
  x == y = not (x /= y)
  x /= y = not (x == y)
```

這意味著，當我們做的一個實例`Eq`，我們可以定義*兩種* `(==)`或`(/=)`，取其更方便; 另一項將根據我們指定的一項自動定義。（但是，我們必須要小心：如果不指定任何一個，我們將得到無限遞歸！）

事實證明，`Eq`（以及其他一些標準類型類）很特殊：GHC能夠自動`Eq`為我們生成實例。像這樣：

```
data Foo' = F' Int | G' Char
  deriving (Eq, Ord, Show)
```

這告訴GHC的自動派生實例`Eq`，`Ord`以及`Show`類型類為我們的數據類型`Foo`。

**類型類和Java接口**

類型類與Java接口非常相似。兩者都定義了一組實現特定操作列表的類型/類。但是，有幾種重要的方法可以使類型類比Java接口更通用：

1. 定義Java類時，必須聲明其實現的任何接口。另一方面，類型類實例是與相應類型的聲明分開聲明的，甚至可以放在單獨的模塊中。

2. 可以為類型類方法指定的類型比為Java接口方法可以指定的簽名更通用，更靈活，尤其是當*多參數類型類*輸入圖片時。例如，假設一個假設類型類

   ```
   class Blerg a b where
     blerg :: a -> b -> Bool
   ```

   使用`blerg`達做*多分派*：其中實現`blerg`編譯器應該選擇取決於*兩種*類型`a`和`b`。用Java沒有簡單的方法可以做到這一點。

   Haskell類型類也可以輕鬆地處理二進制（或三進製或…）方法，如

   ```
   class Num a where
     (+) :: a -> a -> a
     ...
   ```

   在Java中，沒有很好的方法來執行此操作：一方面，兩個參數之一必須是“特權”參數，實際上是要`(+)`在其上調用方法，並且這種不對稱性很尷尬。此外，由於Java的子類型化，獲取某個接口類型的兩個參數並*不能*保證它們實際上是同一類型，這使得實現二進制運算符（例如`(+)`笨拙）（通常需要進行一些運行時類型檢查）成為可能。

**標準類型類別**

這是您應該了解的其他一些標準類型類：

- [Ord](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AOrd)用於其元素可以*完全排序的類型*，即可以比較任何兩個元素以查看哪個元素小於另一個元素。它提供了類似的比較操作`(<)`和`(<=)`，也是`compare`功能。

- [Num](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3ANum)用於“數字”類型，它支持加法，減法和乘法運算。需要注意的一件事是整數實際上是類型多態的類型：

  ```
  Prelude> :t 5
  5 :: Num a => a
  ```

  這意味著像這樣的文字`5`可以用作`Int`s，`Integer`s，`Double`s或作為`Num`（`Rational`，`Complex Double`甚至是您定義的類型...）實例的任何其他類型。

- [Show](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AShow)定義了方法`show`，該方法用於將值轉換為`String`s。

- [閱讀](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#v:Eq/Read)是的雙重性`Show`。

- [整數](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#t%3AIntegral)表示整數類型，例如`Int`和`Integer`。

**類型類示例**

作為製作自己的類型類的示例，請考慮以下內容：

```
class Listable a where
  toList :: a -> [Int]
```

我們可以將其`Listable`視為可以轉換為`Int`s 列表的事物類。看一下類型`toList`：

```
toList :: Listable a => a -> [Int]
```

讓我們為做一些實例`Listable`。首先，`Int`可以轉換為`[Int]`只通過創建一個單列表，並且`Bool`同樣可以轉換，也就是說，通過翻譯`True`來`1`和`False`到`0`：

```
instance Listable Int where
  -- toList :: Int -> [Int]
  toList x = [x]

instance Listable Bool where
  toList True  = [1]
  toList False = [0]
```

我們無需做任何工作即可將的列表轉換`Int`為的列表`Int`：

```
instance Listable [Int] where
  toList = id
```

最後，這是一個二叉樹類型，我們可以通過展平將其轉換為列表：

```
data Tree a = Empty | Node a (Tree a) (Tree a)

instance Listable (Tree Int) where
  toList Empty        = []
  toList (Node x l r) = toList l ++ [x] ++ toList r
```

如果我們根據實現其他功能`toList`，它們也會受到`Listable`約束。例如：

```
-- to compute sumL, first convert to a list of Ints, then sum
sumL x = sum (toList x)
```

`ghci`告知我們類型`sumL`為

```
sumL :: Listable a => a -> Int
```

這很有意義：由於使用，因此`sumL`僅適用於作為實例的類型。這個如何？`Listable``toList`

```
foo x y = sum (toList x) == sum (toList y) || x < y
```

`ghci`告知我們的類型`foo`是

```
foo :: (Listable a, Ord a) => a -> a -> Bool
```

也就是說，`foo`工作在它們的實例類型*都* `Listable`和`Ord`，因為它同時使用`toList`和比較的參數。

作為最後一個更複雜的示例，請考慮以下實例：

```
instance (Listable a, Listable b) => Listable (a,b) where
  toList (x,y) = toList x ++ toList y
```

注意，我們如何將類型類約束放在實例以及函數類型上。這表示對類型`(a,b)`是和都`Listable`一樣長的實例。然後，我們就可以在類型的值上以及在對的定義中使用。請注意，此定義*不是*遞歸的！的版本，我們正在定義是調用*其他*的版本，而不是自己。`a``b``toList``a``b``toList``toList``toList`
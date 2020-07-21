---
date: 2020-07-21 23:00:00
layout: post
title: "CIS 194 07 Folds and monoids 折疊和么半群"
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

Source: [07-folds-monoids](https://www.seas.upenn.edu/~cis194/spring13/lectures/07-folds-monoids.html)

## ▌Folds and monoids 折疊和么半群

CIS 194 第 7 週
2013 年 2 月 25 日

建議閱讀：

- Learn You a Haskell, [Only folds and horses](http://learnyouahaskell.com/higher-order-functions#folds)
- Learn You a Haskell, [Monoids](http://learnyouahaskell.com/functors-applicative-functors-and-monoids#monoids)
- [Fold](http://haskell.org/haskellwiki/Fold) from the Haskell wiki
- Heinrich Apfelmus, [Monoids and Finger Trees](http://apfelmus.nfshost.com/articles/monoid-fingertree.html)
- Dan Piponi, [Haskell Monoids and their Uses](http://blog.sigfpe.com/2009/01/haskell-monoids-and-their-uses.html)
- [Data.Monoid documentation](http://haskell.org/ghc/docs/latest/html/libraries/base/Data-Monoid.html)
- [Data.Foldable documentation](http://haskell.org/ghc/docs/latest/html/libraries/base/Data-Foldable.html)

## ▌Folds, again 再折

我們已經了解如何為列表定義折疊函數 …… 但是我們可以把這種想法推廣到其他數據類型！

考慮以下二進制樹的數據類型，數據存儲在內部節點中：

```haskell
data Tree a = Empty
            | Node (Tree a) a (Tree a)
  deriving (Show, Eq)

leaf :: a -> Tree a
leaf x = Node Empty x Empty
```

讓我們編寫一個函數來計算樹的大小（*即*  `Node`s 的數量）：

```haskell
treeSize :: Tree a -> Integer
treeSize Empty        = 0
treeSize (Node l _ r) = 1 + treeSize l + treeSize r
```

`Integer`s 樹中的數據總和？

```haskell
treeSum :: Tree Integer -> Integer
treeSum Empty     = 0
treeSum (Node l x r)  = x + treeSum l + treeSum r
```

還是一棵樹的深度？

```haskell
treeDepth :: Tree a -> Integer
treeDepth Empty        = 0
treeDepth (Node l _ r) = 1 + max (treeDepth l) (treeDepth r)
```

還是將樹的元素展平為列表？

```haskell
flatten :: Tree a -> [a]
flatten Empty        = []
flatten (Node l x r) = flatten l ++ [x] ++ flatten r
```

您開始看到任何模式了嗎？以上每個函數：

1. 需要 `Tree` 作為輸入

2. 模式匹配在輸入的 `Tree` 上

3. 在 `Empty` 情況下，給出一個簡單的答案

4. 在 `Node` 情況下：

   1. 在兩個子樹上遞歸調用自身
   2. 以某種方式將遞歸調用的結果與數據結合起來 `x` 以產生最終結果

作為優秀的程序員，我們始終努力抽像出重複的模式，對吧？因此，讓我們概括一下。我們需要將上述示例的各個部分作為參數傳遞，這些示例因示例而異：

1. 返回類型
2. `Empty` 情況的答案
3. 如何合併遞歸調用

我們將樹中包含的數據類型稱為 `a`，結果類型稱為 `b`

```haskell
treeFold :: b -> (b -> a -> b -> b) -> Tree a -> b
treeFold e _ Empty        = e
treeFold e f (Node l x r) = f (treeFold e f l) x (treeFold e f r)
```

現在，我們應該能夠更簡單地定義 `treeSize`，`treeSum` 和其他示例。 我們試試吧：

```haskell
treeSize' :: Tree a -> Integer
treeSize' = treeFold 0 (\l _ r -> 1 + l + r)

treeSum' :: Tree Integer -> Integer
treeSum' = treeFold 0 (\l x r -> l + x + r)

treeDepth' :: Tree a -> Integer
treeDepth' = treeFold 0 (\l _ r -> 1 + max l r)

flatten' :: Tree a -> [a]
flatten' = treeFold [] (\l x r -> l ++ [x] ++ r)
```

我們還可以輕鬆地編寫新的樹折疊函數：

```haskell
treeMax :: (Ord a, Bounded a) => Tree a -> a
treeMax = treeFold minBound (\l x r -> l `max` x `max` r)
```

好多了！

### ▌Folding expressions 折疊式

我們還能在哪裡看到折疊？

回顧作業 5 的 `ExprT` 類型和相應的 `eval` 功能：

```haskell
data ExprT = Lit Integer
           | Add ExprT ExprT
           | Mul ExprT ExprT

eval :: ExprT -> Integer
eval (Lit i)     = i
eval (Add e1 e2) = eval e1 + eval e2
eval (Mul e1 e2) = eval e1 * eval e2
```

嗯 ... 這看起來很熟悉！折疊 `ExprT` 起來會是什麼樣子？

```haskell
exprTFold :: (Integer -> b) -> (b -> b -> b) -> (b -> b -> b) -> ExprT -> b
exprTFold f _ _ (Lit i)     = f i
exprTFold f g h (Add e1 e2) = g (exprTFold f g h e1) (exprTFold f g h e2)
exprTFold f g h (Mul e1 e2) = h (exprTFold f g h e1) (exprTFold f g h e2)

eval2 :: ExprT -> Integer
eval2 = exprTFold id (+) (*)
```

現在，我們可以輕鬆地執行其他操作，例如計算表達式中 文字的數量：

```haskell
numLiterals :: ExprT -> Int
numLiterals = exprTFold (const 1) (+) (+)
```

### ▌Folds in general 一般的折疊

要點是，我們可以為許多（儘管不是全部）數據類型實現折疊。`T` 的折疊將為每個 `T` 的構造函數使用一個（高階）參數，編碼如何將由該構造函數存儲的值轉換為結果類型的值 — 假定 `T` 的任何遞歸出現都已被折疊為一個 結果。 我們可能想在 `T` 上編寫的許多函數最終都可以表示為簡單的折疊

> The take-away message is that we can implement a fold for many (though not all) data types. The fold for `T` will take one (higher-order) argument for each of `T`’s constructors, encoding how to turn the values stored by that constructor into a value of the result type—assuming that any recursive occurrences of `T` have already been folded into a result. Many functions we might want to write on `T` will end up being expressible as simple folds.

## ▌Monoids 么半群

這是您應該了解的另一個標準類型類，可在 [`Data.Monoid`](http://haskell.org/ghc/docs/latest/html/libraries/base/Data-Monoid.html) 模塊中找到：

```haskell
class Monoid m where
    mempty  :: m
    mappend :: m -> m -> m

    mconcat :: [m] -> m
    mconcat = foldr mappend mempty

(<>) :: Monoid m => m -> m -> m
(<>) = mappend
```

`(<>) `被定義為 `mappend` 的同義詞（自GHC 7.4.1起），僅僅是因為編寫 `mappend` 很繁瑣

作為 `Monoid` 實例的類型具有一個稱為 `mempty` 的特殊元素，以及一個二進制操作 `mappend`（縮寫為 `(<>)`），它使用該類型的兩個值並產生另一個值。 目的是：`mempty` 是 `<>` 的標識，而 `<>` 是關聯的； 也就是說，對於所有 `x`, `y` 和 `z`

1. `mempty <> x == x`
2. `x <> mempty == x`
3. `(x <> y) <> z == x <> (y <> z)`

關聯法則意味著我們可以明確地寫

```haskell
a <> b <> c <> d <> e
```

因為無論括號如何，我們都將得到相同的結果

還有 `mconcat`，用於組合整個值列表。 默認情況下，它是使用 `foldr` 實現的，但是它包含在 `Monoid` 類中，因為 `Monoid` 的特定實例可能具有更有效的實現方式

`Monoid` 一旦知道要尋找它們，它就會出現在*任何地方*。讓我們編寫一些實例（僅用於實踐；這些都在標準庫中）

> `Monoid`s show up *everywhere*, once you know to look for them. Let’s write some instances (just for practice; these are all in the standard libraries).

列表在串聯下形成一個 monoid：

```haskell
instance Monoid [a] where
  mempty  = []
  mappend = (++)
```

如上所示，加法在整數（或有理數或實數……）上定義了一個非常好的 monoid。但是，乘法也是如此！該怎麼辦？我們不能將相同類型的兩個不同實例賦予相同類型。相反，我們創建兩個 `newtype`，每個實例一個：

```haskell
newtype Sum a = Sum a
  deriving (Eq, Ord, Num, Show)

getSum :: Sum a -> a
getSum (Sum a) = a

instance Num a => Monoid (Sum a) where
  mempty  = Sum 0
  mappend = (+)

newtype Product a = Product a
  deriving (Eq, Ord, Num, Show)

getProduct :: Product a -> a
getProduct (Product a) = a

instance Num a => Monoid (Product a) where
  mempty  = Product 1
  mappend = (*)
```

請注意，要使用 `mconcat` 查找整數列表的乘積，我們必須首先將它們轉換為 `Product Integer` 類型的值：

```haskell
lst :: [Integer]
lst = [1,5,8,23,423,99]

prod :: Integer
prod = getProduct . mconcat . map Product $ lst
```

（當然，這個特定示例很愚蠢，因為我們可以改用標準 `product` 函數，但是這種模式確實很方便）

只要各個組件都成對 (Pairs)，對就形成一個 monoid：

```haskell
instance (Monoid a, Monoid b) => Monoid (a,b) where
  mempty = (mempty, mempty)
  (a,b) `mappend` (c,d) = (a `mappend` c, b `mappend` d)
```

挑戰：您可以為 `Bool` 製作 `Monoid` 實例嗎？ 有多少個不同的實例？

挑戰：如何使函數類型成為 `Monoid` 的實例？
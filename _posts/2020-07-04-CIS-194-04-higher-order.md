---
date: 2020-07-04 19:00:00
layout: post
title: "CIS 194 04 Higher-order programming and type inference"
subtitle: 
description: 
image: https://cdn.jsdelivr.net/gh/CWKSC/MyResources/Common/pixel300x1_Transparent.png
optimized_image: 
category: CIS 194 spring 2013
tags:
  - CIS 194 spring 2013
  - Haskell
author: CWKSC
paginate: false
math: true
---

Source: [04-higher-order](https://www.seas.upenn.edu/~cis194/spring13/lectures/04-higher-order.html)

## ▌Higher-order programming and type inference

## 高階編程和類型推斷

CIS 194 第 4 週
2013 年 2 月 4 日

建議閱讀：

- *Learn You a Haskell for Great Good* chapter “Higher-Order Functions” (Chapter 5 in the printed book; [Chapter 6 online](http://learnyouahaskell.com/higher-order-functions))

## ▌Anonymous functions 匿名函數

假設我們要寫一個函數

```haskell
greaterThan100 :: [Integer] -> [Integer]
```

僅保留輸入列表中大於100的那些Integer。例如

```haskell
greaterThan100 [1,9,349,6,907,98,105] = [349,907,105].
```

到目前為止，我們知道一種執行此操作的好方法：

```haskell
gt100 :: Integer -> Bool
gt100 x = x > 100

greaterThan100 :: [Integer] -> [Integer]
greaterThan100 xs = filter gt100 xs
```

但是給 `gt100` 命名很煩人，因為我們可能永遠不會再使用它了。 相反，我們可以使用匿名函數，也稱為 *lambda* 抽象：

```haskell
greaterThan100_2 :: [Integer] -> [Integer]
greaterThan100_2 xs = filter (\x -> x > 100) xs
```

`\x -> x > 100`（反斜線看起來應該像缺少短腿的 lambda 一樣）是一種函數，它接受單個參數 `x` 並輸出是否 `x` 大於100。

Lambda 抽象可以有多個參數。例如：

```haskell
Prelude> (\x y z -> [x,2*y,3*z]) 5 6 3
[5,12,9]
```

但是，在大於 100 的特定情況下，有一種更好的方式編寫它，而無需使用 lambda 抽象：

```haskell
greaterThan100_3 :: [Integer] -> [Integer]
greaterThan100_3 xs = filter (>100) xs
```

`(>100)`是*運算符部分*：如果 `?` 是運算符，則 `(?y)` 等效於函數 `\x -> x ? y`，並且 `(y?)` 等效於 `\x -> y ? x` 。換句話說，使用運算符部分可以使我們將運算符部分應用於其兩個參數之一。 我們得到的是單個參數的函數。 這裡有些例子：

```haskell
Prelude> (>100) 102
True
Prelude> (100>) 102
False
Prelude> map (*6) [1..5]
[6,12,18,24,30]
```

## ▌Function composition 函數組成

在繼續閱讀之前，您可以寫下類型為

```haskell
(b -> c) -> (a -> b) -> (a -> c)
```

？

我們試試吧。它必須接受兩個參數（均為函數），然後輸出一個函數。

```haskell
foo f g = ...
```

在 `...` 處，我們需要編寫一個類型為 `a-> c` 的函數。好吧，我們可以使用 lambda 抽象創建一個函數：

```haskell
foo f g = \x -> ...
```

`x` 將具有類型 `a`，現在在 `...` 中，我們需要編寫類型為 `c` 的表達式。 好吧，我們有一個函數 `g` 可以將 `a` 變成`b`，一個函數 `f` 可以將 `a` 變成 `c`，這應該可以工作：

```haskell
foo :: (b -> c) -> (a -> b) -> (a -> c)
foo f g = \x -> f (g x)
```

（快速測驗：為什麼我們需要括號`g x`？）

好，那是什麼意思呢？是否`foo`真正做任何有用的或是只是一個無聊的練習與類型的工作？

事實證明，`foo`它實際上是`(.)`，並且代表*函數的組成*。也就是說，如果`f`和`g`是函數，則`f . g`先執行`g`然後再執行的功能`f`。

在編寫簡潔，優雅的代碼時，函數組合可能非常有用。它非常適合“全餐”風格，在這裡我們考慮將數據結構的連續高級轉換組合在一起。

例如，考慮以下功能：

```haskell
myTest :: [Integer] -> Bool
myTest xs = even (length (greaterThan100 xs))
```

我們可以將其重寫為：

```haskell
myTest' :: [Integer] -> Bool
myTest' = even . length . greaterThan100
```

這個版本使發生的事情更加清晰：`myTest'`只是由三個較小函數組成的“管道”。此示例還說明了為什麼函數組合看起來“向後”：這是因為函數應用程序向後！因為我們從左到右閱讀，所以將值也從左到右流動是有意義的。但是在那種情況下，我們應該寫\（（x）f \）來表示給定值\（x \）作為函數\（f \）的輸入。但是，不用謝亞歷克西斯·克勞德·克萊洛特和歐拉，自1734年以來，我們就一直堅持使用倒數表示法。

讓我們仔細看看的類型`(.)`。如果我們要求`ghci`它的類型，我們得到

```haskell
Prelude> :t (.)
(.) :: (b -> c) -> (a -> b) -> a -> c
```

等一下。這裡發生了什麼？括號內發生了什麼`(a -> c)`？

## ▌Currying and partial application 柯里和部分應用

還記得多參數函數的類型看起來多麼奇怪，就像它們中有“額外”箭頭一樣嗎？例如考慮功能

```haskell
f :: Int -> Int -> Int
f x y = 2*x + y
```

我之前曾承諾過，這有一個美麗而深刻的原因，現在終於可以揭露它了：*Haskell中的所有函數都只接受一個參數*。說什麼？！但是`f`上面顯示的函數不接受兩個參數嗎？不，實際上，它不是：它接受一個參數（一個`Int`）並*輸出一個函數*（類型`Int -> Int`）；該函數接受一個參數並返回最終答案。實際上，我們可以這樣等效地編寫`f`的類型：

```haskell
f' :: Int -> (Int -> Int)
f' x y = 2*x + y
```

特別要注意的是，*右側的*功能箭頭`W -> X -> Y -> Z`等效於`W -> (X -> (Y -> Z))`。我們始終可以在類型中最右邊的頂級箭頭周圍添加或刪除括號。

而功能應用程序則*保持左*關聯。也就是說，`f 3 2`確實是的簡寫`(f 3) 2`。考慮到我們之前所說的關於`f`實際接受一個參數並返回一個函數`f`的說法`3`，這是有道理的：我們適用於一個參數，該參數返回一個類型為type的函數`Int -> Int`，即一個將an `Int`加6 的函數。然後，我們`2`通過編寫將該函數應用於自變量`(f 3) 2`，從而得到一個`Int`。由於函數應用程序關聯到左側，因此我們可以縮寫`(f 3) 2`為`f 3 2`，從而`f`為“多參數”函數提供了一個很好的表示法。

“多參數” lambda抽象

```haskell
\x y z -> ... 
```

真的只是語法糖

```haskell
\x -> (\y -> (\z -> ...)).  
```

同樣，函數定義

```has
f x y z = ... 
```

是語法糖

```haskell
f = \x -> (\y -> (\z -> ...)).
```

請注意，例如，我們可以通過將的`\x -> ...`從右側的移動`=`到左側來從上方重寫合成函數：

```haskell
comp :: (b -> c) -> (a -> b) -> a -> c
comp f g x = f (g x)
```

將多參數函數表示為單參數函數返回函數的想法被稱為*currying*，以英國數學家和邏輯學家Haskell Curry的名字命名。（他的名字聽起來很耳熟；是的，是同一個人。）庫裡（Curry）居住於1900年至1982年，一生都在賓夕法尼亞州立大學（Penn State）工作-但他還幫助在UPenn從事ENIAC工作。將多參數函數表示為單參數函數返回函數的想法實際上是由*MosesSchönfinkel*首次發現的，因此我們應該將其*稱為schönfinkeling*。庫裡本人將這個想法歸因於舍芬克爾，但其他人已經開始稱其為“咖哩”，為時已晚。

如果要實際表示兩個參數的函數，則可以使用一個元組的單個參數。即功能

```haskell
f'' :: (Int,Int) -> Int
f'' (x,y) = 2*x + y
```

也可以認為是“兩個論點”，儘管從另一種意義上說，它實際上只接受了一個恰好是一對的論點。為了在兩個參數的函數的兩種表示形式之間進行轉換，標準庫定義了稱為`curry`和的函數`uncurry`，其定義如下（除非使用不同的名稱）：

```haskell
schönfinkel :: ((a,b) -> c) -> a -> b -> c
schönfinkel f x y = f (x,y)

unschönfinkel :: (a -> b -> c) -> (a,b) -> c
unschönfinkel f (x,y) = f x y
```

`uncurry`特別是當您有一對想要對它應用功能時，該功能很有用。例如：

```haskell
Prelude> uncurry (+) (2,3)
5
```

**▌部分申請**

Haskell中的函數經過咖哩處理的事實使*部分應用*特別容易。部分應用的想法是，我們可以採用多個參數的函數，然後將其應用於*其中的某些*參數，然後得出其餘參數的函數。但是正如我們剛剛看到的那樣，在Haskell *中沒有*多重參數的功能！每個函數都可以“部分地”應用到其第一個（也是唯一一個）參數，從而產生其餘參數的功能。

請注意，Haskell使其難以部分應用於第一個參數以外的其他參數。一個例外是中綴運算符，正如我們已經看到的那樣，可以使用運算符部分將其部分應用於兩個自變量。實際上，這並不是很大的限制。確定一種函數的參數順序以使它的部分應用盡可能有用是一種技巧：參數應從“最小變化到最大變化”進行排序，也就是說，通常應相同的參數應為首先列出，然後通常會有所不同的論點應該放在最後。

**▌全餐編程**

讓我們將一個示例中剛剛學到的東西放在一起，它還展示了“全餐”編程風格的強大功能。考慮功能`foobar`，定義如下：

```haskell
foobar :: [Integer] -> Integer
foobar []     = 0
foobar (x:xs)
  | x > 3     = (7*x + 2) + foobar xs
  | otherwise = foobar xs
```

這似乎很簡單，但是不是很好的Haskell樣式。問題是

- 一次做太多事情；和
- 工作水平太低。

不必考慮要對每個元素做什麼，我們可以考慮使用已知的現有遞歸模式對整個輸入進行增量轉換。這是以下情況的慣用實現`foobar`：

```haskell
foobar' :: [Integer] -> Integer
foobar' = sum . map (\x -> 7*x + 2) . filter (>3)
```

這定義`foobar'`為三個函數的“管道”：首先，我們將列表中不大於三個的所有元素丟棄。接下來，我們對其餘列表的每個元素進行算術運算；最後，我們對結果求和。

請注意，在上面的示例中，`map`和`filter`已部分應用。例如，類型`filter`為

```haskell
(a -> Bool) -> [a] -> [a]
```

將它應用於`(>3)`（具有類型的`Integer -> Bool`）會產生一個類型的函數`[Integer] -> [Integer]`，這與將另一個函數組合在一起是正確的事情`[Integer]`。

編碼的這種風格中，我們定義一個函數不參照它的參數，在一定意義上說的功能什麼*的*，而不是什麼它*做* -is被稱為“自由點”的風格。從上面的示例可以看出，它可能非常漂亮。甚至有人甚至說您應該始終努力使用無點樣式。但是太遠了，它會變得非常混亂。`lambdabot`在`#haskell`IRC信道有一個命令`@pl`用於使功能集成到相當的自由點表達式; 這是一個例子：

```haskell
@pl \f g x y -> f (x ++ g x) (g y)
join . ((flip . ((.) .)) .) . (. ap (++)) . (.)
```

這顯然*不是*一種改善！

## ▌褶皺

列表上還有一個要討論的遞歸模式：折疊。以下是列表中的一些函數，它們遵循類似的模式：所有這些函數都以某種方式將列表中的元素“組合”為最終答案。

```haskell
sum' :: [Integer] -> Integer
sum' []     = 0
sum' (x:xs) = x + sum' xs

product' :: [Integer] -> Integer
product' [] = 1
product' (x:xs) = x * product' xs

length' :: [a] -> Int
length' []     = 0
length' (_:xs) = 1 + length' xs
```

這三個功能有什麼共同點，有什麼不同？與往常一樣，該想法將是藉助定義高階函數的能力來抽像出變化的部分。

```haskell
fold :: b -> (a -> b -> b) -> [a] -> b
fold z f []     = z
fold z f (x:xs) = f x (fold z f xs)
```

請注意如何`fold`基本上取代了`[]`用`z`並`(:)`用`f`，也就是說，

```haskell
fold f z [a,b,c] == a `f` (b `f` (c `f` z))
```

（如果`fold`從這個角度考慮，您也許可以弄清楚如何歸納`fold`為列表以外的數據類型……）

現在，讓我們改寫`sum'`，`product'`和`length'`在以下方面`fold`：

```haskell
sum''     = fold 0 (+)
product'' = fold 1 (*)
length''  = fold 0 (\_ s -> 1 + s)
```

（代替`(\_ s -> 1 + s)`我們也可以寫`(\_ -> (1+))`，甚至`(const (1+))`。）

當然，`fold`已經在標準Prelude中以名稱提供了[`foldr`](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#v:foldr)。參數的`foldr`順序略有不同，但功能完全相同。以下是一些Prelude函數，它們的定義如下`foldr`：

- `length` `:: [a] -> Int`
- `sum` `:: Num a => [a] -> a`
- `product` `:: Num a => [a] -> a`
- `and` `:: [Bool] -> Bool`
- `or` `:: [Bool] -> Bool`
- `any` `:: (a -> Bool) -> [a] -> Bool`
- `all` `:: (a -> Bool) -> [a] -> Bool`

還存在[`foldl`](http://haskell.org/ghc/docs/latest/html/libraries/base/Prelude.html#v:foldl)“從左側”折疊的。那是，

```haskell
foldr f z [a,b,c] == a `f` (b `f` (c `f` z))
foldl f z [a,b,c] == ((z `f` a) `f` b) `f` c
```

但是，通常應該使用[`foldl'`from`Data.List`](http://haskell.org/ghc/docs/latest/html/libraries/base/Data-List.html#v:foldl)代替，它的作用與之相同，`foldl`但效率更高。
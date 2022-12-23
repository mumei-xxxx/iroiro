---
title: "魔理沙「PureScript Jelly で、Reactチュートリアル三目並べの《純粋関数型／型安全》版を作るぜ」①"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PureScript"]
published: false
---

> **Moggi**は、計算効果を含んだプログラムを**Kleisli**圏の射とみなすことによって、圏論的に物事をうまく取り扱う方法を提唱した。
> 計算効果をモナドによって捉えると、モナドにおける自然変換$μ$が「分岐をうまくまとめる」重要な働きをすることがわかる。
>
> 『圏論の道案内 ~矢印でえがく数学の世界~』[^1]


霊夢「PureScript Advent Calendar 2022。24日目の記事です。」

https://qiita.com/advent-calendar/2022/purescript

## 完成したもの

https://github.com/mumei-xxxx/purescript-sanmoku-dev0/tree/main/jelly-tic-tac-toe-without-timemachine

![tic-tac-toe](/images/purescript-jelly-tic-tac-toe-01/tic-tac-toe.gif)

## 概略

魔理沙「Reactのチュートリアルに三目並べがある。」

https://ja.reactjs.org/tutorial/tutorial.html

@[codepen](https://codepen.io/gaearon/pen/LyyXgK?editors=0010)

霊夢「三目ならべって、何？」
魔理沙「三目ならべとは、マルバツゲームだ。一手ごとに交互に升目を○か×で埋めていって、縦か横か斜め一列になったものが勝ちなんだぜ。英語でtic-tac-toeという。」
霊夢「なるほどね。」
魔理沙「これを今回は、PureScript で作ろうと思う。」
霊夢「なんで、PureScript で作るの？」
魔理沙「本当に自分の真っ正直なことを言えば、なんとなくいい感じだからだ。」
霊夢「なんとなくいい感じ」
魔理沙「純粋関数型言語で、フロントエンドを作るというのは面白いテーマだと思うぜ。」
魔理沙「素朴なプログラムに数学的、圏論的な意味がある。」
魔理沙「例えば、do構文のなかの`pure`。」
魔理沙「つまり、モナドの`pure`は、型 `a` を受け取って 型 `m a` を返すわけだが。」
魔理沙「これは、恒等関手から `m` への自然変換だな。」
魔理沙「圏論の創始者のマックレーンは、次のように言っている。」

> 関手を研究するために圏を考え出したのではない。
> 自然変換を研究するためだったのだ。[^2]

魔理沙「自然変換という概念は、圏論ではじめて厳密な定義が与えられた。」
魔理沙「その自然変換を、今、人類は、`pure` のような形で利用しているわけだ。」
魔理沙「そして、`Maybe` やリストのような一見単純な構造に、モナド、自然変換が潜んでいる。」
魔理沙「見方を変えれば、これは非常に面白いことだ。」
魔理沙「多分、クライスリ圏とかモナド構造というのは、世界にありふれているのだと思う。それを利用しないだけで。」
魔理沙「Haskell / PureScriptといった道具を使うことで、世界に隠された構造を使うことができる。」
魔理沙「こういった数学的な道具を使って組み立てられた世界で、どういう景色が見えてくるか。」
魔理沙「これは、とても興味深いと思うぜ。」

霊夢「なるほどね。それで、今回は、どんなフレームワークを使うの？」
魔理沙「PureScript のWebフレームワークといえば、Halogen が有名だ。しかし、今回は、Zennでも記事を公開されていたゆきくらげさんが作ったフレームワークである。PureScript Jelly を使うぜ。」

https://jelly.yukikurage.net/

霊夢「なんで Halogen は使わないの？」
魔理沙「Halogen は単純に複雑で、挫折する可能性が高いと思ったぜ。」
霊夢「『単純に複雑』ね。」
魔理沙「また、PureScript Jelly は、SolidJS を意識していたり、割とReactライクに書けるのではと思ったからだぜ。」
霊夢「なるほどね。」
魔理沙「また、Reactの三目ならべのチュートリアルは、タイムマシン機能というものがある。端的に言うと、ゲーム履歴機能だ。」
魔理沙「しかし、これを入れると、ややロジックが込み入って、見せたい、PureScript や Jelly の部分をシンプルに見れられなくなる。」
魔理沙「だから、今回は、タイムマシン機能をのぞいた、純粋な三目ならべの部分を実装していくぜ。」
霊夢「了解した。」

> 仮想 DOM を使わない Web フレームワーク 'Jelly' を作った in PureScript

https://zenn.dev/yukikurage/articles/4735819c3b421b

## 環境/バージョン情報

魔理沙「バージョン情報だ。」
魔理沙「Jelly は現在、絶賛発展中のフレームワークだ。今後、破壊的変更で、この記事に書いてあることが、古くなる可能性は了承しておいて欲しいぜ。」

Windows Home 11 / WSL / Ubuntu 20.04
Node.js 18.12.1
npm 8.19.2
PureScript 0.15.6
PureScript Jelly 0.8.1
PureScript Jelly Signal 0.3.0

## ディレクトリ構成

魔理沙「これが今回のプロジェクトのディレクトリ構成だ。」
魔理沙「`src` 配下で開発して、トランスパイルされたものが、`public` 配下に格納されるぜ。」

```
.
├── package.json
├── package-lock.json
├── packages.dhall
├── public
│   ├── index.css.map
│   ├── index.html
│   ├── index.js
│   ├── style.css
│   └── style.css.map
├── README.md
├── spago.dhall
├── src
│   ├── Components
│   │   ├── Board.purs
│   │   └── Square.purs
│   ├── Main.purs
│   ├── style.scss
│   └── UseCases
│       └── calculateWinner.purs
└── test
    └── Main.purs
```

## 環境構築について

魔理沙「環境構築に関しては、PureScript Advent Calendar。 2022 1日目のゆきくらげさんの記事を参考にさせてもらったぜ。」

> PureScript + Halogen + Tailwind で Web フロント開発環境構築 (in VSCode)

https://qiita.com/yukikurage_2019/items/a2c7b0d1d2c34aee120c


魔理沙「違いは、Tailwind ではなく、Sassを使っていることだぜ。」

```json:package.json
"scripts": {
    "install": "npx spago install",
    "bundle:script": "npx spago bundle-app -t ./public/index.js -y",
    "bundle:scss": "sass --error-css --style=compressed ./src/style.scss ./public/style.css",
    "bundle": "run-s bundle:*",
    "watch:script": "npx spago bundle-app -t ./public/index.js -w",
    "watch:scss": "sass --watch --error-css ./src/style.scss ./public/style.css",
    "watch:server": "npx live-server ./public",
    "watch": "run-p watch:*"
},
```

## calculateWinner 勝者判定

魔理沙「まず、この記事では、`calculateWinner` 関数について解説するぜ。」
魔理沙「コンポーネントについては、次の記事で解説するぜ。」
霊夢「了解した。」
魔理沙「とりあえず、以前、TypeScript で作ったコードがあるから、それをみよう。」
魔理沙「TypeScriptで作ったコードの詳細については以前書いた以下の記事を参考にして欲しい。」

> Next.js + TypeScript + Recoil + Herp社ESLint Config でReactチュートリアルを作る。

https://zenn.dev/purenium/articles/nextjs-recoil-tic-tac-toe


```ts:src/useCases/calculateWinner.ts
/**
 * @description [null, null, null, null, null, null, 'X', null, 'O']のような配列
 */
type SquareValueType = "X" | "O" | null
type BoardArrType = SquareValueType[]

/**
 * @description 勝者を判定する。
 */
export const calculateWinner = (squares: BoardArrType): SquareValueType => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ]

  for (const line of lines) {
    const [a, b, c] = line
    // 縦、横、対角線 で3つXあるいは、Oが連続すれば、連続したvalueを返す。
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a]
    }
  }
  return null
}

```

魔理沙「ここで、`calculateWinner` の書き換えだが、まず、`for`文のなかで、勝者があれば、それを返す。」
魔理沙「それ以外だと `null` を返す。」
魔理沙「これをPureScriptで書き換えるのだが、うまくいかなかった。」
魔理沙「だから、PureScript のDiscordのコミュニティで質問したのだが、そこでゆきくらげさんに教えもらってなんとかいけたのだぜ。」
霊夢「ゆきくらげさんありがとう。」

https://zenn.dev/yukikurage/articles/482a8647421fd5

魔理沙「そこで3パターンくらい教えていただいたからそれを紹介するぜ。」
魔理沙「これ用に別のプロジェクトを作ったから見てほしい。」

https://github.com/mumei-xxxx/purescript-sanmoku-dev0/tree/main/purescript-calculatewinner-examples

```purs:src/Main.purs
module Main where

import Prelude (class Eq, class Ord, class Show, bind, discard, pure, ($), (==), join)
import Control.Alternative (guard)
import Data.Array (all, head, (!!), find)
import Data.Generic.Rep (class Generic)
import Data.Maybe (Maybe(..), isJust)
import Data.Show.Generic (genericShow)

lines :: Array (Array Int)
lines =
  [ [ 0, 1, 2 ]
  , [ 3, 4, 5 ]
  , [ 6, 7, 8 ]
  , [ 0, 3, 6 ]
  , [ 1, 4, 7 ]
  , [ 2, 5, 8 ]
  , [ 0, 4, 8 ]
  , [ 2, 4, 6 ]
  ]

data SquareValueType = X | O

derive instance Eq SquareValueType
derive instance Ord SquareValueType
derive instance Generic SquareValueType _

instance Show SquareValueType where
  show = genericShow

type Board = Array (Maybe SquareValueType)

{-
  https://pursuit.purescript.org/packages/purescript-arrays/7.1.0/docs/Data.Array#v:all
-}

{-
  find https://pursuit.purescript.org/packages/purescript-arrays/7.1.0/docs/Data.Array#v:find
  isJust
  https://pursuit.purescript.org/packages/purescript-maybe/6.0.0/docs/Data.Maybe#v:isJust
-}


calculateWinner1 :: Board -> Maybe SquareValueType
calculateWinner1 boardArr =
  let
    -- | ある Line が同じ SquareValueType で埋まっているかどうか判定する
    isLineMatch :: Array Int -> SquareValueType -> Boolean
    isLineMatch line squareValue = all (\i -> boardArr !! i == Just (Just squareValue)) line

    -- | すべての Line, SquareValueType の組み合わせについて isLineMatch を評価する
    checked :: Array (Maybe SquareValueType)
    checked = do
      line <- lines
      squareValue <- [ X, O ]
      pure
        if isLineMatch line squareValue then
          Just squareValue
        else
          Nothing
  in
    -- | checked の中で一番最初に Just が出てきたものを返す
    -- | find で帰ってくるのは Maybe (Maybe SquareValueType) なので、join で一つ unwrap する
    join $ find isJust checked


calculateWinner2 :: Board -> Maybe SquareValueType
calculateWinner2 boardArr =
  let
    -- | ある Line が同じ SquareValueType で埋まっているかどうか判定する
    isLineMatch :: Array Int -> SquareValueType -> Boolean
    isLineMatch line squareValue = all (\i -> boardArr !! i == Just (Just squareValue)) line

    -- | すべての Line, SquareValueType の組み合わせについて isLineMatch を評価する
    -- | false ならそれはスキップする
    checked :: Array SquareValueType
    checked = do
      line <- lines
      squareValue <- [ X, O ]
      guard $ isLineMatch line squareValue
      pure squareValue
  in
    head checked

{-
  [X, O, Nothing, X, O, O, X, O, O]
  [Just X, Just O, Nothing, Just X, Just O, Just O, Just X, Just O, Just O]

  calculateWinner $ map Just [X, O, X, X, O, O, X, O, O]
(Just X)
-}
calculateWinner :: Board -> Maybe SquareValueType
calculateWinner boardArr = head do
  line <- lines
  squareValue <- [ X, O ]
  guard $ all (\i -> boardArr !! i == Just (Just squareValue)) line
  pure squareValue

```

魔理沙「まず、以下の型から説明していく。」

```purs
data SquareValueType = X | O
```

魔理沙「これが、升目の値を表す型だ。要するにマルバツゲームだから、`X` か `O`かということだ。」
霊夢「なるほどね。」
魔理沙「ただ、PureScript には、`null` がない。だから、`X` か `O` か `Nothing` かの Maybe値の配列で値を考えていくことにする。つまり、」

```purs
type Board = Array (Maybe SquareValueType)
```

魔理沙「ということだ。」

## パターン① let-in式 と if式。

```purs
calculateWinner1 :: Board -> Maybe SquareValueType
calculateWinner1 boardArr =
  let
    -- | ある Line が同じ SquareValueType で埋まっているかどうか判定する
    isLineMatch :: Array Int -> SquareValueType -> Boolean
    isLineMatch line squareValue = all (\i -> boardArr !! i == Just (Just squareValue)) line

    -- | すべての Line, SquareValueType の組み合わせについて isLineMatch を評価する
    checked :: Array (Maybe SquareValueType)
    checked = do
      line <- lines
      squareValue <- [ X, O ]
      pure
        if isLineMatch line squareValue then
          Just squareValue
        else
          Nothing
  in
    -- | checked の中で一番最初に Just が出てきたものを返す
    -- | find で帰ってくるのは Maybe (Maybe SquareValueType) なので、join で一つ unwrap する
    join $ find isJust checked
```

魔理沙「まず、`calculateWinner1`から見ていく。」
魔理沙「`let~in`式と`if`式を使うパターンだ。」

```purs
-- | ある Line が同じ SquareValueType で埋まっているかどうか判定する
isLineMatch :: Array Int -> SquareValueType -> Boolean
isLineMatch line squareValue = all (\i -> boardArr !! i == Just (Just squareValue)) line
```

魔理沙「`isLineMatch` はひとつの列について、同じ値で埋まっているかをチェックする関数だ。」
魔理沙「まず、`Data.Array` の `all` 関数について。」

https://pursuit.purescript.org/packages/purescript-arrays/7.1.0/docs/Data.Array#v:all

```purs
all :: forall a. (a -> Boolean) -> Array a -> Boolean
```

```purs:例
all (_ > 0) [] = True
all (_ > 0) [1, 2, 3] = True
all (_ > 0) [-1, -2, -3] = False
```
魔理沙「まず第一引数に条件式、第二引数に配列をとる。」
魔理沙「例のように、配列の値のすべてが、条件式に対して、真になる場合は`true`を返す。それ以外は、`false`を返すぜ。」
魔理沙「`boardArr`というのは、」

```purs
[Just X, Just O, Nothing, Just X, Just O, Just O, Just X, Just O, Just O]
```

魔理沙「みたいな値だな。」
魔理沙「`line`というのは、`[ 0, 1, 2 ]` のような値だ。」
魔理沙「これで、ひとつの列が、3つ`X`で埋まっているか、それとも`O`で埋まっている場合は、`true`を返す。」
霊夢「なるほどね。」

```purs
-- | すべての Line, SquareValueType の組み合わせについて isLineMatch を評価する
checked :: Array (Maybe SquareValueType)
checked = do
  line <- lines
  squareValue <- [ X, O ]
  pure
    if isLineMatch line squareValue then
      Just squareValue
    else
      Nothing
```

魔理沙「つぎの`checked`だ。」
魔理沙「Arrayはモナドなので、do構文が使えるな。」
魔理沙「`line <- lines`、`squareValue <- [ X, O ]`あたりは、配列なのだが、Haskellのリストモナドと同じ動きをする。」
魔理沙「つまり、`line`、`squareValue`が取り得る値に対して、総当たりで、後続の処理を行う。」
魔理沙「そのあとは、if 式で、さきほどの`isLineMatch`が真なら、その値`Just squareValue`を返す。偽なら、`Nothing`を返している。」
魔理沙「do構文なので、それを`pure`で`Array`として返しているという流れだ。」

```purs
in
  -- | checked の中で一番最初に Just が出てきたものを返す
  -- | find で帰ってくるのは Maybe (Maybe SquareValueType) なので、join で一つ unwrap する
  join $ find isJust checked
```

魔理沙「次は、`let-in` 式の`in` のなかだ。」
魔理沙「最終的な返り値だ。」
魔理沙「まず、`find` だ。これは、`Data.Array`の関数で、名前の通り、条件式が真のものを配列の中から見つけるという関数だ。」

```purs
find :: forall a. (a -> Boolean) -> Array a -> Maybe a
```
```purs
find (contains $ Pattern "b") ["a", "bb", "b", "d"] = Just "bb"
find (contains $ Pattern "x") ["a", "bb", "b", "d"] = Nothing
```
https://pursuit.purescript.org/packages/purescript-arrays/7.1.0/docs/Data.Array#v:find

魔理沙「第一引数に条件式、第二引数に対象の配列を取る。」
魔理沙「条件を満たすものがあれば、`Just a`、なければ`Nothing`を返す。」
魔理沙「つぎに、`isJust`だが、`Maybe`型の`Just`の場合なら、`true`を返す関数だな。」

https://pursuit.purescript.org/packages/purescript-maybe/6.0.0/docs/Data.Maybe#v:isJust

魔理沙「これで、結果が得られるわけだが、コメントにもあるように、findは、`Maybe a`を返す。」
魔理沙「だから、`Maybe (Maybe SquareValueType)`の値が返ってきてしまう。」
魔理沙「そこで`join` 関数の出番だ。」
魔理沙「二重になっているモナドのデータ構造を一重にできるぜ。」

https://pursuit.purescript.org/packages/purescript-prelude/6.0.1/docs/Control.Bind#v:join

```purs
join :: forall a m. Bind m => m (m a) -> m a
```

魔理沙「つまり、`join $ find isJust checked` で `Maybe SquareValueType` を得られるわけだ。」

これをPureScriptの対話型PSCiで試してみるぜ。

```
❯ npx spago repl
PSCi, version 0.15.7
Type :? for help

import Prelude

> import Main
> import Data.Maybe
> calculateWinner1 $ map Just [X, O, X, X, O, O, X, O, O]
(Just X)
```

魔理沙「という感じで成功した。」
魔理沙「注意点としては、`calculateWinner1` は、`Array (Maybe SquareValueType)` を引数にとる。」
魔理沙「だから、`map Just ●●`で、配列を`Array (Maybe SquareValueType)`に変換することだ。」

## パターン② パターン① let-in式 で guard
。
魔理沙「上で解説した、`calculateWinner1`だが、これはもっと短く書ける。」
魔理沙「それが、`calculateWinner2`だ。」

```purs
calculateWinner2 :: Board -> Maybe SquareValueType
calculateWinner2 boardArr =
  let
    -- | ある Line が同じ SquareValueType で埋まっているかどうか判定する
    isLineMatch :: Array Int -> SquareValueType -> Boolean
    isLineMatch line squareValue = all (\i -> boardArr !! i == Just (Just squareValue)) line

    -- | すべての Line, SquareValueType の組み合わせについて isLineMatch を評価する
    -- | false ならそれはスキップする
    checked :: Array SquareValueType
    checked = do
      line <- lines
      squareValue <- [ X, O ]
      guard $ isLineMatch line squareValue
      pure squareValue
  in
    head checked
```

魔理沙「`[]`は、`Alternative` 型クラスのインスタンスだから、`guard` が使える。」
魔理沙「この `guard` 条件式 を使えば、条件に対して、真でない値を取り除けるぜ。」
魔理沙「これを、`in` の中で配列から値を取り出せば、終わりだ。`head` を使えばいいぜ。」
魔理沙「ちなみに、PureScriptの `Data.Array` の型は以下のようになっている。」

```purs
head :: forall a. Array a -> Maybe a
```

https://pursuit.purescript.org/packages/purescript-arrays/4.0.1/docs/Data.Array#v:head

## パターン③ guard を活用して、let-in式もなくす。

魔理沙「パターン③は、`guard` を使う路線を徹底するぜ。」
魔理沙「それで、`let-in` 式もなくなったのが、次のバージョンだな。」

```purs
calculateWinner :: Board -> Maybe SquareValueType
calculateWinner boardArr = head do
  line <- lines
  squareValue <- [ X, O ]
  guard $ all (\i -> boardArr !! i == Just (Just squareValue)) line
  pure squareValue
```

霊夢「かなり短くなったわね。」
魔理沙「`line`と`squareValue`のすべての値を、`guard` でチェックして、合格した、`squareValue`を返す感じだ。」
魔理沙「今回はパターン③を使おうと思う。」
霊夢「モナドと `Data.Array` の関数はかなり強力ね。」

◆◆◆

魔理沙「というわけで、コンポーネントの解説をしだすと長くなるので今日はここまでとするぜ。」
霊夢「では、また明日。」

[^1]: 西郷甲矢人、能美十三『圏論の道案内 ~矢印でえがく数学の世界~』技術評論社 Kindle版 位置No.3983/4269
[^2]: 『圏論の道案内 ~矢印でえがく数学の世界~』 Kindle版 位置No.163/4269
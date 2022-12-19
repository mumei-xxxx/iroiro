---
title: "魔理沙「PureScript Jelly で、Reactチュートリアル三目並べの《純粋関数型／型安全》版を作るぜ」"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PureScript"]
published: false
---

> **Moggi**は、計算効果を含んだプログラムを**Kleisli**圏の射とみなすことによって、圏論的に物事をうまく取り扱う方法を提唱した。
> 計算効果をモナドによって捉えると、モナドにおける自然変換μが「分岐をうまくまとめる」重要な働きをすることがわかる。
>
> 『圏論の道案内 ~矢印でえがく数学の世界~』[^1]

PureScript Advent Calendar 2022 の記事です。
https://qiita.com/advent-calendar/2022/purescript

## 完成したもの

https://github.com/mumei-xxxx/purescript-sanmoku-dev0

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
魔理沙「理屈を言えば、いろいろあるが、完全になんとなくだぜ。」
霊夢「ずこー。」
魔理沙「純粋関数型！型安全！っていうのがなんとなくかっこいい。そして、PureScript は Haskell に影響を受けた言語だ。だから、PureScript 書いたコードは原則、数学の圏論的には、クライスリ圏の射の合成とみなすことができる。」
霊夢「プログラムをクライスリ圏の射の合成としたら、どうだっていうの？」
魔理沙「わからない。わからないが、自分で実際に作ってみて、どういう景色が見えるのかという知的好奇心だぜ。」
霊夢「」

霊夢「なるほどね。それで、今回は、どんなフレームワークを使うの？」
魔理沙「PureScript のWebフレームワークといえば、Halogen が有名だ。しかし、今回は、Zennでも記事を公開されていたゆきくらげさんが作ったフレームワークである。PureScript Jelly を使うぜ。」

https://jelly.yukikurage.net/

霊夢「なんで Halogen は使わないの？」
魔理沙「Halogen は単純に複雑で、挫折する可能性が高いと思ったぜ。」
霊夢「『単純に複雑』ね。」
魔理沙「また、PureScript Jelly は、SolidJSを意識していたり、割とReactライクに書けるのではと思ったからだぜ。」
霊夢「なるほどね。」
魔理沙「また、Reactの三目ならべのチュートリアルは、タイムマシン機能というものがある。端的に言うと、ゲーム履歴機能だ。」
魔理沙「しかし、これを入れると、ややロジックが込み入って、見せたい、PureScript や Jelly の部分をシンプルに見れられなくなる。」
魔理沙「だから、今回は、タイムマシン機能をのぞいた、純粋な三目ならべの部分を実装していくぜ。」
霊夢「了解した。」

仮想 DOM を使わない Web フレームワーク 'Jelly' を作った in PureScript
https://zenn.dev/yukikurage/articles/4735819c3b421b

## 環境/バージョン情報

魔理沙「バージョン情報だ。」
魔理沙「Jelly は発展中のフレームワークだ。今後、破壊的変更で、この記事に書いてあることが、古くなる可能性は了承しておいて欲しいぜ。」

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

魔理沙「環境構築に関しては、PureScript Advent Calendar 2022 1日目のゆきくらげさんの記事を参考にさせてもらったぜ。」

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

魔理沙「まず、コンポーネントについて、解説する前に、calculateWinner 関数についてだ。この部分については、別に記事を作って解説したいから、ここでは、簡単に触れるにとどめる。」
霊夢「了解した。」
魔理沙「とりあえず、以前、TypeScript で作ったコードがあるから、それと比較していくぜ。」
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

```purs:src/UseCases/calculateWinner.purs
module UseCases.Calculatewinner
  ( Board
  , SquareValueType(..)
  , calculateWinner
  , lines
  ) where

import Control.Alternative (guard)
import Data.Array (all, head, (!!))
import Data.Generic.Rep (class Generic)
import Data.Maybe (Maybe(..))
import Data.Show.Generic (genericShow)
import Prelude (class Eq, class Ord, class Show, bind, discard, pure, ($), (==))

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

calculateWinner :: Board -> Maybe SquareValueType
calculateWinner boardArr = head do
  line <- lines
  sv <- [ X, O ]
  guard $ all (\i -> boardArr !! i == Just (Just sv)) line
  pure sv

```

魔理沙「根本のロジック自体は、TypeScriptのコードと同じだ。」
魔理沙「まず、以下の型に注目だぜ。」

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
魔理沙「繰り返しになるが、`calculateWinner`関数については、別記事で解説するぜ。」

## square コンポーネント（升目部分）

魔理沙「コンポーネントについて解説していくぜ。」
魔理沙「まずは、三目ならべの升目部分のコンポーネントだ。」
魔理沙「これも、TypeScript で作ったコードと比較していくぜ。」

```tsx:src/components/Square.tsx
interface SquarePropsType {
  value: SquareValueType
  onClick: () => void
}

/**
 * @description 三目ならべの升目のコンポーネント
 */
export const Square: React.FC<SquarePropsType> = ({ value, onClick }) => {

  return (
    <button className="square" onClick={onClick}>
      {value}
    </button>
  )
}
```

魔理沙「次に掲載するのが PureScript のコードだ。」

```purs:src/Components/Square.purs
module Components.Square
  ( SquarePropsType,
    squareComponent
  )
  where

import Prelude

import Jelly.Component (class Component, textSig)
import Jelly.Element as JE
import Jelly.Prop ((:=), on)
import Jelly.Signal (Signal)
import Data.Maybe (Maybe(..))
import UseCases.Calculatewinner (SquareValue(..))
import Web.HTML.Event.EventTypes (click)

type SquarePropsType m =
  { onClick :: m Unit
  , value :: Signal (Maybe SquareValue) }

squareComponent :: forall m. Component m => SquarePropsType m -> m Unit
squareComponent { onClick, value } = do
  JE.button [ "class" := "square", on click \_ -> onClick ] do
    textSig $ value <#> case _ of
      Just X -> "X"
      Just O -> "O"
      Nothing -> ""

```

魔理沙「解説のため、コメントを多めに書いた。」
魔理沙「コメントと重複する部分もあるかもしれないが説明していくぜ。」
魔理沙「まず、`squareComponent` の引数の型が `SquarePropsType` だ。」
魔理沙「`onClick` についてはイベント関数だ。PureScript は、Haskell と同様、必ず返り値が必要なので、`m Unit` を返す。Unit はPureScript独自のもので、Haskell の空のタプル`()`と同じだぜ。」
魔理沙「続いては、`squareComponent`関数についてだ。」
魔理沙「`JE.button [ "class" := "square", on click \_ -> onClick ]`」
魔理沙「`[]`で囲まれている部分が、button要素のプロパティだ。」
魔理沙「`:=` と `on` は、Jelly のボイラープレートだ。CSSのクラス`square`を設定して、clickしたときに駆動する`onClick`関数を、`on click \_ -> onClick`で設定しているぜ。」
魔理沙「`textSig` も Jelly の関数だ。Signal値を画面表示したいときは、この関数を使うぜ。」
魔理沙「また、`textSig` の部分はパターンマッチングになっている。`value`の値に応じて条件分岐しているぜ。」

### functor の `<$>` と `<#>`

魔理沙「`<#>` 記号は functor の mapFlipped だ。」
魔理沙「これは、functor の map `<$>` の亜種だぜ。（Haskellでは fmap）」

https://pursuit.purescript.org/packages/purescript-prelude/3.0.0/docs/Data.Functor#v:(%3C#%3E)

霊夢「どう違うの？」
魔理沙「まず functor の map `<$>` について説明しよう。」
魔理沙「配列の map は、リスト `[a]`型の`a`型の要素に関数を適用できる。」
魔理沙「functor の map はこれをさらに一般化したものだ。」
魔理沙「型 `f a` の `a`型に関数を適用できる。」
魔理沙「これでなにがうれしいか。」
魔理沙「普通、`(a -> b)` の関数の引数には、`f a` のような型を取れない。型が異なるからだ。f a とは例えば、Maybe a のような型だ。」
魔理沙「だが、functor の map を使えば、`(a -> b)` の関数の引数に`f a`を適用できるぜ。」
魔理沙「そして、functor の map `<$>` は、引数をふたつとる。ひとつは関数`(a -> b)` と`f a` の値だ。」
魔理沙「そして、`<$>`と`<#>`の違いは、型を見ればわかる。」
魔理沙「`<$>` の型は、`(a -> b) -> f a -> f b`[^2] 」
魔理沙「`<#>` の型は、`f a -> (a -> b) -> f b`」
魔理沙「つまり、引数で、関数と`f a` の値をとる順番が逆ということだ。」
魔理沙「以下のような感じだ。」

```
関数 <$> 値
例）
(\n -> n * n) <$> [1, 2, 3]

値 <#> 関数
例）
[1, 2, 3] <#> \n -> n * n
```

魔理沙「`case _ of` は、`\x -> case x of` の略だから、以下は同じになる。」

```purs
(\x -> case x of 
  Just X -> "X"
  Just O -> "O"
  Nothing -> "") <$> value
```

```purs
value <#> case _ of
  Just X -> "X"
  Just O -> "O"
  Nothing -> ""
```
## board コンポーネント（升目が組み合わさった盤全体）

魔理沙「次は、board コンポーネントについて解説するぜ。」

```js
class Board extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null),
      xIsNext: true,
    };
  }

  handleClick(i) {
    const squares = this.state.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }

  renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
  }

  render() {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

```purs:src/Components/Board.purs
module Components.Board
  ( boardComponent
  ) where

import Prelude

import Data.Array (replicate, updateAt, (!!))
import Data.Maybe (Maybe(..), fromMaybe)
import Data.Tuple.Nested ((/\))
import Effect.Class (class MonadEffect)
import Jelly.Component (class Component, textSig)
import Jelly.Element as JE
import Jelly.Prop ((:=))
import Jelly.Signal (Signal, newState, readSignal, writeChannel)
import Components.Square (squareComponent)
import UseCases.Calculatewinner (SquareValueType(..), calculateWinner, nextPlayer)

data GameState = Winner SquareValueType | NextPlayer SquareValueType

derive instance Eq GameState

boardComponent :: forall m. Component m => m Unit
boardComponent = do
  squareArraySig /\ squareArrayChannel <- newState $ replicate 9 Nothing
  gameStateSig /\ gameStateChannel <- newState $ NextPlayer X

  let
    handleClick :: MonadEffect m => Int -> m Unit
    handleClick i = do
      squares <- readSignal squareArraySig
      gameState <- readSignal gameStateSig
      case gameState of
        Winner _ -> pure unit
        NextPlayer p -> when ((squares !! i) == Just Nothing) do
          let
            newSquares = fromMaybe squares $ updateAt (i :: Int) (Just p) squares
          writeChannel squareArrayChannel newSquares
          writeChannel gameStateChannel $ case calculateWinner newSquares of
            Nothing -> NextPlayer $ nextPlayer p
            Just w -> Winner w

    renderSquareComponent :: Component m => Int -> m Unit
    renderSquareComponent valueInt = do
      let
        valSig = do
          squares <- squareArraySig
          pure $ join $ squares !! valueInt
      squareComponent { onClick: handleClick valueInt, value: valSig }

    playStatus :: Signal String
    playStatus = gameStateSig <#> case _ of
      NextPlayer p -> "Next player: " <> show p
      Winner w -> "Winner: " <> show w

  JE.div' do
    JE.div' do
      textSig $ playStatus
    JE.div' do
      JE.div [ "class" := "board-row" ] do
        renderSquareComponent 0
        renderSquareComponent 1
        renderSquareComponent 2
    JE.div' do
      JE.div [ "class" := "board-row" ] do
        renderSquareComponent 3
        renderSquareComponent 4
        renderSquareComponent 5
    JE.div' do
      JE.div [ "class" := "board-row" ] do
        renderSquareComponent 6
        renderSquareComponent 7
        renderSquareComponent 8

```

魔理沙「まずは、以下の `GameState` について解説する。」

```purs
data GameState = Winner SquareValueType | NextPlayer SquareValueType
```

魔理沙「ここは、もとのJSのコードと違う部分だ。」
魔理沙「`data SquareValueType = X | O`だった。」
魔理沙「これはゲームの状態を表すぜ。勝者がいる状態。`Winner SquareValueType`」
魔理沙「つまり、勝者が `X` か `O` か。」
魔理沙「または、勝者がいない状態 `NextPlayer SquareValueType` は次の手番が、次の手番が、 `X` か `O` かということだ。」
魔理沙「この部分は、ゆきくらげさんに提案していただいたぜ。」
霊夢「自分で考えなさいよ。」
魔理沙「次は、`boardComponent`内の下の2行だ。」
魔理沙「`Signal`、コンポーネントの状態を定義しているぜ。」

```purs
squareArraySig /\ squareArrayChannel <- newState $ replicate 9 Nothing
gameStateSig /\ gameStateChannel <- newState $ NextPlayer X
```

魔理沙「`newState`というのが、Jelly の関数だ。厳密には違うが、イメージ的には、React Hooks の `useState` に似た働きをする。」
魔理沙「定義したい状態の初期値を引数にとるのは、useStateと同じだな。」
魔理沙「そして、`Signal` と `Channel` のタプルを返す関数だ。」
魔理沙「`Signal`は、状態。そして、 `Channel` を使って、状態の書き込みができるぜ。」
魔理沙「`replicate 9 Nothing` で、`Nothing` が9個の配列を作る。」

```purs
[Nothing, Nothing, Nothing, Nothing, Nothing, Nothing, Nothing, Nothing, Nothing]
```
### handleClick

魔理沙「`handleClick` 関数。これはclickしたときの処理だ。」
魔理沙「clickして、内部の状態を更新する副作用がある部分だぜ。」
魔理沙「これは型が難しい。」

```purs
handleClick :: MonadEffect m => Int -> m Unit
```

魔理沙「`MonadEffect`は、`m` がその中で副作用を起こせるモナドだということを表す型クラスだ。」
魔理沙「具体的には、Jelly の関数 `writeChannel` が状態を更新する。これが、副作用を起こせるモナドだ。」
魔理沙「つぎに、もとのJSのコードを見ると、`if (calculateWinner(squares) || squares[i]) {……`でアーリーリターンしていることがわかる。」
魔理沙「つまり、①勝者がすでにいる場合②clickした升目がclick済み（すでに値が入っている）場合は、clickしても、何も反応しなくするということだ。」
魔理沙「PureScriptには、アーリーリターンがないから、パターンマッチングと`when`を使って実装していくぜ。」
魔理沙「まず、」

```purs
case gameState of
  Winner _ -> pure unit
```

魔理沙「つまり、①勝者がすでにいる場合は、`pure unit`。」
魔理沙「何もしないということだな。」
魔理沙「つぎに、」

```purs
NextPlayer player -> when ((squares !! i) == Just Nothing) do
```

魔理沙「勝者がいない次のプレイヤーがいる場合`NextPlayer player`だ。」
魔理沙「`when`というのは、条件節で、`when…`がtrueのとき、実行するということだ。」
魔理沙「②clickした升目がclick済み（すでに値が入っている）の否定は、値が、`Just Nothing`のときだ。この時、値を更新する。」
魔理沙「Jelly の `writeChannel` で、状態を書き込めるぜ。」

```purs
writeChannel :: forall m a. MonadEffect m => Channel a -> a -> m Unit
```

魔理沙「`writeChannel`の引数は、`a`だ。」
魔理沙「だから、コードがややさかのぼるが、`readSignal`の型は以下だ。」

```purs
readSignal :: forall m a. MonadEffect m => Signal a -> m a
```

魔理沙「つまり、`readSignal` は`Signal a`から`Signal`をはがすんだぜ。」
魔理沙「`Signal`の取得の書き方のパターンが何パターンかあるが、ここは`Signal`取得のときに、`writeChannel`で書き込みために、`readSignal`で`Signal`をはがす必要があったんだ。」
霊夢「なるほどね。」
### renderSquareComponent

魔理沙「`renderSquareComponent`についてだ。」
魔理沙「`squareComponent { onClick: handleClick valueInt, value: valSig }`と9回かくのは、大変なので、もとのReactの例のように、`renderSquareComponent`に升のindexの数字を入れるだけにしたいんだぜ。」

```purs
renderSquareComponent :: Component m => Int -> m Unit
renderSquareComponent valueInt = do
  let
    valSig :: Signal (Maybe SquareValueType)
    valSig = do
      squares <- squareArraySig
      pure $ join $ squares !! valueInt
  squareComponent { onClick: handleClick valueInt, value: valSig }
```
魔理沙「`Signal`はモナドだから、do構文が使える。」
魔理沙「ここは、`squareComponent`の引数で、`Signal`をとるから、`readSignal` を使う必要はないぜ。」

## playStatus

```purs
playStatus :: Signal String
playStatus = gameStateSig <#> case _ of
  NextPlayer player -> "Next player: " <> show player
  Winner winner -> "Winner: " <> show winner
```

魔理沙「また、`hogeSig <#> case _ of`型のパターンマッチングが出てきたな。」
魔理沙「ここも`playStatus`自体が`Signal`だから、`readSignal` を使う必要はないぜ。」
魔理沙「形は違うが、一応」

```purs
do
  gameStatus <- gameStatusSig
```

魔理沙「と同じだぜ。」

## root コンポーネント

```purs:src/Main.purs
module Main where

import Prelude

import Data.Foldable (traverse_)
import Effect (Effect)
import Effect.Aff (launchAff_)
import Effect.Class (liftEffect)
import Jelly.Aff (awaitBody)
import Jelly.Component (class Component)
import Jelly.Element as JE
import Jelly.Hooks (runHooks_)
import Jelly.Hydrate (mount)
import Jelly.Prop ((:=))

import Components.Board (boardComponent)

main :: Effect Unit
main = launchAff_ do
  bodyMaybe <- awaitBody
  liftEffect $ traverse_ (runHooks_ <<< mount rootComponent) bodyMaybe

rootComponent :: forall m. Component m => m Unit
rootComponent =
  JE.div [ "class" := "game" ] do
    JE.div [ "class" := "game-board" ] do
      boardComponent

```

[^1]: 西郷甲矢人、能美十三『圏論の道案内 ~矢印でえがく数学の世界~』技術評論社 Kindle版 位置No.3983/4269
[^2]: https://github.com/purescript/purescript-prelude/blob/v3.0.0/src/Data/Functor.purs#L24
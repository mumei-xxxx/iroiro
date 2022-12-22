---
title: "魔理沙「PureScript Jelly で、Reactチュートリアル三目並べの《純粋関数型／型安全》版を作るぜ」②"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PureScript"]
published: false
---

> 数学は論理を探求するひとつの方法である。
>
> ウィトゲンシュタイン『論理哲学論考』6.234 [^1]

> この三昧に遊化（ゆけ）するに、端坐参禅を正門とせり。
> この法は、人人の分上にゆたかにそなはれりといへども、
> いまだ修せざるにはあらはれず、証せざるにはうることなし。[^2]
> 
> （この三昧に遊化（あそ）ぶには、端坐して参禅するをその正門としている。
> この法は、人々（めいめい）の身の上に何不足なくそなわっているのであるが、
> 修行しないと実現しないし、修行して実証しないと自分のものにならない。）
> 
> 道元禅師「辨道話」

魔理沙「PureScript Advent Calendar 2022 最終日の記事だ。」

https://qiita.com/advent-calendar/2022/purescript

## 完成したもの（再掲）

https://github.com/mumei-xxxx/purescript-sanmoku-dev0/tree/main/jelly-tic-tac-toe-without-timemachine

![tic-tac-toe](/images/purescript-jelly-tic-tac-toe-01/tic-tac-toe.gif)


## ディレクトリ構成（再掲）

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
  ( SquarePropsType
  , squareComponent
  )
  where

import Prelude
import Data.Maybe (Maybe(..))
import Web.HTML.Event.EventTypes (click)

import Jelly.Component (class Component, textSig)
import Jelly.Element as JE
import Jelly.Prop ((:=), on)
import Jelly.Signal (Signal)

import UseCases.Calculatewinner (SquareValueType(..))

{-
  squareComponent の引数の型
  onClick clickしたときに呼び出される関数
  升目の値 Signal かつ Maybe。X or O or Nothing
-}
type SquarePropsType m =
  { onClick :: m Unit
  , value :: Signal (Maybe SquareValueType) }

{-
  squareComponent 三目ならべの盤のひとつの升目のComponent
  SquarePropsType m を引数にとり、 m Unitを返す。
  Unit はHaskellの空のタプル()と同じ。
  HTMLを描画する。
  呼び出すときは、
  squareComponent { onClick: ●●, value: ■■ }
  のようにする。
-}
squareComponent :: forall m. Component m => SquarePropsType m -> m Unit
squareComponent { onClick, value } = do
  JE.button [ "class" := "square", on click \_ -> onClick ] do
    -- パターンマッチング
    -- textSig は Signal を表示する Jelly の関数
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
魔理沙「`<$>` の型は、`(a -> b) -> f a -> f b`[^3] 」
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
import UseCases.Calculatewinner (SquareValueType(..), calculateWinner)

nextPlayer :: SquareValueType -> SquareValueType
nextPlayer = case _ of
  X -> O
  O -> X

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
        NextPlayer player -> when ((squares !! i) == Just Nothing) do
          let
            newSquares = fromMaybe squares $ updateAt (i :: Int) (Just player) squares
          writeChannel squareArrayChannel newSquares
          writeChannel gameStateChannel $ case calculateWinner newSquares of
            Nothing -> NextPlayer $ nextPlayer player
            Just winner -> Winner winner

    renderSquareComponent :: Component m => Int -> m Unit
    renderSquareComponent valueInt = do
      let
        valSig :: Signal (Maybe SquareValueType)
        valSig = do
          squares <- squareArraySig
          pure $ join $ squares !! valueInt
      squareComponent { onClick: handleClick valueInt, value: valSig }

    playStatus :: Signal String
    playStatus = gameStateSig <#> case _ of
      NextPlayer player -> "Next player: " <> show player
      Winner winner -> "Winner: " <> show winner

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

魔理沙「最後は、root コンポーネントだ。」

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

魔理沙「`main`の中で、`rootComponent`をレンダーするイメージだな。」
魔理沙「`main`の内部については、難しくて、正直私にもよくわかってないぜ。」
魔理沙「現時点ではとりあえず、コンポーネントをレンダーするボイラープレートとしてとらえて欲しい。」
魔理沙「Reactで言えば、`ReactDOM.render`みたいな感じだ。」
魔理沙「ということで、一通りの解説が終了した。」

## 感想

霊夢「今回の感想はどうだった？」
魔理沙「まず、`calculateWinner`関数でも`Data.Array`のメソッドなどを見てきたが」
魔理沙「`Data.Array`のメソッドやモナドが強力と感じたぜ。」
魔理沙「`Data.Array`に配列操作のメソッドがかなり充実していて、便利だと思った。」
魔理沙「配列のモナドで、`foreach`的な処理も宣言的に書けた。」
魔理沙「パターンマッチングもかなり強力と思ったな。」
魔理沙「それと全体的に括弧を書くのが減らせて、そこがよかったな。」
霊夢「なるほどね。」
霊夢「PureScript Jellyについてはどうだい。」
魔理沙「Jelly.Signal は最初、ゆきくらげさんが、`purescript-signal` が純粋性が壊れているといって、FRP（Functional Reactive Programming）の考えも取り入れて、自作したものだが、なかなか、よく考えられていて、ひたすら感服したぜ。」
魔理沙「あと、全体的にシンプルでいい感じだ。」
魔理沙「Real DOMを使うのも、今風でそりがいい感じだ。」
魔理沙「この記事の内容は、Jelly が進化していくに連れ、どんどん古くなっていくだろうから、『2022年末はこうだった』という記録的なものとして見てほしいね。」
魔理沙「そんな感じだ。作っていて感じたのは、もっと複雑なものを作ってみないとわからない感じもしたね。」
霊夢「なるほどね。」
魔理沙「それでは、ここまで読んでくださった読者の皆様、そして、協力していだいたゆきくらげさんはじめ、ディスコード PureScript JPコミュニティの皆さんに改めて感謝をいいたい。」
魔理沙、霊夢「ありがとうございました。」
魔理沙「それでは、サラダバー」
霊夢「サラダバー」

[^1]: ウィトゲンシュタイン 著 野矢 茂樹訳『論理哲学論考』（岩波文庫）p.135 https://www.iwanami.co.jp/book/b246897.html
[^2]: 鏡島 元隆 監修、水野 弥穂子 訳註『原文対照現代語訳 道元禅師全集 正法眼蔵 1』（春秋社）p.3
https://www.shunjusha.co.jp/book/9784393150214.html
[^3]: https://github.com/purescript/purescript-prelude/blob/v3.0.0/src/Data/Functor.purs#L24
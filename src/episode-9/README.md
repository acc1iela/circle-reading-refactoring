# データの再編成

データ構造はプログラムに対して重要な役割を果たす。
この章ではそれらに焦点を当てたリファクタリング群の紹介がされています。

**変数の分離**
一つの値を異なる目的で利用している時、その値を分離することでコードを読みやすくすることができます。

**変数名の変更**
読んで字の如く、変数に適切名前をつけるのは難しいがとても重要なこと。

**問い合わせによる導出変数の置き換え**
時には変数を完全に取り除くことが最善なケースがある、その場合は導出変数の置き換えが有効。

**参照から値への変更または値から参照への変更**
参照オブジェクトと値オブジェクトの混乱による問題の解消

---

## 変数の分離

```js
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

⬇︎

```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

リファクタリング後のコードは変数名が明確化されており`const`宣言が使用されているのでこの`area`が再代入されないということが保証されている。(good)

**変数の分離をするべき時はいつ？？**

変数の分離をするべき時は、変数が複数の役割を持ってしまっている時。
基本的にループ用変数を除き、多くの変数は一つの役割を持つべき。
上記の例で出しているように一つの変数を二つのことに使用すると読み手が混乱してしまう原因になる。

例）

```js
function distanceTravelled(scenario, time) {
  let result;
  let acc = scenario.primaryForce / scenario.mass;
  let primaryTime = Math.min(time, scenario.delay);
  result = 0.5 * acc * primaryTime * primaryTime;
  let secondaryTime = time - scenario.delay;
  if (secondaryTime > 0) {
    let primaryVelocity = acc * scenario.delay;
    acc = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
    result += primaryVelocity * secondaryTime + 0.5 * acc * secondaryTime * secondaryTime;
  }
  return result;
}
```

読みづらい、、色々問題箇所はあるもののまずは`acc`という変数が二つの役割を持っていることが問題。

1. 最初の力で加えられた初期化速度を保持する
2. 最初と 2 回目の両方の力による加速度を保持する

```js
function distanceTravelled(scenario, time) {
  let result;
  const primaryAcceleration = scenario.primaryForce / scenario.mass;
  let primaryTime = Math.min(time, scenario.delay);
  result = 0.5 * primaryAcceleration * primaryTime * primaryTime;
  let secondaryTime = time - scenario.delay;
  if (secondaryTime > 0) {
    let primaryVelocity = primaryAcceleration * scenario.delay;
    let secondaryAcceleration = (scenario.primaryForce + scenario.secondaryForce) / scenario.mass;
    result +=
      primaryVelocity * secondaryTime + 0.5 * secondaryAcceleration * secondaryTime * secondaryTime;
  }
  return result;
}
```

`acc`を`primaryAcceleration`に変更することで、`acc`がどのような役割を持っているのかが明確になった。
そしてこの変数は一度しか代入されないということを保証するために`const`宣言を使用している。

`acc`も正直変数名としては分かりづらいので`primaryAcceleration`に変更をしている。

**例）入力パラメータへの代入**

変数を分離すべき別のケースとして、入力パラメータに代入している場合がある。

```js
function discount(inputValue, quantity) {
  if (inputValue > 50) inputValue = inputValue - 2;
  if (quantity > 100) inputValue = inputValue - 1;
  return inputValue;
}
```

上記の例では　`inputValue`という変数が関数への入力パラメータと、呼び出し側に戻す結果の保持という二つの役割を持っている。
この場合次のように変数を分離できる。

```js
function discount(originalInputValue, quantity) {
  let inputValue = originalInputValue;
  if (inputValue > 50) inputValue = inputValue - 2;
  if (quantity > 100) inputValue = inputValue - 1;
  return inputValue;
}
```

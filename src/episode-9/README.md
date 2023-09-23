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

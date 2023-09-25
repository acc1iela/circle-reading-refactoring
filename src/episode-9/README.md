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

`acc`も正直変数名としては分かりづらいので`secondaryAcceleration`に変更をしている。

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

より適切な名前にするために変数名の変更を 2 回行う。

```js
function discount(inputValue, quantity) {
  let result = inputValue;
  if (inputValue > 50) result = inputValue - 2;
  if (quantity > 100) result = inputValue - 1;
  return result;
}
```

---

## フィールド名の変更

レコード構造のフィールド名がプログラム全体で広く使われている場合は特に重要。

> フローチャートを見せてくれても、テーブルを隠されたら煙に巻かれたままだろう。テーブルを見せてくれれば、通常フローチャートはいらない。それだけですぐわかる。 - Philips Brooks

データ構造は非常に重要 → 理解しやすくするためには適切な名前をつけることが重要。

**例）フィールド名の変更**
`const organization = { name: 'Acme Gooseberries', country: 'GB' };`
// これの name を title に変更したい

```js
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name() {
    return this._name;
  }
  set name(aString) {
    this._name = aString;
  }
  get country() {
    return this._country;
  }
  set country(aCountryCode) {
    this._country = aCountryCode;
  }
}

// nameをtitleに変更したい
// 以前出てきたレコードのカプセル化を使う
const organization = new Organization({ name: 'Acme Gooseberries', country: 'GB' });
```

上記でレコード構造をクラスにカプセル化したため、名前を変更するのに必要な場所は getter, setter, コンストラクタ、内部データ構造の 4 つになる。

```js
class Organization {
  constructor(data) {
    this._title = data.name;
    this._country = data.country;
  }
  get name() {
    return this._title;
  }
  set name(aString) {
    this._title = aString;
  }
  get country() {
    return this._country;
  }
  set country(aCountryCode) {
    this._country = aCountryCode;
  }
}

// nameをtitleに変更したい
const organization = new Organization({ title: 'Acme Gooseberries', country: 'GB' });
```

1. `name`の値をクラスの private フィールドに変更（`_title`）
2. `get name()` `name` の値を`_title`から返すように変更
3. `set name()` `name` の値を`_title`に代入するように変更

フィールド名として`_title`を導入したものの外部からアクセスメソッドの名前は`name`のままになっている。
これにより外部からは`name`としてアクセスできるものの内部的には`_title`としてのデータが保持されるようになった。

この変更で外部のコードに影響を与えずに内部のフィールド名が変更出来ている。
外部からは`organization.name`としてアクセスできるものの内部的には`organization._title`としてアクセス出来るようになった。

```js
  constructor(data) {
    this._title = data.title !== undefined ? data.title : data.name;
    this._country = data.country;
  }

  // コンストラクタの呼び出し側を全て調べて新しい名前を使用するように変更できる
  const organization = new Organization({ title: 'Acme Gooseberries', country: 'GB' });
```

これによりコンストラクタ側の呼び出しは name, title どちらでも呼び出せるようになった（title が優先される）
ここまで済んだら`name`のフィールド名を`title`に変更することができる。

```js
class Organization {
  constructor(data) {
    this._title = data.title;
    this._country = data.country;
  }
  get title() {
    return this._title;
  }
  set title(aString) {
    this._title = aString;
  }
  get country() {
    return this._country;
  }
  set country(aCountryCode) {
    this._country = aCountryCode;
  }
}
```

---

## 問い合わせによる導出変数の置き換え

**このリファクタリングを行う動機**

- ソフトウェアにおける問題の発生源として大きな割合を占めるのは変更可能なデータ
- データの変更を通じてコードの別の箇所が厄介な形で結合されるのはありがち
  - 1 箇所への変更が見つかりにくい間接的な影響を及ぼすことがある
- 変更可能なデータの影響範囲はできる限り小さくすることが重要

例）

```js
get discountedTotal() {
  return this._discountedTotal;
}

set discount(aNumber) {
  const old = this._discount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber;
}
```

⬇︎

```js
get discountedTotal() {
  return this._baseTotal - this._discount;
}

set discount(aNumber) {
  this._discount = aNumber;
}
```

1. `_discountedTotal`の計算ロジックがゲッターに移動された。これにより`_discountedTotal`の値を手動で更新する必要がなくなった。
2. `discount`セッターは新しい割引額を設定するだけになった。古い割引額との差を計算して`_discountedTotal`の更新を手動で行う必要がなくなった。

---

## 参照から値への変更

```js
class Product {
  applyDiscount(arg) {
    this._price.amount -= arg;
  }
}
```

`applyDiscount`メソッドは`_price`オブジェクトの`amount`プロパティを直接変更している
これはオブジェクトの内部状態を直接変更してしまっていて不変性が保たれない可能性がある

⬇︎

```js
class Product {
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }
}
```

変更後のコードは新しい`Money`オブジェクトを作成して、そのオブジェクトの`amount`プロパティに割引後の価格を設定している
それから`_price` プロパティに新しい`Money`オブジェクトを代入している、これにより`_price`オブジェクトは変更されることなく新しいオブジェクトが作成されるため不変性が保たれる。

- オブジェクトやデータ構造を入れ子にする時内部オブジェクトは参照か値として扱うことが可能
- 両者の明確な違い
  - 内部オブジェクトのプロパティ更新をどうやって処理するか
    - 参照として扱う場合 → 内部オブジェクトのプロパティを更新して同じ内部オブジェクトを保持する
    - 値として扱う場合は期待するプロパティを持つ新しい内部オブジェクトにまるごと置き換える

別例）

```js
class Address {
  constructor(city, street) {
    this.city = city;
    this.street = street;
  }
}

class User {
  constructor(address) {
    this.address = address;
  }

  updateAddress(city, street) {
    this.address.city = city;
    this.address.street = street;
  }
}
```

リファクタリング前の`updateAddress`メソッドは`address`オブジェクトのプロパティを直接変更している。☠️

**何が問題？**
外部からこのオブジェクトを参照している場合、予期しない変更が発生する可能性がある。

⬇︎

```js
class Address {
  constructor(city, street) {
    this.city = city;
    this.street = street;
  }
}

class User {
  constructor(address) {
    this.address = address;
  }

  updateAddress(city, street) {
    // ここで新しいオブジェクトを作成している
    this.address = new Address(city, street);
  }
}
```

リファクタリング後の実装では新しい`Address`オブジェクトを作成してそれを`address`プロパティに代入している。
これにより元の`address`オブジェクトは変更されず新しいオブジェクトが作成されるためオブジェクトの普遍性が保たれる。🌈

---

## 値から参照への変更

同じデータに対して複数のコピーを行うと、値が更新された際に問題となる。
この場合は、値そのものではなく、参照に変更する必要がある。

例）

```js
class Color {
  constructor(r, g, b) {
    this.r = r;
    this.g = g;
    this.b = b;
  }
}

// 同じ色を表す複数のインスタンス
let red1 = new Color(255, 0, 0);
let red2 = new Color(255, 0, 0);
let red3 = new Color(255, 0, 0);

console.log(red1 === red2); // false
console.log(red2 === red3); // false
```

上記の例では同じ色を表す複数のインスタンスが作成されているが、それぞれのインスタンスは別のオブジェクトを参照しているため、それぞれのインスタンスは等しくない。

⬇︎

```js
class Color {
  constructor(r, g, b) {
    this.r = r;
    this.g = g;
    this.b = b;
  }
}

class ColorRepository {
  constructor() {
    this.colors = {};
  }

  get(r, g, b) {
    const key = `${r}-${g}-${b}`;
    if (!this.colors[key]) {
      this.colors[key] = new Color(r, g, b);
    }
    return this.colors[key];
  }
}

const colorRepository = new ColorRepository();

// リポジトリを使用して同じ色を参照
let red1 = colorRepository.get(255, 0, 0);
let red2 = colorRepository.get(255, 0, 0);
let red3 = colorRepository.get(255, 0, 0);

console.log(red1 === red2); // true
console.log(red2 === red3); // true
```

上記の例では`ColorRepository`クラスを作成して色のオブジェクトを管理している。
同じ色を表す`Color`オブジェクトはリポジトリを通じて取得されているため、同じオブジェクトを参照している。

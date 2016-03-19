# 関数型プログラミングとは
- **純粋関数**のみでプログラミングを構築することが前提となる
- 純粋関数にはモジュール性があるため、テスト・再利用・並列化・一般化・推論が容易になる
- 要するにバグが発生しにくくなる効果もある
- 需要な概念、**参照透過性**と**置換モデル**

## 純粋関数とは
- **副作用**のない関数
- 副作用がある関数とは以下のように単に結果を返す以外に何か操作および、処理を行う関数
  1. 変数を変更する
  - データ構造を直接変更する
  - オブジェクトのフィールドを設定する
  - 例外をスローする、またはエラーで停止する
  - コンソールに出力する、またはユーザ入力を読み取る
  - ファイルを読み取る、またはファイルに書き込む
  - 画面上に描画する

### コードサンプル
```scala
// 副作用を持つコード
class cafe {
  def buyCoffee(cc: CreditCard): Coffee = {
    val cup = new Coffee()
    cc.charge(cup.price) // -->> クレジットカードにコーヒー代をチャージする
    cup
  }
}
```

- 上記のようなクレジットカードにチャージをするという行為は外部サービスとの連携が必要となり副作用を持つ関数となり、テストもやりにくい
- 複数注文する場合はbuyCoffeeを複数回呼び出すことになり、チャージもその都度行うことになるためモジュール性に欠ける

```scala
// 副作用を排除したコード
class cafe {
  def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
    val cup = new Coffee()
    (cup, Charge(cc, cup.price))
  }
}

case class Chage(cc: CreditCard, amount: Double) {
  def combine(other: Charge): Charge =
    if (cc == other.cc)
      Charge(cc, amount + other.amount)
    else
      throw new Exception("Can't combine charges to different cards")
}
```

## 参照透過性
- プログラムの意味を変えることなく、式をその結果に置き換えることができる
- 式eがあり、すべてのプログラムpにおいて、pの意味に影響を与えることなく、p内のすべてのeをeの評価結果と置き換えることができるとしたら、eは参照透過です。関数fがあり、式f(x)が参照透過なすべてのxに対して、参照透過であるとしたら、fは純粋関数である

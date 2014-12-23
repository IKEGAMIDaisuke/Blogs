# はじめに

この文章は
[Theorem Prover Advent Calendar 2014](http://qiita.com/advent-calendar/2014/theorem_prover)
の 12/24(水) の記事のために書かれました（たぶんね）。

## KeY-project とは

KeY-project とはプログラム、特に Java (の subset) プログラムに対する形式手法のための
ツール群です。以下のツールが開発されているようです：

* KeY Prover (for Java Card)
* KeY-Hoare
* KeYmaera
* KeY for C
* ASMKey

本稿では Key Prover の簡単な紹介を行います。
その他のツールについては、筆者の能力の限界＆アドベントカレンダーの締切のせいもあり、
紹介することができません。

概要を掴むには、
[本家のページ](http://www.key-project.org/)
や
[Wikipedia(en)](http://en.wikipedia.org/wiki/KeY)
などを参照したほうがより正確だと思いますが、
ここ数日で簡単なプログラムの検証（？）をやってみたのでその経験も話します。

# KeY Prover の紹介

## 概要
## Java Card
## JML
## 具体例

本稿では KeY Prover を初めて利用する読者のために、シンプルな具体例を提供します：

1. Add
2. ElemIndex
3. Max

全てのファイルはこのリポジトリの src/ 以下にあります。

### Add

最初にシンプルな JML 記述によるメソッドの振る舞い検証を見ます。
次のプログラムに対する検証を行います。

```Java
public class Add {
  
  public static int add(int x, int y) {
    return x + y;
  }
  
}
```
注：このようなプログラム add を（クラス/メソッド）にすることは、
もちろん実際には推奨されません。しかし、説明のために用意しました。

足し算の可換則と結合則を検証します：

1. x + y == y + x （可換則）
2. (x + y) + z == x + (y + z) （結合則）

事後条件では特別な記述 \\result が使えます。
メソッド add に対する \\result は x + y です。
というのは、メソッド add にreturn x + y; という文があるからです。

そこで、可換則の記述は次のようになります：

```Java
/*@ ensures y + x == \result; @*/
```

一方、事後条件にメソッドの呼び出しも使えます。
そこで、結合則を書くときには \\result の代わりにメソッド呼び出しを使ってみましょう：

```Java
/*@ ensures (\forall int z; add(x, y) + z == x + add(y, z)); @*/
```

このように、一般に調べたい事後条件を JML で書く方法は一通りではありません。
その代わりに、

```Java
/*@ ensures add(add(x, y), z) == (blah blah blah); @*/
```

と書くこともできます。

さて、事後条件はまとめて書くことができます。KeY Prover への入力は最終的に次のようになります：

```Java
public class Add {
  
  /*@ ensures y + x == \result; // commutative
    @ ensures (\forall int z; add(x, y) + z == x + add(y, z)); // associative
    @*/
  public static int add(int x, int y) {
    return x + y;
  }
  
}
```

では、実際に KeY Prover での検証過程を見ていきましょう。

#### KeY Prover を起動

KeY Prover を起動すると、次のようなウィンドウが現れます：

![img/Add/start.png](/20141224-IntroductionToKeYProver/img/Add/start.png?raw=true)

この小さいウィンドウをリサイズして大きくしておきます。

![img/Add/resized.png](/20141224-IntroductionToKeYProver/img/Add/resized.png?raw=true)

#### ファイルをロード

![img/Add/prepare.png](/20141224-IntroductionToKeYProver/img/Add/prepare.png?raw=true)

左の Contract Targets に事後条件を記述したメソッドが現れます。
今回は add だけですが、複数のメソッドの証明をするときには、個々を選択する必要があります。
選択を忘れると、先頭のメソッドの検証だけをするので注意が必要です。

右には選択されたメソッドに対する Contract が現れます。

Start Proof のボタンを押すと、いよいよ自動証明を行う前のウィンドウが現れます。　

#### 自動証明を始める

![img/Add/go.png](/20141224-IntroductionToKeYProver/img/Add/go.png?raw=true)

左上のいかにも処理を始めそうなボタンを押せば、自動証明が始まります。
その右横に用いる SMT ソルバの選択ボタンがあります。ここで取り替えができます。
左には 0 OPEN GOAL があります。
右には Current Goal があります。
Java では実行時例外が発生する可能性があるため、
検証式には例外に関するいろいろな条件がつきますが、
今回の検証では次の式が本質です：

    (blah blah blah)
    -> javaAddInt(y, x) = result
	   & \forall int z;
	       (   inInt(z)
		    -> javaAddInt(add(x, y), z) = javaAddInt(x, add(y, z))))
       & (blah blah blah)

この式は KeY Prover が JML アノテーションと Java Card プログラムから自動的に変換したもので、
手入力する必要はありません。

Proof Search Strategy ボタンを押すと、SMT ソルバに与えるメモリの上限などを変更できるのですが、
今回の場合はデフォルトで証明が終わるので、変更は必要ありません。

#### 結果

証明が成功すると、次の二つのウィンドウが現れます：

![img/Add/proved.png](/20141224-IntroductionToKeYProver/img/Add/proved.png?raw=true)

![img/Add/result.png](/20141224-IntroductionToKeYProver/img/Add/result.png?raw=true)

最初のウィンドウには、実行時間の統計情報などが現れます。
結果のウィンドウには、
左に自動証明で用いた推論規則の一覧が、
右に推論が進んだ式が現れます。

各推論規則はクリックできて、推論がどのように進むかを確かめることができます。
証明に成功したときは、推論規則の最後に緑色の Closed goal が現れます。
今回は成功しましたが、失敗するときはいくつかの Open goal(s) が現れます。

add は副作用を持ちません。このときは、`/*@ pure @*/` を明示的につけます：

```Java
public static /*@ pure @*/ int add(int x, int y)
```

`/*@ pure @*/` はプログラム中にループがあるときの検証を楽にするものです。
次節で紹介するループ不変量の記述にできることが増えます。
今回の検証にはループを持つプログラムが全体にないので必要ありませんが、
覚えておくとよいです。

今回の事例では推論は SMT ソルバを使わなくても sequent calculus でできそうです。
実際 Proof Statistics を注意深く見ると SMT solver apps: 0 と書いてあります。
本節は、事後条件の書き方を紹介することが目的でした。

本節以降は、ウィンドウなどの画面を引用しません（どれも似たようなものなので）。

### ElemIndex

KeY の配布物には examples が含まれています。
その中(04-LinkedList)に linked list の search の検証があります。
ここでいう search は、リスト中に与えられた要素が存在するかどうかを調べるプログラムです。

本節では linked list の代わりに、よりシンプルに、 Java Card の配列に対して同じことを調べます。
また、search という名前は曖昧だと感じたので Haskell に倣って elemIndex と命名しました。
elemIndex は、配列の中に与えられた要素が存在しなければ -1 を、
存在するとき、そのなかで最小の index を返します。

検証対象のプログラムは以下の通りです：

```Java
public class ElemIndex {
  
  public static int elemIndex(int x, int[] a) {
    int i = 0, r = -1;

    while (r == -1 && i < a.length) {
      if (a[i] == x) r = i; else i++;
    }
    return r;
  }
  
}
```

このプログラムを検証するときは、事前条件と事後条件の他に、
ループ不変量とループ変量の二つを書く必要があります。
ループ不変量はループに入る前とループに入った後の条件を検証するために必要です。
ループ変量はループが停止することを保証するために必要です。

ともあれ、事前条件と事後条件を書いてみましょう。

```Java
  /*@ requires a != null;
    @ ensures
    @   (\result == -1 &&
    @     (\forall int j; 0 <= j && j < a.length; a[j] != x)) ||
    @     (0 <= \result && \result < a.length && a[\result] == x &&
    @        (\forall int j; 0 <= j && j < \result; a[j] != x));
    @*/
  public static /*@ pure @*/ int elemIndex(int x, int[] a) {
    (blah blah blah)
```

前節 add の諸性質に比べるとずいぶん長くなりましたが、個々に見ていけば複雑ではありません。

まず、事前条件として `a! = null` が必要です。
プログラム elemIndex に `a == null` の場合を追加すれば、条件は変わりますが、
ここでは説明を簡単にするために、それを考えません。

`int x` は Java では primitive type なので `null` になることはありません。

次に事後条件を書きます。
配列 `a` の中に `x` が存在しないとき、または `x` が存在しないときで場合分けをします：

```Java
/*@ ensures
    (x が存在しないとき) || (x が存在するとき) // 疑似記述
  @*/
```
`x` が存在しないときは、

* `\result == -1`
* `(\forall int j; 0 <= j && j < a.length; a[j] != x)`

の両方をみたします。これらを `&&` でつなぎます。

一方、`x` が存在するときは、

* `(0 <= \result && \result < a.length)`
* `a[\result] == x`
* `(\forall int j; 0 <= j < \result; a[j] != x)`

の全てが成り立ちます。最後の式は index の最小性を意味します。

以上のことから、事後条件は次のようになります：

```Java
  /*@ requires a != null;
    @ ensures
    @   (\result == -1 &&
    @     (\forall int j; 0 <= j && j < a.length; a[j] != x)) ||
    @     (0 <= \result && \result < a.length && a[\result] == x &&
    @        (\forall int j; 0 <= j && j < \result; a[j] != x));
    @*/
  public static /*@ pure @*/ int elemIndex(int x, int[] a)
    (blah blah blah)
```

次にループ不変量を書きます。ループ不変量とはループ中に変わらない条件のことで、
かつ、事前条件と事後条件を導出するために必要な条件でなければ検証ができません。
導出のためにどのようなループ不変量を書いたらいいのかは、自明ではないのですが、
まあともあれ、やってみましょう。今回は試行錯誤せずに正答を与えます。

ループ中は次のことが成り立っています：

* `a != null`
* `0 <= i <= a.length`

ここで、 `i < a.length` ではなく `i <= a.length` であることに注意してください。
while の終了条件で `i < a.length` ですが、ループ中では `i` は `a.length` になり得ます。
実際、`i < a.length` のまま Key Prover に渡すと証明ができずに open goal が残ります。

事前条件と事後条件を導出するためのループ不変量はこれだけでは足りません。
次の条件のどちらかも必要です：

1. `r == -1`
2. `r == i && i < a.length && a[r] == x`

配列 `a` の中に `x` が存在する場合と、存在しない場合です。

最後に、ループ中に代入しうるローカル変数を明示します。
今回は `r` と `i` です。このとき、`/*@ assignable r, i; @*/` と書きます。

最終的に、（導出を成功させる）ループ不変量の記述は次のようになります：

```Java
    /*@ loop_invariant
      @   a != null && 0 <= i && i <= a.length &&
      @   (\forall int j; 0 <= j && j < i; a[j] != x) &&
      @   (r == -1 || (r == i && i < a.length && a[r] == x));
      @ assignable r, i;
      @ (blah blah blah)
	  @*/
```

次に、ループ変量を書きます。ループ変量はループ中に減る値で、
かつループが終了することを導出するものでなければなりません。
ループ不変量と同様に、検証に成功するためのループ変量の記述は自明ではありませんが、
ここでは正答を示します。

ループ中では `r == -1` を満たす限り `i` がどんどん増えていきます。
なので、減る値は `a.length - i` です。
一方、`r != -1` のときは減る値はありません。
以上のことからループ変量は `/*@ decreases r == -1 ? a.length - i : 0` @*/ になります。

これで検証に必要な JML アノテーションの記述は終わりです。
最終的なプログラムは以下の通りです：

```Java
public class ElemIndex {

  /*@ requires a != null;
    @ ensures
    @   (\result == -1 &&
    @     (\forall int j; 0 <= j && j < a.length; a[j] != x)) ||
    @     (0 <= \result && \result < a.length && a[\result] == x &&
    @        (\forall int j; 0 <= j && j < \result; a[j] != x));
    @*/
  public static /*@ pure @*/ int elemIndex(int x, int[] a) {
    int i = 0, r = -1;

    /*@ loop_invariant
      @   a != null && 0 <= i && i <= a.length &&
      @   (\forall int j; 0 <= j && j < i; a[j] != x) &&
      @   (r == -1 || (r == i && i < a.length && a[r] == x));
      @ assignable r, i;
      @ decreases r == -1 ? a.length - i : 0;
      @*/
    while (r == -1 && i < a.length) {
      if (a[i] == x) r = i; else i++;
    }
    return r;
  }

}
```

今回の検証も SMT ソルバを用いるまでもなく Java DL と sequent calculus のみで、
導出が終わります。
本節では、Java DL は配列の検証を行うことが可能であることを紹介しました。
筆者の知る限り、Hoare logic ではそこまでの能力はありません。

### Max

# KeY Prover を支える技術
## Hoare logic & first order dynamic logic
### Java Card DL
## sequent calculus
## SMT ソルバ
## Taclets

# おわりに

筆者の Twitter account は @ikegami__ です。
体調が良いときしか見ませんが、簡単なコメントはこちらにくださっても結構です。

# 謝辞

鹿島先生 && Chalmers Key internal presentation && 故システム検証研究センター

# 参考文献
Using a fixed seed with SecureRandom (SecureRandom)
===================================================

警告されている問題点
--------------------

疑似乱数生成器(SecureRandom)のシードとして第三者による推測が容易な値を設定しているため、生成する擬似乱数も同様に推測される可能性があります[1](#1)。推測が容易な擬似乱数を暗号鍵や認証関連のトークン等の秘密情報の生成に使っている場合は、情報の漏洩、ディジタル署名の偽造、なりすましが発生します。

対策のポイント
--------------

推測が困難な疑似乱数を生成するためには、推測が困難なシードを疑似乱数生成器(SecureRandom)に与えます。

対策の具体例
------------

以下のいずれかの対策を実施してください。

-   (対策1)
    SecureRandomに明示的なシードを与えない(デフォルトの動作に任せる)
-   (対策2) SecureRandomに推測が困難なシードを設定する

### (対策1) SecureRandomに明示的なシードを与えない(デフォルトの動作に任せる)

特別な理由がない限り、SecureRandomにシードを与える必要はありません。デフォルトで内部的に推測困難なシードが設定されるためです。以下に実装例を示します。

```java
    // 引数のないコンストラクタを使用する
    Random r = new SecureRandom();
    ...
    // r.setSeed()関数の呼び出しは避ける
    byte[] buffer = new byte[16];
    r.nextBytes(buffer);
```

### (対策2) SecureRandomに推測が困難なシードを設定する

アプリの都合上、シードを明示的に設定する必要がある場合は、推測が困難なシードを使用する必要があります。推測が困難なシードを生成する方法としては、generateSeed(もしくはgetSeed)メソッドを使うことができます。ここで取得するシードの長さは、RTCの標準、ガイドラインや設計方針に従いましょう。
以下に実装例を示します。

```java
    Random r = new SecureRandom();
    ...
    final byte[] seed = SecureRandom.generateSeed(32); // 256ビットのシードを取得
    r.setSeed(seed);
```

誤った対応
----------

以下にLintによって検出されない誤った対応例を示します。

### SecureRandomのコンストラクタに定数や時刻を渡す

下の例のように、コンストラクタにバイト列を渡してシードを設定すると、Lintには検出されませんが、生成する疑似乱数が推測容易になるため、用途によっては大事な情報の漏えいに・改ざんに繋がります。

-   例1
```java
        Random r = new SecureRandom(new byte[] { 1, 2, 3} );
```

-   例2
```java
        Random r = new SecureRandom(System.currentTimeMillis());
```

### 引数がリテラルでない場合

setSeed()の引数が次の例ようにリテラルでない場合にはLintの検出から漏れてしまうことに注意が必要です。ただし、「Lintの検出例」に上げた「現在時刻をシードに与えている」の場合のみ例外的に検出されます。クラスのstatic finalで修飾された定数扱いのフィールドについても同様です。

-   例1
```java
        Random r = new SecureRandom();
        final byte[] seed = new byte[] { 1, 2, 3 };
        r.setSeed(seed);
```

-   例2
```java
        Random r = new SecureRandom();
        final long nanoTime = System.nanoTime();
        r.setSeed(nanoTime);
```

-   例3
```java
        Random r = new SecureRandom();
        r.setSeed("constant seed".getBytes());
```

Lintの検出例
------------

### 定数値のシードを与えている

定数値のシードを与えると、常に同じ擬似乱数列が生成されることになり、次に生成される擬似乱数が予測されてしまうおそれがあります。

```java
    Random r = new SecureRandom();

    // 定数のシードを与えている
    r.setSeed(new byte[] {
        0x04, 0xFD, 0xA8, 0x36, 0x9A, 0x75, 0x0A, 0xEC,
        0x77, 0x1B, 0x8A, 0x64, 0x3D, 0x19, 0xDE, 0xAD,
        0xA2, 0xF9, 0x66, 0x64, 0x0A, 0x3B, 0xBE, 0xEF,
        0x6E, 0x69, 0x42, 0x33, 0x0A, 0x91, 0xF7, 0x57
    });
```

-   Lint 結果（Warning）

<!-- -->

    Do not call `setSeed()` on a `SecureRandom` with a fixed seed: it is not secure. Use `getSeed()`.

### 現在時刻をシードに与えている

System.currentTimeMillis()メソッドやSystem.nanoTime()メソッドの返す値は、高々数秒程度の誤差しか伴わずに予測されると考えるべきです。そのため、攻撃者は比較的少数のシードをしらみつぶしに使うことによりSecureRandomのシードを推定することができ、したがって生成する擬似乱数を完全に予測されてしまうおそれがあります。

```java
    Random r = new SecureRandom();

    // 現在時刻 (ミリ秒単位) をシードに与えている
    r.setSeed(System.currentTimeMillis());
```

-   Lint 結果（Warning）

<!-- -->

    It is dangerous to seed `SecureRandom` with the current time because that value is more predictable to an attacker than the default seed.

注釈
----

###### 1
 Androidにおいて[2](#2)SecureRandomのnextBytes()メソッドで得られるバイト列はランダムに決定されるわけではなく、シードと呼ばれる初期値と、それまでのnextBytes()メソッド呼び出し回数によって一意に決定されます。このメソッドを次々に呼び出して得られるビット列を並べても一見何の規則性もないように見える、すなわちランダムに値を生成しているように見える[3](#3) ため、擬似乱数生成器と呼ばれています。得られるビットのシーケンスは、同じシードを与えれば常に同じであることに注意が必要です。

このルールで指摘していることは、擬似乱数生成器が生み出すビット列はシードにのみ依存するため、シードが推測容易だったり固定されていたりすると生成する値の予測を容易としてしまい、上記のような被害が発生しうるということです。

###### 2
「Androidにおいて」と断ったのは、[SecureRandomのAPIドキュメント](https://developer.android.com/reference/java/security/SecureRandom.html)にも記述されているとおり、実装によっては擬似乱数生成器ではなく乱数生成器であることもありうるからです。擬似乱数生成器とは異なり、乱数生成器は、ランダムさを伴うと考えられる観測値から偏りや自己相関がなくなるよう加工したバイト(ビット)列を出力します。詳しくは[RFC 4086](https://tools.ietf.org/html/rfc4086) を参照してください。

###### 3
「ランダムに値を生成しているように見える」ことは、一般に統計的仮説検定の枠組みで評価されます。「全くランダムにビット列を生成していること」を帰無仮説、「ランダムに生成していないこと」を対立仮説として、標本分布が正規分布やχ二乗分布に漸近的に従う複数の統計量について(多くの場合は有意水準1%として) 仮説検定します。詳細は [NISTのSP800-22Rev.1a](https://csrc.nist.gov/publications/detail/sp/800-22/rev-1a/final)を参照してください。

外部リンク
----------

-   SecureRandom | Android Developers
    -   https://developer.android.com/reference/java/security/SecureRandom.html
-   関連するcoding practiceとして、以下のCERTの推奨事項があります。
    -   [MSC63-J. Ensure that SecureRandom is properly
        seeded](https://www.securecoding.cert.org/confluence/display/java/MSC63-J.+Ensure+that+SecureRandom+is+properly+seeded)
-   やや発展的なトピックですが、シードとして使えるランダムなノイズを得る方法やその加工方法、その他暗号用途の擬似乱数生成器についてはRFC
    4086が参考になります。
    -   [RFC 4086: Randomness Requirements for
        Security](https://tools.ietf.org/html/rfc4086)
-   こちらも発展的なトピックですが、ランダムに生成された値らしさを評価する方法
    (統計的仮説検定) については、NISTのSP800-22 Rev.
    1aが参考になります。
    -   [NIST SP 800-22 Rev.
        1a](https://csrc.nist.gov/publications/detail/sp/800-22/rev-1a/final)

------------------------------------------------------------------------

補足
----

### AndroidアプリではSecureRandomは明示的にシードを設定する必要がない

SecureRandomはAndroid
6.0以降ではnextBytes()メソッドを呼び出すたびにgetrandomシステムコールを呼び出すか、あるいは/dev/urandomを必要なだけ読みだすのがデフォルトの挙動となっているため、シードを設定する必要がありません。Android 6.0以降ではsetSeed()の呼び出しも実質的にno-opsになっており、呼び出しても呼び出さなくても生成される擬似乱数列に違いはありません。

また、Android 5.1以前では初回のnextBytes()メソッドの呼び出し時に、/dev/urandomから256ビット読み出し、それを元にシードを生成するのがデフォルトの挙動になっています。setSeed()メソッドによって再シードしない限りは、初回呼び出し時に/dev/urandomから得たエントロピーの他に不確実性は追加されないため、再シードなしに長期的に擬似乱数を生成するとそれだけ予測が容易になっていくと考えられます。ただし、典型的なAndroidアプリの用途やライフサイクルを考えるならば、再シードが必要なケースはないと考えられます。

いずれのケースでも明示的にシードを設定する必要はなく、デフォルトの挙動で目的に適うはずです。

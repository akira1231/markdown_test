# Insecure HostnameVerifier (AllowAllHostnameVerifier)

## 警告されている問題点

SSL/TLS証明書のホスト名検証を正しく行わないため、HTTPS通信などに対する中間者攻撃が可能になり、通信内容が漏洩したり、改ざんされたりするリスクがあります。

## 対策のポイント

原則として、デフォルト設定のままサーバーとのSSL/TLS接続を行い、サーバー証明書のホスト名検証をバイパスしないようにしてください。

## 対策の具体例

HTTPS通信を含めSSL/TLS通信では、サーバーとの暗号化通信を安全に行うために、サーバーの証明書検証およびホスト名検証を決められた手順で行う必要があります。Androidが提供する通信API(クラス)はデフォルトで手順通りの検証を行っており、特別な理由がない限りは設定や実装を変更せずに使用してください。

```java
    void onConnectStart() {
        try {
            URL url = new URL("https://www.example.com");
            HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();            connection.connect();
        } catch (IOException e) {
        }
    }
```

デバッグや開発中にホスト名検証をバイパスし、そのまま市場に出てしまう事例がしばしば報告されています。こういった事故を防ぐために、Android 7.0(API 24)以降では「Network Security Configuration」を利用することができます。

以下に用途に応じていくつか簡単な例を示します。
> 詳しくは[「Network Security Configuration | Android Developers」][2]や[「Androidアプリのセキュア設計・セキュアコーディングガイド」][3]などを参照してください。

デバッグビルドでのみ使用する開発専用のルートCA証明書を指定する場合は、以下のように設定します。

```xml:manifest.xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <!--<debug-overrides>タグの設定は、デバッグビルドのときだけ反映される-->
        <debug-overrides>
            <trust-anchors>
                <certificates src="@raw/debug_cas" />
            </trust-anchors>
        </debug-overrides>
    </network-security-config>
```

また、ドメイン(ホスト名)毎にルートCA証明書(例ではプライベートCA証明書)を切り替えることも可能です。

```xml:manifest.xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <!--<domain-config>タグを使用することで特定のドメインに対して有効なルートCAを限定します-->
        <domain-config>
            <domain includeSubdomains="true">onlyfordevelopment.com</domain>
            <trust-anchors>
                <certificates src="@raw/private_ca" />
            </trust-anchors>
        </domain-config>
    </network-security-config>
```

最後に、アプリが行う全てのHTTPS通信時を特定のルートCA証明書(例ではプライベートCA証明書)に限定する場合には、以下のように<base-config>タグを用います。

```xml:manifest.xml
    <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>
        <!--<base-config>タグではアプリ全体で有効なルートCAを限定します-->
        <base-config>
            <trust-anchors>
                <certificates src="@raw/private_ca" />
            </trust-anchors>
        </base-config>
    </network-security-config>
```

## 不適切な例

### AllowAllHostnameVerifierクラスを使用する  

以下のメソッドの引数にorg.apache.http.conn.ssl.AllowAllHostnameVerifierクラス(API 22以降deprecated)のインスタンスを指定した場合、ホスト名検証がバイパスされ、意図せぬサーバーに接続してしまう可能性があるため、使用しないでください。

-   org.apache.http.conn.ssl.SSLSocketFactoryクラス(API 22以降deprecated)のsetHostnameVerifier()メソッド
-   javax.net.ssl.HttpsURLConnectionクラスのsetHostnameVerifier()メソッド


```java
    void onConnectStart() {
        try {
            URL url = new URL("https://www.example.com");
            HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();            connection.setHostnameVerifier(new AllowAllHostnameVerifier());
            connection.connect();
        } catch (IOException e) {
        }
    }
```

LintはAllowAllHostnameVerifierクラスの使用を検知すると、次のような警告メッセージを出力します。

-   Lint結果(Warning)  
    "Using the AllowAllHostnameVerifier HostnameVerifier is unsafe because it always returns true, 
    which could cause insecure network traffic due to trusting TLS/SSL server certificates for wrong hostnames"

### 定数ALLOW_ALL_HOSTNAME_VERIFIERを使用する 

以下のメソッドの引数に定数org.apache.http.conn.ssl.SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIERを指定した場合、ホスト名検証がバイパスされ、意図せぬサーバーに接続してしまう可能性があるので、使用しないでください。

-   org.apache.http.conn.ssl.SSLSocketFactoryクラス(API22以降deprecated)のsetHostnameVerifier()メソッド
-   javax.net.ssl.HttpsURLConnectionクラスのsetHostnameVerifier()メソッド


```java
    protected void onConnectStart() {
        try {
            URL url = new URL("https://www.example.com");
            HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();            connection.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
            connection.connect();
        } catch (IOException e) {
        }
    }
```

LintはALLOW_ALL_HOSTNAME_VERIFIERの使用を検知すると、次のような警告メッセージを出力します。

-   Lint結果(Warning)  
    "Using the ALLOW_ALL_HOSTNAME_VERIFIER HostnameVerifier is unsafe because it always returns true, which could cause insecure network traffic due to trusting TLS/SSL server certificates for wrong hostnames"

## 外部リンク

-   [プレス発表【注意喚起】HTTPSで通信するAndroidアプリの開発者はSSLサーバー証明書の検証処理の実装を][1]
-   [Network Security Configuration | Android Developers][2]
-   [『Android アプリのセキュア設計・セキュアコーディングガイド』][3]  
    [「5.4.2.5. 独自のHostnameVerifier は作らない（必須）」][3-1]  
    [「5.4.3.3. 証明書検証を無効化する危険なコード」][3-2]  
    [「5.4.3.7. Network Security Configuration」][3-3]  


[1]: https://www.ipa.go.jp/about/press/20140919\_1.html
[2]: https://developer.android.com/training/articles/security-config.html
[3]: https://www.jssec.org/dl/android\_securecoding.pdf
[3-1]: http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#%E7%8B%AC%E8%87%AA%E3%81%AEhostnameverifier%E3%81%AF%E4%BD%9C%E3%82%89%E3%81%AA%E3%81%84-%EF%BC%88%E5%BF%85%E9%A0%88%EF%BC%89
[3-2]: http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#%E8%A8%BC%E6%98%8E%E6%9B%B8%E6%A4%9C%E8%A8%BC%E3%82%92%E7%84%A1%E5%8A%B9%E5%8C%96%E3%81%99%E3%82%8B%E5%8D%B1%E9%99%BA%E3%81%AA%E3%82%B3%E3%83%BC%E3%83%89
[3-3]: http://www.jssec.org/dl/android_securecoding/5_how_to_use_security_functions.html#network-security-configuration

## 注釈

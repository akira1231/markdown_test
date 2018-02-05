# Content provider does not require permission (ExportedContentProvider)

## 警告されている問題点

ContentProviderへのアクセス制御が十分に施されていないため、不特定多数のアプリからの情報の読み書きが発生し、大事な情報の漏洩や改ざんにつながるリスクがあります。

## 対策のポイント

-   原則、ContentProviderが扱う情報への不特定多数のアプリからのアクセスを不許可にしてください
-   ContentProviderの情報への不正なアクセスを許可しないよう適切なアクセス制御を施してください

## 対策の具体例

### ContentProviderを非公開にする

仕様・設計上問題なければContentProviderを非公開にします。Android 2.2(API
8)以前の端末ではContentProviderを非公開にできないため、Android 2.2(API
8)以前の端末を非サポートにする(minSdkVersionを9以上にする)ことを前提とします。

下に示す例のように`<provider>`で`android:exported`属性を`false`にし、ContentProviderを非公開に設定することで、不特定多数のアプリからのアクセスを不許可にできます。  
targetSdkVersionが16以前か17以降かによって、@android:exported@属性のデフォルト値が異なりますので、公開・非公開に関わらず明示的に設定してください。

> `<provider>`で`android:exported`属性が設定されていない場合の挙動は、`targetSdkVersion`によって以下のように異なります。非公開のつもりが誤って公開となるのを避け、また設計意図を明確にするために公開・非公開に関わらず明示的に設定することをお勧めします。  
> `targetSdkVersion`が16以下の場合：`android:exported`のデフォルト値は`true`  
> `targetSdkVersion`が17以上の場合：`android:exported`のデフォルト値は`false`

```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example"
        android:versionCode="1"
        android:versionName="1.0" >
        <uses-sdk android:minSdkVersion="9" />
        <application
            android:icon="@drawable/ic_launcher"
            android:label="@string/app_name" >
            <!-- 公開が不要のため明示的に非公開に設定する -->
            <provider
                android:exported="false"
                android:name="com.example.provider.providerClass3"
                android:authorities="com.example.provider.providerData">
            </provider>
        </application>
    </manifest>
    </code>
```

### ContentProviderへの適切なアクセスを制御を設定する

他のアプリからのアクセスが必要な場合はContentProviderへ目的に応じて適切なPermissionを設定してください。ContentProviderへのPermissionの設定の仕方には大きく次の3種類があります。

-   単一のPermissionにより他アプリからの読み書きを制限する\
    これは`<provider>`タグの`android:permission`属性としてPermissionを設定するもので、他のコンポーネント(Activity、Service、Broadcast
    Receiver)と同様のものです
-   読み書き別々にPermissionを設定する\
    `<provider>`タグの`<android:read-permission>`と`<android:write-permission>`によって読み書き別々にPermissionを設定するもので、他のコンポーネントにはなく、ContentProvider特有のものです
-   特定のパスに対してPermissionを設定する\
    `<provider>`の下位タグ`<path-permission>`により特定のパスに対してPermissionを設定するもので、これもContentProvider特有のものです

以下では上で述べた、それぞれのPermissionの設定方法について例を示します。

> ここではシステム定義のDangerous Permission(protecttionLevelがdangerousなPermission)を利用した例を取り上げます。第三者アプリはユーザーの許可によってPermissionを取得することが出来るため、実際の制限はユーザーの判断にゆだねられます。そのため、アクセス可能な情報はユーザー情報に限定することが必要です。
情報のやり取りを自社アプリに限定するなどその他のケースについては、「[Androidアプリのセキュア設計・セキュアコーディングガイド](1)」(外部リンク)にある情報はContentProviderを安全に利用する方法が具体的な例とともに示されておりお勧めです。

-   単一のPermissionにより他アプリからの読み書きを制限する  
下の例は、許可されたアプリだけがContentProviderにアクセス出来るように@WRITE\_CONTACTS@
    Permissionで制限しています。
        <code class="xml">
        <?xml version="1.0" encoding="utf-8"?>
        <manifest ... >
                :
            <application ... >
                <!-- Permission属性で他アプリがPermissionを必要とするように設定 -->
                <provider
                    android:exported="true"
                    android:name="com.example.provider.providerClass"
                    android:authorities="com.example.provider.providerData"
                    android:permission="android.permission.WRITE_CONTACTS">
                </provider>
                :
            </application>
            :
        </manifest>
        </code>

<!-- -->

-   読み書きそれぞれに別のPermissionを設定して制限する\
    下の例の場合、第三者アプリは@READ\_CONTACTS@
    Permissionを持たなければContentProviderの@query()`メソッドを呼びだすことができません。同様に`insert()`、`update()`あるいは`delete()`メソッドを呼び出すには、`WRITE\_CONTACTS@
    Permissionを持っている必要があります。
        <code class="xml">
        <?xml version="1.0" encoding="utf-8"?>
        <manifest ... >
                :
            <application ... >
                <!-- 読み書きそれぞれ別のPermissionを設定 -->
                <provider
                    android:exported="true"
                    android:name="com.sample.provider.providerClass"
                    android:authorities="com.example.provider.providerData"
                    android:readPermission="android.permission.READ_CONTACTS"
                    android:writePermission="android.permission.WRITE_CONTACTS">
                </provider>
                :
            </application>
            :
        </manifest>
        </code>

<!-- -->

-   `<path-permission>`によってアクセス可能なパスを限定する\
    `<path-permission>`を用いることによってアクセスを許可する情報の範囲を、特定のパターンにマッチするものに限定することができます。さらに各URI毎に別々のPermissionを設定することに加え、読み書き別々のPermissionを設定することも可能です。
        <code class="xml">
        <?xml version="1.0" encoding="utf-8"?>
        <manifest ... >
                :
            <application ... >
                <!-- <path-permission>でアクセス可能なパスを限定 -->
                <provider
                    android:exported="true"
                    android:name="com.example.provider.providerClass"
                    android:authorities="com.examble.provider.providerData">
                    <path-permission
                        android:pathPrefix="/you_can_access_here"
                        android:permission="android.permission.READ_CONTACTS">
                    </path-permission>
                </provider>
                :
            </application>
            :
        </manifest>

    </code>

不適切な例
----------

### `<path-permission>`がすべてのパスに対するアクセスを許可する設定になっている

Lintは`<path-permission>`のパス指定(`pathPrefix`など)の設定値については特に検査しません。例えば下記のような例では、ContentProviderの持つすべてのリソースへのアクセスを許可していますが、Lintは警告メッセージを出力しません。アクセス可能なパスの指定範囲が意図と異なっていないか、仕様や設計の確認を十分に行い、誤った設定にならないよう心がけてください。

    <code class="xml">
        <path-permission
            android:pathPrefix="/"
            android:permission="android.permission.READ_CONTACTS">
        </path-permission>
        <path-permission
            android:pathPattern="/.*"
            android:permission="android.permission.WRITE_CONTACTS">
        </path-permission>
    </code>

### 公開・非公開が不明瞭

下の例では@android:exported@属性の設定はありませんが、targetSdkVersionが16以下の場合、`<provider>`タブの@android:exported@属性のデフォルト値が@true@のため検出対象になります。ただし、LintはtargetSdkVersionに関わらず、`<provider>`に@android:exported@指定がないものをtrueとして扱っています。

    <code class="xml">
    <?xml version="1.0" encoding="utf-8"?>
    <manifest ...>
        :
        <application
           ... >

            <provider
                android:name="com.sample.provider.providerClass1"
                android:authorities="com.sample.provider.providerData">
            </provider>
        </application>
    </manifest>
    </code>

\
targetSdkVersionの違いによる振る舞いに依存せず、公開・非公開は@android:exported@属性によって明示的に設定してください。

問題を検出した場合、Lintは次のような警告を出力します。

-   Lint結果(Warning)

<!-- -->

    Exported content providers can provide access to potentially sensitive data 

### `<path-permmision>`タグをを記載しているがPermissionで守られていない(Lint検出対象外)

Providerの下位タグ`<path-permission>`でアクセス可能なパスを設定しているが、Providerと`<path-permission>`の両方でPermissionを設定していない場合実質的に任意の他のアプリからそのパスへのアクセスが可能となります。このようなケースをLintは検知せずあたかもProviderがPermissionで保護されている状況とみなします。しかし結果的に十分なアクセス制御がされている状況とはなっていないため、注意してください。

    <code class="xml">
      <provider android:name=".MyProvider"
                android:authorities="downloads"
                android:exported="true">
           <path-permission android:pathPrefix="/all_downloads" />
       </provider>
    </code>

### (注意) `<path-permission>`でPermissionを設定するとLintは誤って問題ありと指摘します

`<path-permission>`で(正しく)Permissionを設定すると、Lintは"Protecting
an unsupported element with a permission is a non-op and potentially
dangerous"と指摘するので注意してください。

    <code class="xml">
       <provider android:name=".MyProvider"
                 android:authorities="downloads" android:exported="true">
            <!-- path-permissionでandroid:permission属性は有効にもかかわらずLintは問題ありとする -->
            <path-permission android:pathPrefix="/all_downloads"
                android:permission="android.permission.ACCESS_ALL_DOWNLOADS"/>
       </provider>
    </code>

\
Lintは`<path-permission>`タグで`<android:permission>`は無効であると誤って判断しています。これについては「Invalid
Permission Attribute (InvalidPermission)」の項を参照してください。

外部リンク
----------

-   Content Provider \| Android Developers\
    https://developer.android.com/reference/android/content/ContentProvider.html

<!-- -->

-   Content Providerのガイド\
    https://developer.android.com/guide/topics/providers/content-providers.html

<!-- -->

-   `<path-permission>` \| Android Developers\
    https://developer.android.com/guide/topics/manifest/path-permission-element.html

<!-- -->

-   [Androidアプリのセキュア設計・セキュアコーディングガイド](1)  
    -   「4.3 Content Providerを作る・利用する」にContent
        Providerを安全に使用するための指針や実装例が解説されています
    -   「5.2 PermissionとProtection
        Level」にPermissionの適切な使用方法の解説があります

[1]: http://www.jssec.org/dl/android\_securecoding.pdf

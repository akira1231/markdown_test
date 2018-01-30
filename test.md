# Title
## 警告されている問題点
ほげほげほげほげ
## 対策のポイント
以下のいずれかの対策を実施してください。
- ほげほげ１
- ほげほげ２
- ほげほげ３
## 対策の具体例
### ほげほげ１に対する対策例（<-端的なタイトルを設定してください）
ほげほげほげほげほげほげほげ[^注釈１]ほげ。
ほげほげほげほげほげほげほげほげ。
```xml:manifest.xml
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
```
### ほげほげ2に対する対策例（<-端的なタイトルを設定してください）
ほげほげほげ[^注釈２]ほげほげほげほげほげ。  
ほげほげほげほげほげほげほげほげ。
> TIPSとしての引用引用  
> TIPSとしての引用引用  
> TIPSとしての引用引用  
### ほげほげ3に対する対策例（<-端的なタイトルを設定してください）
ほげほげ `hoge1 = 1`  ほげほげ  
ほげほげほげほげほげほげほげほげ。
## 不適切な例
### こんな不適切な例
ほげほげほげほげほげほげほげほげ。  
ほげほげほげほげほげほげ[^注釈３]ほげほげ。
```xml:manifest.xml
<path-permission
    android:pathPrefix="/"
    android:permission="android.permission.READ_CONTACTS">
</path-permission>
```
### あんな不適切な例
ほげほげほげほげほげほげほげほげ。  
[ほげほげほげほげほげほげほげほげ。][リンク１]

## 外部リンク
[リンク１のテキスト][リンク１]
[リンク１]: http://hogehoge1
[リンク２のテキスト][リンク２]
[リンク２]: http://hogehoge2
[リンク３のテキスト][リンク３]
[リンク３]: http://hogehoge3

###注釈

[^注釈１]: ほげほげほげ１

[^注釈２]: ほげほげほげ２

[^注釈３]: ほげほげほげ３。[リンク４](http://hogehoge)

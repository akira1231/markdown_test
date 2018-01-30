# markdown_test
Appearance test for Markdown

>>> minSdkVersionを9以上にする[1](#1)


```xml
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
```

###### 1
Android 2.2 (API 8)以前の端末ではContentProviderは、`<provider>`で`android:exported`属性の値に関わらず公開になってしまうため、アプリの仕様上問題無ければAndroid 2.2(API 8)以前の端末を非サポートにする(minSdkVersionを9以上にする)ことをお勧めします

-   Content Provider \| Android Developers\
    https://developer.android.com/reference/android/content/ContentProvider.html

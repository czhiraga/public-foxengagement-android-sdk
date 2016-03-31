# - F.O.X Engagement Android Integration Guide -

## F.O.Xエンゲージメント配信とは

本サービスは、以下２つのキャンペーンを配信するためのネットワーク広告となっています。

* **アクション目的の広告配信**

データフィードを活用しダイナミッククリエイティブ、レコメンド広告を配信します。

* **インストール目的の広告配信**

アドネットワークのようなピクチャーバナーによる広告を配信します。

## 目次

* **[1. インストール](#install)**
* **[2. API](#about_api)**
* **[3. コードへの組み込み](#code_sample)**

<div id="install"></div>
## 1. インストール

### 1.1 入手

> [SDKダウンロード](https://github.com/cyber-z/public-foxengagement-android-sdk/releases)

直接jarファイルをプロジェクトに組み込む際には、SDK本体を[ダウンロード](https://github.com/cyber-z/public-foxengagement-android-sdk/releases)してください。
ダウンロードしたSDKを展開し、jarファイルをアプリケーションのプロジェクトに組み込んでください。

### 1.2 Gradleによる導入

Android StudioプロジェクトにてGradleを使用し導入する場合は以下の手順を行います。<br>
`build.gradle`に下記設定を適切な箇所に追加します。

```
repositories {
    maven {
        url "https://github.com/cyber-z/public-foxengagement-android-sdk/raw/master/mavenRepo"
    }
}

dependencies {
    compile 'co.cyberz.fox:engagement-sdk:1.0.0'
}
```

### 1.3 GooglePlayServicesの導入

本SDKの組み込みにはlibsの`FOX_Engagement_Android_SDK_<version>.jar`と別途`GooglePlayServices`が必須となります。<br>
組み込み対象のアプリにはGooglePlayServicesをご導入の上、AdvertisingIDを取得出来る状態が必須となっております。

[GradleにおけるGooglePlayServicesのdependenciesの設定]

```
compile 'com.google.android.gms:play-services:8.3.0'
```

広告に関する機能のみを使う場合、以下のように設定することで広告機能のみをインポートすることが可能です。

```
compile 'com.google.android.gms:play-services-ads:8.3.0'
```


**[EclipseにおけるGooglePlayServicesの設定]**

以下のライブラリプロジェクトをプロジェクトにインポートします。
```
{android_sdk}/extras/google/google_play_services/libproject/google-play-services_lib
```
> 詳細は[Goolge Developers](https://developers.google.com/android/guides/setup)をご確認ください。

### 1.4 AndroidManifest.xmlの設定

**[パーミッションの設定]**

本SDKは`INTERNET`・`ACCESS_NETWORK_STATE`の２つのパーミッションが必須ととなります。

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

**[meta-dataの設定]**

* `FEG_API_KEY`:発行されたAPI KEYを指定してください。
* GooglePlayServiceの仕様上、必要となります。(固定)

```xml
<meta-data
    android:name="FEG_API_KEY"
    android:value="発行されたAPI KEY" />
<meta-data
    android:name="com.google.android.gms.version"
    android:value="@integer/google_play_services_version" />
```

<div id="about_api"></div>
## 2. API

### AdViewクラス

|返り値型|メソッド|詳細|
|---:|:---|:---|
|-|AdView ( Context c )|コンストラクター|
|-|AdView ( Context c, AttributeSet attrs )|コンストラクター|
|void|show ( String placementId , AdSize adSize)<br><br>`placementID` : 広告表示ID (管理者より発行されます)<br>`adSize` : 広告サイズ|バナー広告を表示します。|
|void|show ( String placementId , AdSize adSize, DahliaAdViewListener listener )<br><br>`placementID` : 広告表示ID (管理者より発行されます)<br>`adSize` : 広告サイズ<br>`listener` : 広告表示の際のイベントを取得するためのリスナー|バナー広告を表示します。|

### AdView.OnStateListenerインターフェース

|返り値型|メソッド|詳細|
|---:|:---|:---|
|void|onSuccess ( View v )<br><br>`v` : 広告のView|広告の表示が正常だった場合に呼ばれます。|
|void|onFailed ( View v ) <br><br> `v` : 広告のView|広告が表示できなかった場合に呼ばれます。|
|void|onClicked ( View v ) <br><br> `v` : 広告のView|広告がクリックされた場合に呼ばれます。|
|boolean|onFallback ( )|表示する広告がなかった場合に呼ばれます。Fallbackの場合、メディア側で用意したクリエイティブを表示することが可能となっています。その場合は返り値にtrueを指定してください。また、任意の処理を行うため広告枠を非表示にする場合にはfalseを返してください。|

> ・onFallbackが発生しtrueを返した場合、メディア側で用意したクリエイティブが表示される際のonSuccessとonFailedは呼ばれません。

### AdSizeクラス

|型|変数名|詳細|
|:---:|:---:|:---|
|static|B320x50|横320×縦50(ピクセル)の広告をリクエストします|
|static|B320x100|横320×縦100(ピクセル)の広告をリクエストします|
|static|B300x250|横300×縦250(ピクセル)の広告をリクエストします|
|static|B336x280|横336×縦280(ピクセル)の広告をリクエストします|
|static|B112x112|横112×縦112(ピクセル)の広告をリクエストします|

> 表示する際、端末の解像度別の倍率（Density）で最適化されます。

<div id="code_sample"></div>
## 3. コードへの組み込み

### 3.1 バナー広告表示サンプル その１

javaコードのみでの実装

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.test_activity);

   // 既存レイアウトに追加
   LinearLayout ll = (LinearLayout) findViewById(R.id.scroll);
   // バナー広告表示View
   AdView mAdView = new AdView(this);
   mAdView.show("広告表示ID", AdSize.B320x100, new AdView.OnStateListener() {
      @Override
      public void onSuccess(View v) {
        // バナー広告が正常に表示された場合の処理
        Toast.makeText(this, "成功", Toast.LENGTH_SHORT).show();
      }

      @Override
      public void onFailed(View v) {
        // バナー広告の読み込みに失敗した場合の処理
        Toast.makeText(this, "読み込み失敗", Toast.LENGTH_SHORT).show();
        v.setVisibility(View.GONE);
      }

      @Override
	  public void onClick(View v) {
        Toast.makeText(BigBenBannerTestActivity.this, "ユーザーにクリックされました", Toast.LENGTH_SHORT).show();
  	  }

	  @Override
	  public boolean onFallback() {
        Toast.makeText(BigBenBannerTestActivity.this, "Fallback", Toast.LENGTH_SHORT).show();
        // そのままメディア側で送られてきたクリエイティブを表示する場合はtrue
        return true;
      }
   });
   ll.addView(mAdView);
}
```

### 3.2 バナー広告表示サンプル その２

layoutのXMLで定義しての実装

[xml]
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scroll"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- widthとheightはwrap_contentを指定してください -->
    <co.cyberz.engagement.view.banner.AdView
        android:id="@+id/banner"
        android:layout_gravity="center_horizontal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>  
```

> ※`layout_width`と`layout_height`には`"wrap_content"`を指定してください。

[java]
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.test_activity);

   // xmlからBannerViewオブジェクトの呼び出し
   AdView mAdView = (AdView) findViewById(R.id.banner);
   // バナー広告表示View
   mAdView.show("広告表示ID", AdSize.B320x100, new AdView.OnStateListener() {
      @Override
      public void onSuccess(View v) {
        // バナー広告が正常に表示された場合の処理
        Toast.makeText(this, "成功", Toast.LENGTH_SHORT).show();
      }

      @Override
      public void onFailed(View v) {
        // バナー広告の読み込みに失敗した場合の処理
        Toast.makeText(this, "読み込み失敗", Toast.LENGTH_SHORT).show();
        v.setVisibility(View.GONE);
      }

      @Override
   public void onClick(View v) {
        Toast.makeText(this, "ユーザーにクリックされました", Toast.LENGTH_SHORT).show();
     }

   @Override
   public boolean onFallback() {
        Toast.makeText(this, "Fallback", Toast.LENGTH_SHORT).show();
        // そのままメディア側で送られてきたクリエイティブを表示する場合はtrue
        return true;
      }
   });
}
```

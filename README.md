# Appmartアプリ内課金: PhoneGap

![last-version](http://img.shields.io/badge/last%20version-1.1-green.svg "last version:1.1") 

![license apache 2.0](http://img.shields.io/badge/license-apache%202.0-brightgreen.svg "licence apache 2.0")

AppmartのPhoneGap用のアプリ内課金システムのプラグインです。このサンプルをfork・cloneしていただき、自由にご利用ください。 

このサンプルの対象サービスは:

+ アプリ内課金：都度決済 

---

## 目次

```
1- 導入手順

	I- pluginを導入（project型）
		- パーミッション設定
		- サンプルをclone
		- Workspaceに追加 (Eclipse)
		- PhoneGapプロジェクトに導入（Eclipse）

	II- プラグイン組み込み
		- Activityクラスを変更
		- Javaとの連動
		- HTMLを変更

2- リファレンス

	I- エラーメッセージ

```



---


## 導入手順


### pluginを導入（project型）


#### パーミッション設定

```xml
<!-- 課金API用 -->
<uses-permission android:name="jp.app_mart.permissions.APPMART_BILLING" />
```

#### サンプルをclone

```shell
cd /home/user/your_directory
git clone https://github.com/info-appmart/appmart-inbilling-phonegap
```

> 注意点：　Eclipseにうまく読み込まれないために、workspace以外のフォルダーにcloneしてください。


#### Workspaceに追加 (eclipse)

+ ⇒ File
+ ⇒ Import
+ ⇒ Existing Android Code Into Workspace
+ ⇒ 先ほどcloneしたプロジェクトを選択

![Eclipse:appmart phoneGap](http://s14.postimg.org/5hbwc1t7l/phonegap_capture_2.png "Eclipse:appmart phoneGap")

#### PhoneGapプロジェクトに導入（eclipse）

+ ⇒ PhoneGapプロジェクトに右クリック　
+ ⇒ Properties 
+ ⇒ Android
+ ⇒ Libraries  :  Add (Pluginを選択)

![Eclipse:appmart phoneGap](http://s27.postimg.org/wza6sbrcj/phonegap_plugin.png "Eclipse:appmart phoneGap")

### プラグイン組み込み

#### Activityクラスを変更

> Oncreate methodを更新


```java
//javascriptInterfaceオブジェクト
private MyJavascriptInterface mc;
	
//開発者情報
public String devId = "YOUR_DEVELOPPER_ID";
public String licenceKey = "YOUR_LICENCE_KEY";
public String publicKey= "YOUR_PUBLIC_KEY";
public String appId= "YOUR_APP_ID";

@Override
public void onCreate(Bundle savedInstanceState){
	
    super.onCreate(savedInstanceState);

    //下記3行を追記
    super.init();
    mc = new MyJavascriptInterface(this);
    appView.addJavascriptInterface(mc, "appmart");
    
    loadUrl(launchUrl);
}
    
```

> 【開発情報】を書き換えてください。デベロッパー管理画面よりご確認いただけます。(サービス管理>検索>アプリ名)


![Eclipse:appmart phoneGap](http://s21.postimg.org/h5xp3grw7/appmart_info.png "Eclipse:appmart phoneGap")


#### Javaとの連動

> Javaと連動するために、JavascriptInterfaceクラスを用意します。 

> 内部クラスのため、**activityクラス内に定義してください**。　callbackオブジェクト経由で決済画面よりのデータを受け取ります。

```java
class MyJavascriptInterface{
	
	Context mContext;
	PhoneGapPlugin plugin;
	    	
	/* constructor */
	public MyJavascriptInterface(Context context){
		mContext= context;
	}
		
	/* 決済実行 */
	@JavascriptInterface
	public void doSettlement(String serviceId){
		
		//Callbackオブジェクト
		AppmartResultInterface callback = new AppmartResultInterface(){
			
			//決済ID
			String transactionId;

			/* エラー発生時呼び出し */
			@Override
			public void settlementError(int errorCode) {
				Toast.makeText(mContext, "エラー発生：　errorCode: "+errorCode, Toast.LENGTH_LONG).show();					
			}

			/* 決済画面から戻ってきた時に呼び出し（決済はまだ未確定！）　*/
			@Override
			public void settlementWaitValidation(String transactionId) {
				
				// 決済IDを保存
				this.transactionId = transactionId;
				
				//TODO ユーザーにコンテンツを提供
				
				//決済を確定
				plugin.confirmSettlement();
				
			}

			/* 決済が確定された際に呼び出し */
			@Override
			public void settlementValidated(boolean result) {
				if(result)
				Toast.makeText(mContext, "決済が確定されました。決済ID: "+transactionId, Toast.LENGTH_LONG).show();
				else
				Toast.makeText(mContext, "決済が確定されませんでした", Toast.LENGTH_LONG).show();					
			}
			
		};
		    		
		
		plugin = PhoneGapPlugin.getInstance(mContext); 		
		plugin.setParameters(devId, licenceKey, publicKey, appId);
		plugin.doSettlement(serviceId, callback);
	}
	
}
```

#### HTMLを変更

> ボタンをクリックする際にjavascriptでMyJavascriptInterfaceのdoSettlement関数を呼び出します。


```html
<script>       

	var alreadyClicked = new Array(); 

    function do_settlement(obj, itemId){

    	for (var i = 0; i < alreadyClicked.length; i ++) {
        		if(alreadyClicked[i] == itemId.id){
        			alert("既にクリックされました");
        			return;
        		}        		
        	}
        	
    	//重複クリック防止
		alreadyClicked.push(itemId.id);

    	//決済	
    	window.appmart.doSettlement(itemId);
    }
</script>

<button id="first_button" onclick="do_settlement(this, 'your_service_id')" value="settlement">Settlement</button>

```

##  リファレンス

### エラーメッセージ


| エラーコード     |説明  |
|---------- | ---|
|-1|サービスIDがnull|
|-2|パラメータ問題|
|-3|例外発生|
|-22|service　bind不可能。|

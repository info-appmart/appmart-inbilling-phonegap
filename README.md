# Appmartアプリ内課金: PhoneGap

![last-version](http://img.shields.io/badge/last%20version-1.1-green.svg "last version:1.1") 

![license apache 2.0](http://img.shields.io/badge/license-apache%202.0-brightgreen.svg "licence apache 2.0")

PhoneGapアプリ用のAppmartアプリ内課金システムのプラグインです。このサンプルをfork・cloneしていただき、自由にご利用ください。 

このサンプルの対象サービスは:

+ アプリ内課金：都度決済 

---


## Ready-to-useサンプル


### プロジェクト設定 

> サンプルをclone

```shell
cd /home/user/your_directory
git clone https://github.com/info-appmart/appmartPhoneGapPlugin.git
```

> Workspaceに追加 (eclipse)

- File => Import => Existing Android Code Int Workspace
- 先ほどcloneしたプロジェクトを選択


> PhoneGapプロジェクトに導入（eclipse）

PhoneGapプロジェクトに右クリック　=>　properties => Android　 ⇒　Libraries => Add => Pluginを選択

### プラグイン組み込み

> Activityクラスを変更

###### Oncreate methodを更新

```java

private MyJavascriptInterface mc;
	
//開発者情報
public String devId = "YOUR_DEVELOPPER_ID";
public String licenceKey = "YOUR_LICENCE_KEY";
public String publicKey= "YOUR_PUBLIC_KEY";
public String appId= "YOUR_APP_ID";

@Override
public void onCreate(Bundle savedInstanceState){
	
    super.onCreate(savedInstanceState);

    //追記
    super.init();
    mc = new MyJavascriptInterface(this);
    appView.addJavascriptInterface(mc, "appmart");
    
    loadUrl(launchUrl);
}
    
```


###### JavascripInterfaceクラスを作成 (Activityクラス内に追加)


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
		
		AppmartResultInterface callback = new AppmartResultInterface(){
			
			//決済ID
			String transactionId;

			@Override
			public void settlementError(int errorCode) {
				Toast.makeText(mContext, "エラー発生：　errorCode: "+errorCode, Toast.LENGTH_LONG).show();					
			}

			@Override
			public void settlementWaitValidation(String transactionId) {
				
				// 決済IDを保存
				this.transactionId = transactionId;
				
				//TODO ユーザーにコンテンツを提供
				
				//決済を確定
				plugin.confirmSettlement();
				
			}

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


> htmlを変更


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



> パーミッション設定

```xml
<!-- 課金API用 -->
<uses-permission android:name="jp.app_mart.permissions.APPMART_BILLING" />
```



##  リファレンス

### エラーメッセージ


| エラーコード     |説明  |
|---------- | ---|
|-1|サービスIDがnull|
|-2|パラメータ問題|
|-3|例外発生|
|-22|service　bind不可能。|

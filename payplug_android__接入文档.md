### PayPlug Android  接入文档
1. 添加PayPlug1.0.0.jar依赖

    下载PayPlug1.0.0.jar，拷贝到`Module/libs`目录下，在`Module/build.gradle`中添加依赖
   
        dependencies {
            compile files('libs/PayPlug1.0.0.jar')
        }
2. 配置应用权限
   
    主应用需要几个基础的权限配置，请将以下的几个权限加入到主应用的AndroidManifest.xml中
   
         <uses-permission android:name="android.permission.INTERNET"/>
          <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
          <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
          <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
          <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
          <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
          <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
          <uses-permission android:name="android.permission.RECEIVE_SMS"/>
          <uses-permission android:name="android.permission.SEND_SMS"/>
          <uses-permission android:name="android.permission.READ_SMS"/>
          <uses-permission android:name="android.permission.GET_TASKS"/>
          <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
          <uses-permission android:name="android.permission.USE_CREDENTIALS"/>
          <uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
          <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS"/>
          <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
          <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
          <uses-permission android:name="android.permission.WRITE_SETTINGS"/>
          <uses-permission android:name="android.permission.RECORD_AUDIO"/>
          <uses-permission android:name="android.permission.MESSAGE"/>
          <uses-permission android:name="android.permission.RECEIVE"/>
          <uses-permission android:name="android.permission.DOWNLOAD_WITHOUT_NOTIFICATION"/>
          <uses-permission android:name="android.permission.RESTART_PACKAGES"/>
          <uses-permission android:name="android.permission.VIBRATE"/>
          <uses-permission android:name="android.permission.EXPAND_STATUS_BAR"/>
          <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
          <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
          <uses-feature android:name="android.hardware.camera"/>
          <uses-permission android:name="android.permission.CAMERA"/>
          <uses-feature android:name="android.hardware.camera.autofocus"/>
          <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
          <uses-permission android:name="android.permission.RECORD_VIDEO"/>
          <uses-permission android:name="com.lenovo.lsf.device.permission.MESSAGE"/>
          <uses-permission android:name="com.lenovo.lsf.device.permission.RECEIVE"/>

    另外将以下组件加入到`<application></application>`节点中
    
    
        <activity android:name="org.apkplug.app.apkplugActivity"
            android:configChanges="orientation|screenSize|keyboardHidden" 
            android:theme="@android:style/Theme.Translucent.NoTitleBar">
            <intent-filter>
                <Action android:name="android.intent.action.MAIN"/>
            </intent-filter>
        </activity>
        
        <activity android:launchMode="singleTop"
            android:name=".wxapi.WXPayEntryActivity"
            android:exported="true"/>
        
        <service android:name="org.apkplug.app.apkplugService"/>
        
        <service android:name="org.openudid.OpenUDID_service">
            <intent-filter>
                <Action android:name="org.OpenUDID.GETUDID"/>
            </intent-filter>
        </service>
        
        <provider android:name="org.apkplug.app.apkplugProvider" android:authorities="xxx.apkplugprovider"/>
    
    注：xxx为应用包名
    
    加入PayPlug后台应用下容器的id

        <meta-data  android:name="payplug_containerid" android:value="xxx" ></meta-data>
        
3. 下载容器对应资源包

1. 初始化SDK

        public class ProxyApplication extends Application {
        	private FrameworkInstance frame=null;
        	public void onCreate() {
        		super.onCreate();
        
        		try {
        			frame= FrameworkFactory.getInstance().start(null, this);
        			final BundleContext context =frame.getSystemBundleContext();
        			String appid="你的appid";
        			String app_secret="你的app_secret";
        			PayManager.getInstance().init(context,appid,app_secret);
        
        		} catch (Exception e) {
        			e.printStackTrace();
        		}
        
        
        	}
        }

    
2. 向PayPlug后台请求支付凭证

        OkHttpClient client = new OkHttpClient.Builder().connectTimeout(30, TimeUnit.SECONDS).build();
        
        RequestBody formBody = new FormBody.Builder()
                .add("app_id", "") //应用id
                .add("container_id", "") //容器id
                .add("order_no", "") //订单号
                .add("commodity_id", "") //商品id
                .add("amount", "") //商品总价
                .add("count", "") //商品数量
                .add("channel", "") //支付渠道
                .add("client_ip", "") //客户端ip
                .add("currency", "cny") //货币类型
                .add("subject", "") //订单名称
                .add("body", "") //订单内容
                .add("webhook_url", "") //订单回调接口
                .add("description", "") //订单描述
                .add("extra","{}") //自定义参数
                .add("_time", "") //下单时间
                .add("sign", "") //下单时间
                .build();
                
        Request request = new Request.Builder()
                .url("http://api.payplug.cn/v1/order/create.json")
                .post(formBody)
                .build();
                
        client.newCall(request).enqueue(new Callback() {
                    @Override public void onFailure(Request request, IOException throwable) {
                        throwable.printStackTrace();
                    }
        
                    @Override public void onResponse(Response response) throws IOException {
                        if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
                        //支付凭证
                        String orderInfo = response.body().string();
                    }
                });
        
3. 传入orderInfo调用支付

        PayManager.getInstance().doPay(orderInfo, new ResultCallBack() {
                        @Override
                        public void onResult(int resultCode, PayResponse resultInfo) {
                            if (resultCode == ResultCallBack.PAY_SUCCESS) {
   
        
                            } else if (resultCode == ResultCallBack.PAY_CANCEL) {
                                
                                       
                            } else if (resultCode == ResultCallBack.PAY_FAIL) {
        
        
                            } else if (resultCode == ResultCallBack.PAY_HANDLING) {
        
                            } else if (resultCode == ResultCallBack.PLUG_VERSION_TO_LOW) {
        
                            }else if (resultCode == ResultCallBack.PAY_PLUG_NOFIND) {
        
                            }else if (resultCode == ResultCallBack.PAY_CONFIG_PARAMS_ERROR) {
                                
                            }else if (resultCode == ResultCallBack.PAY_REQUEST_PARAMS_ERROR) {

                            }
                        }
                           });
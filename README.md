# 消息推送

## 简介  
卓易推送(DroiPush)是面向开发者提供消息推送服务的功能模块，开发者可以通过服务器开放API或者DroiBaaS管理后台向集成该模块的应用推送消息。  

## 安装  

### 安装依赖包  

由于卓易推送SDK依赖CoreSDK和第三方包，所以需要配置依赖关系和下载地址；  

###依赖包仓库  

在工程Module目录下`build.gradle`文件中配置如下依赖包仓库地址：

    repositories {
        ......  
        
        // 第三方依赖包仓库地址
        mavenCentral()
        
        // Core 和 Push SDK 仓库地址
        maven {
            url "https://github.com/DroiBaaS/DroiBaaS-SDK-Android/raw/master/"
        }
        
        ......
    }

###依赖关系  

在工程Module目录下`build.gradle`文件中配置如下依赖关系：

    dependencies {
        compile fileTree(include: ['*.jar'], dir: 'libs')
        
        // Core SDK
        compile 'com.droi.sdk:Core:+'
        
        // 推送 SDK
        compile 'com.droi.sdk:push:+'
        
        // OkHttp用于网络通信，版本不低于3.8.0
        compile 'com.squareup.okhttp3:okhttp:3.8.0'
        
        // Android Support V4 包
        compile 'com.android.support:support-v4:23.2.1'
        
        //lz4-java（Core-1.1.4128版本后必须添加）
        compile 'org.lz4:lz4-java:1.4.0'
        ......
    }

Core SDK 详细内容请参考DroiBaaS官网[快速入门](http://www.droibaas.com/html/doc_24138.html)。

**注意：  
1. Okhttp版本不低于3.8.0；  
2. Core-1.1.4128版本开始需要添加lz4-java依赖**  

### 推送SDK配置参数  

在AndroidManifest中可以配置应用的AppID和渠道号：  

    <application
        ......   >
        
        <!-- 其他配置 -->
             ......
        
        <!-- BaaS平台应用注册AppID，必填 -->
        <meta-data
            android:name="com.droi.sdk.application_id"
            android:value="应用的AppID" />
        
        <!-- 应用渠道号，用于标记统计、自更新渠道， 可选填 -->
        <meta-data
            android:name="com.droi.sdk.channel_name"
            android:value="应用指定的渠道号" />
            
        <!-- 其他配置 -->
             ......
             
    </application>
	
在工程Module目录的`build.gradle`文件中配置`DROI_PUSH_APPID`参数，内容为应用AppID：  

    android {  
    
        defaultConfig {  
        
            /*配置applicationId */  
            applicationId "你的应用包名"  
        
            /*配置应用APPID */  
            manifestPlaceholders = [DROI_PUSH_APPID: '你的应用AppID']  
        }  
        
        // 其他配置...  
    }  

**注意：  
Android Studio工程build.gradle需要配置`applicationId`和`DROI_PUSH_APPID`信息，否则会影响推送；**  

## 使用

1. 初始化  
在Application的`onCreate()`方法中调用：  

		DroiPush.initialize(context, apiKey);  

    **注意：  
	由于DroiBaaS策略变更，推送SDK初始化需传入ApiKey参数。获取步骤如下：  
	若推送服务未激活，先通过web控制台->应用管理中心->设置详情->安全设置->推送服务激活服务;  
	通过web控制台->应用管理中心->应用设置->安全密钥获取推送ApiKey；**  

2. 获取设备、应用ID与渠道  
SDK提供接口获取设备ID(deviceId)、应用ID(appId)、渠道号(channel)等信息。

	- 获取deviceId  

	        DroiPush.getDeviceIdInBackground(mContext, new GetDeviceIdCallback(){
                @Override
                public void onGetDeviceId(String deviceId) {
                    // 通过回调返回deviceId，成功返回deviceId字符串，否则返回null
                }
            });
	        
	- 获取appId  

		    String appId = DroiPush.getAppId(context);

	- 获取channel  

		    String channel = DroiPush.getChannel(context);

    **注意：  
	deviceId首次获取需要联网，无网络情况下返回null；** 

3. 标签操作  
   卓易推送支持向指定标签所有设备推送消息，提供`覆盖`、`追加`、`删除`、`获取`标签的接口。标签参数为所有标签拼接字符串，各标签之间使用`英文逗号`分隔。

	- 覆盖标签  
	  使用传入标签替换已有标签，接口调用如下所示：  

            DroiPush.replaceTagsInBackground(getApplicationContext(), tag, new DroiCallback<String>(){  
                @Override  
                public void result(final String result, DroiError droiError) {  
                    // result为服务端返回信息  
                    if (droiError.isOk()) {  
                        // 替换标签成功  
                    } else {  
                        // 获取出错提示信息  
                        String resultStr = droiError.getAppendedMessage();  
                    }  
                }  
            });  

	- 追加标签  
	将传入标签追加到已有标签当中，接口调用如下所示：  

		    DroiPush.appendTagsInBackground(getApplicationContext(), tag, new DroiCallback<String>() {  
                @Override  
                public void result(final String result, DroiError droiError) { 
                    // result为服务端返回信息
                    String resultStr = "";  
                    if (droiError.isOk()) {  
                        // 追加标签成功  
                    } else {  
                        // 获取错误码及提示信息  
                        int errorCode = droiError.getCode();  
                        String errorInfo = droiError.getAppendedMessage();  
                    }  
                }  
            });  

	- 删除标签  
	从已有标签中删除传入的标签，接口调用如下所示：  

		    DroiPush.deleteTagsInBackground(getApplicationContext(), tag, new DroiCallback<String>() {  
                @Override  
                public void result(final String result, DroiError droiError) {  
                    // result为服务端返回信息  
                    if (droiError.isOk()) {  
                        // 删除标签成功  
                    } else {  
                        // 获取错误码及提示信息  
                        int errorCode = droiError.getCode();  
                        String errorInfo = droiError.getAppendedMessage();  
                    }  
                }  
            });  

	- 获取所有标签  
	查询标签并返回，接口调用如下所示：  

			DroiPush.queryTagsInBackground(getApplicationContext(), new DroiCallback<String>() {  
                @Override  
                public void result(final String result, DroiError droiError) {  
                    if (droiError.isOk()) {  
                        // 获取标签成功，result为标签JSON串，例如{"tags":"aa,bb"}    
                    } else {  
                        // 获取错误码及提示信息  
                        int errorCode = droiError.getCode();  
                        String errorInfo = droiError.getAppendedMessage();  
                    }  
                }  
            });	 
    
        返回标签为JSON字符串，以下为字段含义：  
        
| 字段名称 | 字段含义   |  
| --------  | ----- |  
| tags | 所有标签拼接字符串，使用英文逗号分隔，如"aa,bb,cc"   |  

**注意：  
	应用tag数量最多限制为1024个，每个长度最多为256字符；  
    tag内容不能加入URLEncode等其他的变换处理，建议使用原生字符串；**    

4. 设置静默时间  
   静默时间指的每天对应的时间段内，不展示推送消息；比如为了防止骚扰到用户，可以把每天0:00到6:00设置为静默时间。 

	- 设置静默时间  

			DroiPush.setSilentTime(context, 0, 0, 6, 0);

	- 获取静默时间  

			int[] time = DroiPush.getSilentTime(context);

	- 清除静默时间  

			DroiPush.clearSilentTime(context);

5. 启用/禁用推送  
禁用后，推送功能将不可用，请谨慎调用  

	- 启用/禁用推送  
	
            DroiPush.setPushableState(context, true);

	- 获取推送开关状态  

			boolean isEnabled = DroiPush.getPushableState(context);

6. 透传消息处理  
DroiMessageHandler抽象类提供处理透传消息的方法`onHandleCustomMessage`，以下为示例代码：

        DroiPush.setMessageHandler(new DroiMessageHandler() {
            @Override
            public void onHandleCustomMessage(final Context context, final String message) {
                // TODO Auto-generated method stub
                //message为透传消息内容，应用获取后自行处理
                new Handler(getMainLooper()).post(new Runnable() {
                    @Override
                    public void run() {
                        // TODO Auto-generated method stub
                        Toast.makeText(context, "Demo Custom Msg:" + message, Toast.LENGTH_LONG).show();
                    }
                });
            }
        });

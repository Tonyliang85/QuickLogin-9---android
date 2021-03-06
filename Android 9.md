#1. 接入指南

sdk技术问题沟通QQ群：609994083</br>

**注意事项：**

1. 目前SDK支持中国移动2/3/4G、中国电信4G的取号能力，中国联通的取号能力暂未开放。
2. 一键登录取号能力在取号过程中，用户的手机终端的数据流量必须打开或者终端需允许应用使用数据流量（WiFi和数据流量同时打开时，SDK会强制切换到数据流量执行取号逻辑，将会消耗用户少量流量），当信号弱或者网络有干扰时，时延会高于平均值，取号成功率较低。
3. 对于双卡手机，移动认证SDK只对当前的流量卡取号，如果用户需要使用第二张卡登录，建议开发者在授权登录页面上增加其他登录方式。

## 1.1. 接入流程

**1.申请appid和appkey**

根据《开发者接入流程文档》，前往中国移动开发者社区（dev.10086.cn)，按照文档要求创建开发者账号并申请appid和appkey，并填写应用的包名和包签名。

**2.申请能力**

应用创建完成后，在能力配置页面上，勾选应用需要接入的能力类型，如一键登录，并配置应用的服务器出口IP地址。（如果在服务端需要用非对称加密方法对一些重要信息进行加密处理，请在能力配置页面填写RSA加密的公钥）

**3.添加appid白名单**

开发者在完成步骤1和步骤2后，将appid提供给移动认证工作人员，移动方将在2个工作日内将appid加入白名单，白名单添加完毕，开发者就可以开始联调对接。

**4.上线审核**

应用上线前，开发者需要将一键登录取号能力的场景所使用的授权页面（授权页面参考授权页面规范）提供给移动认证产品接口人，审核无误后可正式上线。

## 1.2. 开发流程

**第一步：下载SDK及相关文档**

请在开发者群或官网下载最新的SDK包

**第二步：搭建开发环境**

jar包集成方式：

1. 在Eclipse/AS中建立你的工程。 
2. 将`*.jar`拷贝到工程的libs目录下，如没有该目录，可新建。

如果使用android studio进行开发，在app的主module的build.gradle中加入依赖配置：

```
implementation 'com.cmictop.sso:sdk:9.0.4'
```

注：其中9.0.4是一键登录对应的SDK的版本号


**第三步：开始使用移动认证SDK**

**[1] AndroidManifest.xml设置**

添加必要的权限支持: 

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_SETTINGS"/>
```

注意：在调用`取号`或`授权登录`方法时，需提前申请`READ_PHONE_STATE`，否则会报错

**[2] 创建一个AuthnHelper实例。**

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过AuthnHelper进行调用。因此，调用SDK，首先需要创建一个AuthnHelper实例

**示例代码：**

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }
```
**方法原型：**

```java
public static AuthnHelper getInstance(Context context)
```

**参数说明：**

| 参数    | 类型    | 说明                                               |
| ------- | ------- | -------------------------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

**[3] 实现回调。**

所有的SDK接口调用，都会传入一个回调，用于接收SDK返回的调用结果。结果以`JsonObject`的形式传递，`TokenListener`的实现示例代码如下：

```java
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        if (jObj != null) {
            mResultString = jObj.toString();
            mHandler.sendEmptyMessage(RESULT);
            if (jObj.has("token")) {
                mtoken = jObj.optString("token");
            }
        }
    }
};
```
**[4] 混淆策略**

请避免混淆一键登录SDK，在Proguard混淆文件中增加以下配置：

```java
-dontwarn com.cmic.sso.sdk.**
-keep public class com.cmic.sso.sdk.*{;}
```



<div STYLE="page-break-after: always;"></div>

# 2. 获取token

## 2.1. 准备工作

在中国移动开发者社区进行以下操作：

1. 获得appid和appkey；
2. 勾选一键登录能力；
3. 配置应用服务器的出口ip地址

##2.2. 流程说明 

1. 开发者调用SDK取号方法，取号成功后，SDK将缓存取号临时凭证scrip；
2. 用户同意应用获取本机号码，调用授权登录方法，获取接口调用凭证token；
3. 携带token进行接口调用，获取用户的手机号码信息。

一键登录整体流程：

![](image/login_process.png)

##2.3. 预取号

在用户正式授权给开发者使用本机号码登录前，开发者可以提前调取预取号方法，获知当前用户的终端状态和网络环境是否可以从网关获取号码。



为了保证SDK取号成功率，开发者在取号前必须保证：

1. 应用必须提前获取用户手机`READ_PHONE_STATE`权限（用于判断双卡、换卡）。
2. 运营商目前只支持中国移动2/3/4G和中国电信4G，开发者在调用取号方法前，可以预先判断当前用户的运营商类型和网络制式，仅针对SDK目前支持的运营商和网络制式使用一键登录功能。
3. 网关取号必须在数据流量打开的情况下进行，因此，用户如果关闭数据流量，将无法成功取号。（Android SDK 9.0+版本后，对于首次通过数据流量成功登录过的用户，二次+登录时，允许在数据流量关闭时取号）

**请求示例代码**

```java
/***
判断和获取READ_PHONE_STATE权限逻辑
***/   

//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用取号方法
mAuthnHelper.getPhoneInfo(APP_ID, APP_KEY, mListener);
```

**取号方法原型：**

```java
public void getPhoneInfo(final String appId, 
                         final String appKey, 
                         final TokenListener listener)
```

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID，在开发者社区创建应用时获取                      |
| appkey   | String        | 应用密钥，在开发者社区创建应用时获取                         |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段          | 类型   | 含义                          |
| ------------- | ------ | ----------------------------- |
| resultCode    | String | 接口返回码，“103000”为成功。  |
| desc          | String | 成功标识，true为成功。        |
| securityphone | String | 手机号码掩码，如“138XXXX0000” |

## 2.4. 创建授权页（一键登录）

为了确保用户在登录过程中将手机号码信息授权给开发者使用的知情权，一键登录需要开发者提供授权页登录页面供用户授权确认。开发者在调用授权登录方法前，必须弹出授权页，明确告知用户当前操作会将用户的本机号码信息传递给应用。授权页面的设计、布局、生成、弹出和消失，由开发者自行处理，但必须遵守移动认证授权页面设计规范。

注1：如果开发者需要使用**一键登录**服务，必须按照规定创建授权页面

注2：**本机号码校验**不需要创建授权页面，可以直接跳过2.4章

### 2.4.1. 调用逻辑

![](image/authPage.png)

### 2.4.2. 页面规范细则

1、页面必须包含登录/注册按钮，授权登录方法`loginAuth`必须绑定该按钮使用。

2、登录按钮文字描述必须包含“登录”或“注册”等文字，不得诱导用户授权。

3、页面需要提示应用获取到的是用户的本机号码，例如，可以在页面显示本机号码的掩码（securityphone），或者提示用户将使用“本机号码”作为账号登录或注册。

4、页面必须包含移动认证协议条款，其中：

​	条款名称：《中国移动认证服务条款》

​	条款页面地址：http://wap.cmpassport.com/resources/html/contract.html

5、应用在上线前需将满足上述1~4的授权页面（正式上线版的）截图提供给产品接口人审核。

6、应用后续升级时，如果授权页面有较大改动（针对1~4内容进行修改），需将改动的授权页面截图提供给产品接口人审核。

7、对于未遵照1~4设计要求，或通过技术手段故意屏蔽不弹出授权页面但获得调用接口凭证token的行为，移动有权将APP所有一键登录相关的能力下架，待整改后再重新上架服务。

### 2.4.3. 构建授权页面

授权登录页面由开发者设计和构建，在构建前，需调用SDK提供的授权页埋点记录方法。

**埋点记录方法原型**

```java
public void recordAuthEvent(String mAppId)
```

**参数说明**

| 参数   | 类型   | 说明        |
| :----- | :----- | :---------- |
| mAppId | String | 应用的AppID |

## 2.5. 授权

用户调用授权方法，获取取号token

**请求示例代码：**

```java
//1.调用取号方法getPhoneInfo

//2.实现回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};


//3.用户点击授权按钮时，调用loginAuth方法，传入应用定义的uaAuthactivity，获取token
    mAuthnHelper.loginAuth(Constant.APP_ID, Constant.APP_KEY, mListener);
    ……
```

**授权方法原型：**

```java
public void loginAuth(final String appId, 
                      final String appKey, 
                      final TokenListener listener)
```

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID                                                  |
| appkey   | String        | 应用密钥                                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

TokenListener的参数JSONObject，含义如下：

| 字段        | 类型   | 含义                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| resultCode  | String | 接口返回码，“103000”为成功                                   |
| authType    | String | 登录方式：1：WIFI下网关鉴权</br>2：网关鉴权</br>3：其他      |
| authTypeDes | String | 登录方式描述                                                 |
| token       | String | 成功时返回：临时凭证，token有效期2min，一次有效；同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| openId      | String | 成功时返回：用户身份唯一标识。（uaAuthActivity传入null时，不返回） |

## 2.6. 获取手机号码（服务端）

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起获取用户手机号接口的调用。

调用本接口，必须保证：

1. token在有效期内。（2分钟）
2. token还未使用过。
3. 应用服务器出口IP地址在开发者社区中配置正确。
4. 如果使用RSA加密，确保应用的公钥在开发者社区正确填写。

**接口说明：**

请求地址：https://www.cmpassport.com/unisdk/rsapi/loginTokenValidate

协议： HTTPS 

请求方法： POST+json,Content-type设置为application/json

**请求示例代码：**

```json
{
    appid = 10000001;
    msgid = 34a5588136d6404784831609cdcdc633;
    sign = 2240b9213b9b8dccfe7f6257a21071cf;
    strictcheck = 0;
    systemtime = 20180529112443243;
    token = STsid00000015275642798949tUyg6KsmyEWKk005bCfuxUmCXeqeFRK;
    version = "2.0";
}

```

**参数说明**

| 参数                | 约束 | 说明                                                         |
| :------------------ | :--: | :----------------------------------------------------------- |
| version             | 必选 | 填2.0                                                        |
| msgid               | 必选 | 标识请求的随机数即可(1-36位)                                 |
| systemtime          | 必选 | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck         | 必选 | 暂时填写"0"                                                  |
| appid               | 必选 | 业务在统一认证申请的应用id                                   |
| expandparams        | 可选 | 扩展参数                                                     |
| token               | 必选 | 需要解析的凭证值。                                           |
| sign                | 必选 | 当**encryptionalgorithm≠"RSA"**时，sign = MD5（appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名（appid+token，加密方法是SHA256withRSA）, 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm | 可选 | 推荐使用。开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |

**返回说明**

| 参数         | 类型   | 说明                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| inresponseto | string | 对应的请求消息中的msgid                                      |
| systemtime   | string | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode   | string | 返回码                                                       |
| msisdn       | string | 表示手机号码，如果加密方式为RSA，应用需要用私钥进行解密      |

## 2.7. 本机号码校验（服务端）

开发者获取token后，需要将token传递到应用服务器，由应用服务器发起本机号码校验接口的调用。

调用本接口，必须保证：

1. token在有效期内（2分钟）。
2. token还未使用过。
3. 应用服务器出口IP地址在开发者社区中配置正确。

对于本机号码校验，需要注意：

1. 本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。
2. 签订合同后，将不在提供每天免费的测试次数。

**接口说明：**

请求地址： https://www.cmpassport.com/openapi/rs/tokenValidate

协议： HTTPS

请求方法： POST+json,Content-type设置为application/json

**请求示例代码：**

```
{
    body =     {
        openType = 1;
        phoneNum =0A2050AC434A32DE684745C829B3DE570590683FAA1C9374016EF60390E6CE76;
        requesterType = 0;
        sign = 87FCAC97BCF4B0B0D741FE1A85E4DF9603FD301CB3D7100BFB5763CCF61A1488;
        token = STsid0000001517194515125yghlPllAetv4YXx0v6vW2grV1v0votvD;
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        timestamp = 20180129105523519;
        version = "1.0";
    };
}
```

**参数说明：**

| 参数          | 层级  | 约束                         | 说明                                                         |
| ------------- | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    | **1** | 必选                         |                                                              |
| version       | 2     | 必选                         | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | 2     | 必选                         | 使用UUID标识请求的唯一性                                     |
| timestamp     | 2     | 必选                         | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | 2     | 必选                         | 应用ID                                                       |
| **body**      | **1** | 必选                         |                                                              |
| openType      | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | 2     | 是                           | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| phoneNum      | 2     | 是                           | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | 2     | 是                           | 身份标识，字符串形式的token                                  |
| sign          | 2     | 是                           | 签名，HMACSHA256( appId +     msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |

**返回说明：**

| 参数         | 层级  | 类型   | 说明                                                         |
| ------------ | ----- | :----- | :----------------------------------------------------------- |
| **header**   | **1** |        |                                                              |
| msgId        | 2     | string | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | string | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | string | 应用ID                                                       |
| resultCode   | 2     | string | 平台返回码                                                   |
| **body**     | **1** |        |                                                              |
| resultDesc   | 2     | String | 平台返回码                                                   |
| message      | 2     | String | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams | 2     | String | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |



<div STYLE="page-break-after: always;"></div>

# 3. SDK方法说明

## 3.1. 获取管理类的实例对象

### 3.1.1. 方法描述

获取管理类的实例对象

**原型**

```java
public static AuthnHelper getInstance(Context context)
```

### 3.1.2. 参数说明

| 参数      | 类型      | 说明                              |
| ------- | ------- | ------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

## 3.2. 预取号

### 3.2.1. 方法描述

**功能**

本方法用于发起取号请求，并返回用户当前网络环境是否具备取号条件的结果。SDK将在后台完成网络判断、数据网络切换等内部操作并向网关请求申请获取用户本机号码。取号请求成功后，开发者就可以调用并弹出由开发者自定义布局的授权页面。**注意：取号请求前，开发者需提前申请`READ_PHONE_STATE`权限，否则取号会失败！**

**调用建议**

建议开发者在应用开启时调用本方法，有助于提升登录的成功率和用户登录体验。

### 3.2.2. 参数说明

**请求参数**

| 参数       | 类型            | 说明                                       |
| :------- | :------------ | :--------------------------------------- |
| appId    | String        | 应用的AppID                                 |
| appkey   | String        | 应用密钥                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段          | 类型   | 含义                          |
| ------------- | ------ | ----------------------------- |
| resultCode    | String | 接口返回码，“103000”为成功。  |
| desc          | String | 成功标识，true为成功。        |
| securityphone | String | 手机号码掩码，如“138XXXX0000” |

### 3.2.3. 示例

**请求示例代码**

```java
/***
判断和获取READ_PHONE_STATE权限逻辑
***/   

//创建AuthnHelper实例
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }

//实现取号回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};

//调用取号方法
mAuthnHelper.getPhoneInfo(APP_ID, APP_KEY, mListener);
```

**响应示例代码**

```
{
	"resultCode":"103000",
	"desc":"true",
	"securityphone:"138XXXX0000"
}
```

## 3.3. 授权

### 3.3.1. 方法描述

用户调用授权方法，获取取号token

**授权方法原型：**

```java
public void loginAuth(final String appId, 
                      final String appKey, 
                      final TokenListener listener)
```

###3.3.2. 参数说明

**参数说明：**

| 参数     | 类型          | 说明                                                         |
| :------- | :------------ | :----------------------------------------------------------- |
| appId    | String        | 应用的AppID                                                  |
| appkey   | String        | 应用密钥                                                     |
| listener | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |

**返回说明：**

TokenListener的参数JSONObject，含义如下：

| 字段        | 类型   | 含义                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| resultCode  | String | 接口返回码，“103000”为成功                                   |
| authType    | String | 登录方式：1：WIFI下网关鉴权</br>2：网关鉴权</br>3：其他      |
| authTypeDes | String | 登录方式描述                                                 |
| token       | String | 成功时返回：临时凭证，token有效期2min，一次有效；同一用户（手机号）10分钟内获取token且未使用的数量不超过30个 |
| openId      | String | 成功时返回：用户身份唯一标识。（uaAuthActivity传入null时，不返回） |

### 3.3.3. 示例

```java
//1.调用取号方法getPhoneInfo

//2.实现回调
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        …………	// 应用接收到回调后的处理逻辑
    }
};


//3.用户点击授权按钮时，调用loginAuth方法，传入应用定义的uaAuthactivity，获取token
    mAuthnHelper.loginAuth(Constant.APP_ID, Constant.APP_KEY, mListener);
    ……
```

## 3.4. 设置取号超时

###3.4.1. 方法描述

设置取号超时时间，默认为8秒

**原型**

```java
public void setTimeOut(long timeOut)
```

###3.4.2. 参数说明

**请求参数**

| 参数    | 类型 | 说明                       |
| ------- | ---- | -------------------------- |
| timeOut | long | 设置超时时间（单位：毫秒） |

**响应参数**

无



## 3.5. 获取网络状态和运营商类型

### 3.5.1. 方法描述

本方法用于获取用户当前的网络环境和运营商

**原型**

```java
public JSONObject getNetworkType(Context context)
```

### 3.5.2. 参数说明

**请求参数**

| 参数    | 类型    | 说明       |
| ------- | ------- | ---------- |
| context | Context | 上下文对象 |

**响应参数**

参数JSONObject，含义如下：

| 参数     | 类型   | 说明                                                         |
| -------- | ------ | ------------------------------------------------------------ |
| operator | String | 运营商类型：</br>1.移动流量；</br>2.联通流量；</br>3.电信流量 |
| netType  | String | 网络类型：</br>0.未知；</br>1.流量；</br>2.wifi；</br>3.数据流量+wifi |

### 3.5.3. 示例

```java
/**
 * 需要权限：READ_PHONE_STATE， ACCESS_NETWORK_STATE
 * operator获取网络运营商 1.移动流量 2.联通流量网络 3.电信流量网络
 * netType 网络状态：0未知；1流量 2 wifi；3 数据流量+wifi
 */
private void getNetAndOprate(){
    mResultString = mAuthnHelper.getNetworkType(mContext).toString();
    mResultDialog.setResult(StringFormat.logcatFormat(mResultString));
}
```



## 3.6. 删除临时取号凭证

### 3.6.1. 方法描述

开发者调用取号方法`getPhoneInfo`成功后，SDK将取号的一个临时凭证缓存在本地（凭证使用keystore加密），有效期30天。开发者可以根据产品的场景、风险控制级别，确定何时删除凭证。

SDK将在2个地方会更新生成新的凭证，一个是开发者调用取号成功后，会缓存凭证；一个是开发者调用授权方法成功后，会缓存凭证。本地若保存临时凭证，将允许应用在纯wifi下成功获取token，取号时间将缩短，成功率会提升。

**特殊情况：**

基于安全考虑，在两种情况下，凭证将不保存在本地，而只会保存在正在运行的内存中：

1. keystore加密失败；
2. 手机获取了root权限。

这2种情况下，凭证的有效时间将与应用程序内存的生命周期有关。

**原型**

```java
public void delScrip()
```

### 3.6.2. 参数说明

**请求参数**

无

**响应参数**

无

<div STYLE="page-break-after: always;"></div>

# 4. 服务端接口说明

## 4.1. 获取手机号码接口

业务平台或服务端携带用户授权成功后的token来调用认证服务端获取用户手机号码等信息。

### 4.1.1. 接口说明

**请求地址：**https://www.cmpassport.com/unisdk/rsapi/loginTokenValidate

**协议：** HTTPS 

**请求方法：** POST+json,Content-type设置为application/json

**注意：开发者需到开发者社区填写服务端出口IP地址后才能正常使用**

</br>

### 4.1.2. 参数说明

**请求参数**

| 参数                |  类型  | 约束 | 说明                                                         |
| :------------------ | :----: | :--: | :----------------------------------------------------------- |
| version             | string | 必选 | 填2.0                                                        |
| msgid               | string | 必选 | 标识请求的随机数即可(1-36位)                                 |
| systemtime          | string | 必选 | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck         | string | 必选 | 暂时填写"0"                                                  |
| appid               | string | 必选 | 业务在统一认证申请的应用id                                   |
| expandparams        | string | 可选 | 扩展参数                                                     |
| token               | string | 必选 | 需要解析的凭证值。                                           |
| sign                | string | 必选 | 当**encryptionalgorithm≠"RSA"**时，sign = MD5（appid + version + msgid + systemtime + strictcheck + token + appkey)（注：“+”号为合并意思，不包含在被加密的字符串中），输出32位大写字母；</br>当**encryptionalgorithm="RSA"**，业务端RSA私钥签名（appid+token，加密方法是SHA256withRSA）, 服务端使用业务端提供的公钥验证签名（公钥可以在开发者社区配置）。 |
| encryptionalgorithm | string | 可选 | 开发者如果需要使用非对称加密算法时，填写“RSA”。（当该值不设置为“RSA”时，执行MD5签名校验） |

</br>

**响应参数**

| 参数         | 类型   | 约束 | 说明                                                         |
| ------------ | ------ | ---- | ------------------------------------------------------------ |
| inresponseto | string | 必选 | 对应的请求消息中的msgid                                      |
| systemtime   | string | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultcode   | string | 必选 | 返回码                                                       |
| msisdn       | string | 必选 | 表示手机号码，如果加密方式为RSA，应用需要用私钥进行解密      |

</br>

### 4.1.3. 示例

**请求示例**

```
{
    appid = 3000******76; 
    msgid = 335e06a28f064b999d6a25e403991e4c;
    sign = 213EF8D0CC71548945A83166575DFA68;
    strictcheck = 0;
    systemtime = 20180129112955435;
    token = STsid0000001517196594066OHmZvPMBwn2MkFxwvWkV12JixwuZuyDU;
    version = "2.0";
}
```

**响应示例**

```
{
    inresponseto = 335e06a28f064b999d6a25e403991e4c;
    msisdn = 14700000000;
    resultCode = 103000;
    systemtime = 20180129112955477;
}
```

## 4.2. 本机号码校验接口

校验用户输入的号码是否本机号码。
应用将手机号码传给移动认证SDK，移动认证SDK向认证服务端发起本机号码校验请求，认证服务端通过网关获取本机手机号码和第三方应用传输的手机号码进行校验，返回校验结果。

### 4.2.1. 接口说明

**调用次数说明：**本产品属于收费业务，开发者未签订服务合同前，每天总调用次数有限，详情可咨询商务。

**请求地址：** https://www.cmpassport.com/openapi/rs/tokenValidate

**协议：** HTTPS

**请求方法：** POST+json,Content-type设置为application/json

**回调地址：**请参考开发者接入流程文档

### 4.2.2.  参数说明

**请求参数**

| 参数          | 类型   | 层级  | 约束                         | 说明                                                         |
| ------------- | ------ | ----- | ---------------------------- | ------------------------------------------------------------ |
| **header**    |        | **1** | 必选                         |                                                              |
| version       | string | 2     | 必选                         | 版本号,初始版本号1.0,有升级后续调整                          |
| msgId         | string | 2     | 必选                         | 使用UUID标识请求的唯一性                                     |
| timestamp     | string | 2     | 必选                         | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId         | string | 2     | 必选                         | 应用ID                                                       |
| **body**      |        | **1** | 必选                         |                                                              |
| openType      | String | 2     | 否，requestertype字段为0时是 | 运营商类型：</br>1:移动;</br>2:联通;</br>3:电信;</br>0:未知  |
| requesterType | String | 2     | 是                           | 请求方类型：</br>0:APP；</br>1:WAP                           |
| message       | String | 2     | 否                           | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams  | String | 2     | 否                           | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |
| phoneNum      | String | 2     | 是                           | 待校验的手机号码的64位sha256值，字母大写。（手机号码 + appKey + timestamp， “+”号为合并意思）（注：建议开发者对用户输入的手机号码的格式进行校验，增加校验通过的概率） |
| token         | String | 2     | 是                           | 身份标识，字符串形式的token                                  |
| sign          | String | 2     | 是                           | 签名，HMACSHA256( appId +     msgId + phonNum + timestamp + token + version)，输出64位大写字母 （注：“+”号为合并意思，不包含在被加密的字符串中,appkey为秘钥, 参数名做自然排序（Java是用TreeMap进行的自然排序）） |
|               |        |       |                              |                                                              |

**响应参数**

| 参数         | 层级  | 类型   | 约束 | 说明                                                         |
| ------------ | ----- | :----- | :--- | :----------------------------------------------------------- |
| **header**   | **1** |        | 必选 |                                                              |
| msgId        | 2     | string | 必选 | 对应的请求消息中的msgid                                      |
| timestamp    | 2     | string | 必选 | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| appId        | 2     | string | 必选 | 应用ID                                                       |
| resultCode   | 2     | string | 必选 | 平台返回码                                                   |
| **body**     | **1** |        | 必选 |                                                              |
| resultDesc   | 2     | String | 必选 | 平台返回码                                                   |
| message      | 2     | String | 否   | 接入方预留参数，该参数会透传给通知接口，此参数需urlencode编码 |
| expandParams | 2     | String | 否   | 扩展参数格式：param1=value1\|param2=value2  方式传递，参数以竖线 \| 间隔方式传递，此参数需urlencode编码。 |

### 4.2.3. 示例

**请求示例**

```
{
    body =     {
        openType = 1;
        phoneNum =0A2050AC434A32DE684745C829B3DE570590683FAA1C9374016EF60390E6CE76;
        requesterType = 0;
        sign = 87FCAC97BCF4B0B0D741FE1A85E4DF9603FD301CB3D7100BFB5763CCF61A1488;
        token = STsid0000001517194515125yghlPllAetv4YXx0v6vW2grV1v0votvD;
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        timestamp = 20180129105523519;
        version = "1.0";
    };
}

```

**响应示例**

```
{
    body =     {
        message = "";
        resultDesc = "\U662f\U672c\U673a\U53f7\U7801";
    };
    header =     {
        appId = 3000******76;
        msgId = f11585580266414fbde9f755451fb7a7;
        resultCode = 000;
        timestamp = 20180129105523701;
    };
}

```

<div STYLE="page-break-after: always;"></div>



# 5. 返回码说明

##5.1. SDK返回码

使用SDK时，SDK会在认证结束后将结果回调给开发者，其中结果为JSONObject对象，其中resultCode为结果响应码，103000代表成功，其他为失败。成功时在根据token字段取出身份标识。失败时根据resultCode定位失败原因。

| 返回码 | 返回码描述                                     |
| ------ | ---------------------------------------------- |
| 103000 | 成功                                           |
| 102101 | 无网络                                         |
| 102102 | 网络异常                                       |
| 102103 | 未开启数据网络                                 |
| 102121 | 用户取消登录                                   |
| 102203 | 输入参数错误 |
| 102223 | 数据解析异常                                   |
| 102507 | 请求超时 |
| 102508 | 数据网络切换失败                               |
| 200002 | 手机未安装sim卡                                |
| 200005 | 用户未授权（READ_PHONE_STATE）                 |
| 200006 | 用户未授权（SEND_SMS）                         |
| 200007 | authType仅使用短信验证码认证                   |
| 200008 | 1. authType参数为空；2. authType参数不合法；   |
| 200009 | 应用合法性校验失败（包名包签名未填写正确）     |
| 200010 | 预取号时imsi获取失败或者没有sim卡              |
| 200012 | 	取号失败，跳短信验证码登录|
| 200013 | 短信上行发送短信失败（短信上行） |
| 200014| 手机号码格式错误（短验） |
| 200015| 	短信验证码格式错误|
| 200016| 	更新KS失败|
| 200017	|  非移动卡不支持短信上行|
| 200018 | 不支持网关登录|
| 200019 | 不支持短信验证码登录 |
| 200020 | 用户取消登录 |
| 200021 | 数据解析异常（服务器异常可重新尝试） |
| 200022 | 无网络状态 |
| 200023 | 请求超时 |
| 200024 | 数据网络切换失败 |
| 200025 | 未知错误一般出现在线程捕获异常，请配合异常打印分析 |
| 200026 | 输入参数错误 |
| 200027 | 预取号时未开启数据流量 |
| 200028 | 网络请求出错（根据日志分析） |
| 200029 | 请求出错,上次请求未完成 |
| 200030 | 没有初始化参数 |
| 200031 | 生成token失败 |
| 200032 | KS缓存不存在 |
| 200033 | 复用中间件获取Token失败 |
| 200034 | 预取号token失效 |
| 200035 | 协商ks失败 |
| 200036 | 预取号失败 |
| 200037 | 获取不到openid |
| 200038 | 电信重定向失败 |
| 200039 | 电信取号接口返回失败 |
| 200040 | UI资源加载异常 |
| 200042 | 授权页弹出异常 |
## 5.2. 获取手机号码接口返回码

| 返回码 | 返回码描述                                                |
| ------ | --------------------------------------------------------- |
| 103000 | 返回成功                                                  |
| 103101 | 签名错误                                                  |
| 103102 | 包名/包签名/bundle id错误                                 |
| 103106 | 输入手机号不合法（短验）                                  |
| 103107 | 已存在相同的随机数（短验）                                |
| 103108 | 短信验证码错误                                            |
| 103109 | 短信验证码超时                                            |
| 103111 | wap网关IP错误                                             |
| 103112 | 错误的请求                                                |
| 103113 | token校验错误                                             |
| 103119 | appid不合法                                               |
| 103122 | btid不存在                                                |
| 103125 | 手机号格式错误（短验）                                    |
| 103127 | 证书验证：版本过期                                        |
| 103133 | sourceid不合法                                            |
| 103138 | appid已存在                                               |
| 103139 | sourceid已存在                                            |
| 103202 | 缓存用户不存在或者验证短信输入失败次数过多                |
| 103203 | 缓存用户不存在（短信上行）                                |
| 103204 | 缓存随机数不存                                            |
| 103205 | 服务器异常                                                |
| 103207 | 发送短信失败                                              |
| 103211 | 其他错误                                                  |
| 103214 | 访问缓存或数据库错误                                      |
| 103401 | 消息签名为空                                              |
| 103404 | 加密失败                                                  |
| 103408 | 此sourceId注册已达上限99次                                |
| 103409 | query参数为空                                             |
| 103412 | 无效的请求                                                |
| 103414 | 参数效验异常                                              |
| 103505 | 重放攻击                                                  |
| 103511 | 源IP不合法                                                |
| 103810 | token校验失败，接口版本不一致                             |
| 103811 | token为空                                                 |
| 103901 | 短信验证码下发次数已达上限 (一分钟内最多5次,一天最多10次) |
| 103902 | 临时取号凭证scrip校验失败                                 |
| 103911 | 请求过于频繁（短信校验）                                  |
| 103922 | 自动升级文件没找到                                        |
| 104201 | token已经被校验过或者超时失效                             |
| 104202 | 短信验证失败过多                                          |
| 105001 | 联通网关取号失败                                          |
| 105002 | 移动网关取号失败                                          |
| 105003 | 电信网关取号失败                                          |
| 105005 | 短信上行发送信息为空                                      |
| 105007 | 手机号码格式错误（短信上行）                              |
| 105008 | 短信内容为空（短信上行）                                  |
| 105010 | script失效或者非法                                        |
| 105011 | 参数加密的私钥失效或者非法                                |
| 105012 | 不支持电信取号                                            |
| 105013 | 不支持联通取号                                            |
| 105018 | 用户权限不够                                              |
| 105019 | 应用未授权（未勾选能力）                                  |
| 105302 | AppId不在白名单                                           |

## 5.3. 本机号码校验接口返回码

本返回码表仅针对`本机号码校验接口`使用

| 返回码 | 说明                       |
| ------ | -------------------------- |
| 000    | 是本机号码（纳入计费次数） |
| 001    | 非本机号码（纳入计费次数） |
| 002    | 取号失败                   |
| 003    | 调用内部token校验接口失败  |
| 004    | 加密手机号码错误           |
| 102    | 参数无效                   |
| 124    | 白名单校验失败             |
| 302    | sign校验失败               |
| 303    | 参数解析错误               |
| 606    | 验证Token失败              |
| 999    | 系统异常                   |
| 102315 | 次数已用完                 |


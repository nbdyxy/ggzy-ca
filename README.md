
# 广州公共资源交易中心CA集成说明文档

## 目录

* [概述](#summary)
* [接口说明](#api)
* [其他事项](#other)

## <div id='summary'>概述</div>

### 项目背景
广州公共资源交易中心目前已引入4家公司提供CA服务（GDCA、NETCA、共享盾、标信通），3家公司提供签章服务（翔晟签章、共享盾、标信通）。目前，由于缺乏对各家公司提供的调用接口的统一规划，用户在使用系统时需要安装多个驱动且存在浏览器的限制（只能使用IE），体验较差。此外，各公司的调用方式和兼容范围不能做到完全统一也增加了业务系统开发商兼容的难度。为此，基于交易中心CA使用的业务场景，本着各CA使用之间相互独立的原则，集成了GDCA、NETCA和共享盾（暂未实现）的接入，旨在提升用户体验和开发效率。
### 依赖说明
#### 客户端依赖引入
浏览器端引入js脚本调用CA客户端提供的服务
```html
<script language="JavaScript" src="yourpath/jquery-1.11.3.min.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/jquery.jsonp.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/json2.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/gdca.min-1.2.1.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/netca.min-1.2.1.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/gzggzy-integration.min-1.2.1.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/stamper.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/netcaseal.js" type="text/javascript"></script>
<script language="JavaScript" src="yourpath/ebidsun.min-1.2.1.js" type="text/javascript"></script>
```
##### 说明：
1. jquery-1.11.3.min.js，集成脚本基于jquery实现，建议引入1.11以上的版本；
2. jquery.jsonp.js、json2.js是各CA公司用于同驱动服务通讯的工具包；
3. gdca.min-1.2.1.js，兼容GDCA的封装脚本；
4. netca.min-1.2.1.js，兼容NETCA的封装脚本；
5. ebidsun.min-1.2.1.js，兼容标信通的封装脚本（注：目前暂未集成标系统CA相关方法，只集成了标信通签章）；
6. stamper.js，GDCA的签章依赖脚本；
7. netcaseal.js，NETCA的签章依赖脚本；
8. gzggzy-integration.min-1.2.1.js，各CA集成脚本，本文档主要介绍该脚本的接入使用，详见[客户端接口说明](#jsapi)章节。
7. [文件下载链接](https://ca.gzggzy.cn/js.zip)
8. [客户端Demo](http://ca.gzggzy.cn/yyh/gzggzy-integration.html)
9. CA驱动请登录各CA公司官网下载最新版本驱动

##### 兼容
所有常规浏览器以及IE10+

#### 服务端依赖引入
服务端引入jar包调用相关CA服务  
> [jar包下载链接](http://ca.gzggzy.cn/gzggzy-ca-0.0.2.jar)

1. 安装jar包
```
mvn install:install-file -Dfile={your path}\gzggzy-ca-0.0.2.jar -DgroupId=cn.gzggzy -DartifactId=gzggzy-ca -Dversion=0.0.2 -Dpackaging=jar
```
2. 引入工程
```xml
<dependency>
    <groupId>cn.gzggzy</groupId>
    <artifactId>gzggzy-ca</artifactId>
    <version>0.0.2</version>
</dependency>
```
3. [服务端接口说明](#javaapi)
4. [服务端Demo](http://ca.gzggzy.cn/gzggzy-ca-example/access/integration.do)
5. [DEMO源代码地址](https://gitlab.gzggzy.cn/yuyh/gzggzy-ca-example)
> 请使用中心网络并配置内部DNS或者在本机hosts（C:\Windows\System32\drivers\etc\hosts）添加如下配置访问代码仓库
```
10.197.39.141 gitlab.gzggzy.cn
```
6. [配置文件](http://ca.gzggzy.cn/ca_config.properties)
> 该配置文件需放在服务端src目录下，暂不支持修改文件名称
7. NETCA服务器端配置
> Windows服务器
>> 把[NetcaJcrypto.dll](http://ca.gzggzy.cn/NetcaJCrypto.dll)放在java.library.path目录下面，通常放到 %JAVA_HOME%/bin 目录下。

> Linux服务器
>> 请根据[安装手册](http://ca.gzggzy.cn/安装手册.doc)安装[NETCA_CRYPTO(linux32&64)](http://ca.gzggzy.cn/NETCA_CRYPTO(linux32&64).zip)

#### API接口方式
服务器端除提供Jar包方式外，也提供了API接口的调用方式，开发商接入只需调用相关接口，不需要对服务器做额外的配置以及维护CRL列表
> 需在中心中台内查看接口说明及申请调用权限


## <div id='api'>接口说明</div>

### <div id='jsapi'>客户端接口说明</div>

客户端把参数和方法统一封装在对象中，该对象固定接收window.localStorage作为入参，全局变量存储在localStorage中。引入js以后通过以下方式即可实现调用：
```js
var gzggzy = new Gzggzy(window.localStorage);
gzggzy.login(pin)
    .done(function(response) {
        alert(response.msg);
    })
    .fail(function(response) {
        alert(response.msg);
    });
```
#### 关于$.Deferred()和response对象
1. 大部分客户端方法借助jquery $.Deferred()对象实现了异步回调，并把结果封装在response对象中作为deferred回调的传参。

> done(function(response)用于执行方法调用成功的回调

> fail(function(response)用于执行方法调用失败的回调

> always(function(response)用于执行无论方法是否调用成功的回调

例如上述对登录方法login的调用，登录成功时执行done内的回调，登录失败时执行fail内的回调。

2. response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行成功，false：执行失败
msg | 执行成功时反馈相应的CA或签章信息，执行失败时返回失败的原因



#### <div id='jsapi-list'>API概览</div>

接口名称 | 接口功能
---|---
[login](#login) | 用户登录（仅限于客户端）
[checkDevice](#checkDevice) | 根据localStorage获取登录设备的CA类型
[getCachePassword](#getCachePassword) | 获取当前登录设备的密码
[getCert](#getCert) | 获取证书信息
[getCertInfo](#getCertInfo) | 获取证书基本信息（公共资源定制）
[p1Sign](#p1Sign) | p1签名
[p1Verify](#p1Verify) | p1验签
[p7Sign](#p7Sign) | p7签名
[p7Verify](#p7Verify) | p7验签
[envelopeEncrypt](#envelopeEncrypt) | 数字信封加密
[envelopeDecrypt](#envelopeDecrypt) | 数字信封解密(带原文)
[envelopeDecryptWithoutPlain](#envelopeDecryptWithoutPlain) | 数字信封解密(不带原文)
[signSealWithPositionAndUpload](#signSealWithPositionAndUpload) | 坐标定位签章并上传文件
[signSealWithPositionAndReturnB64](#signSealWithPositionAndReturnB64) | 坐标定位签章并返回盖章后的B64文件
[signTextSealWithPositionAndReturnB64](#signTextSealWithPositionAndReturnB64) | 坐标定位签文本章并返回盖章后的B64文件
[signForGPOB64](#signForGPOB64) | 医药耗材定制签章方法

#### <div id='login'><h3 style='color: red;'>login</h3></div>

1. 方法描述
> 用户登录

2. 输入参数
> 无
 
3. 输出参数
> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：登录成功，false：登录失败
msg | 登录成功时返回登录成功，登录失败时返回失败的原因

#### <div id='checkDevice'><h3 style='color: red;'>checkDevice</h3></div>

1. 方法描述
> 根据localStorage获取登录设备的CA类型

2. 输入参数
> 无
 
3. 输出参数
> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：登录成功，false：登录失败
msg | 执行成功时返回设备类型，执行失败时返回失败的原因

#### <div id='getCachePassword'><h3 style='color: red;'>getCachePassword</h3></div>

1. 方法描述
> 获取当前登录设备的密码

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
ebidsun | 否 | string | 是否为标信通用户，标信通用户传入 EBIDSU，并跳过获取密码的环节
 
3. 输出参数
> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：登录成功，false：登录失败
msg | 执行成功时返回设备密码，执行失败时返回失败的原因
ebidsun | 扩展字段，暂无实际意义

#### <div id='getCert'><h3 style='color: red;'>getCert</h3></div>

1. 方法描述

> 获取base64编码后的证书

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
type | 是 | int | 证书类型，1---签名证书  2---加密证书
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回证书base64编码，执行失败时返回失败的原因

#### <div id='getCertInfo'><h3 style='color: red;'>getCertInfo</h3></div>

1. 方法描述

> 获取证书基本信息（公共资源定制）

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回certInfo对象，执行失败时返回失败的原因

> certInfo对象

参数名称 | 描述
---|---
trustId | 信任服务号
serialNumber | 序列号
subjectCn | 证书持有人
validFromDate | 有效期(始)
validToDate | 有效期(终)
caType |  证书类型: JG---机构、YW---业务、ZRR---自然人

#### <div id='p1Sign'><h3 style='color: red;'>p1Sign</h3></div>

1. 方法描述

> p1签名，首先检查设备的登录状态，若已登录则直接签名，若未登录会弹出登录框先登录再签名

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
signPlainData | 是 | string | 签名原文
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回p1签名的base64编码，执行失败时返回失败的原因

#### <div id='p1Verify'><h3 style='color: red;'>p1Verify</h3></div>

1. 方法描述

> p1验签，因各CA验签方法不同，要求调用方必须传入CA的类型

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
signCert | 是 | string | 签名证书
signPlainData | 是 | string | 签名原文
signedData | 是 | string | 签名数据
cadeviceType | 是 | string | CA类型 'GDCA'或'NETCA'
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回验证成功，执行失败时返回失败的原因

#### <div id='p7Sign'><h3 style='color: red;'>p7Sign</h3></div>

1. 方法描述

> p7签名，首先检查设备的登录状态，若已登录则直接签名，若未登录会弹出登录框先登录再签名

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
Signmsg_plain_data | 是 | string | 消息原文
Sign_flag | 是 | int | 0-带原文 1-不带原文
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回p7签名的base64编码，执行失败时返回失败的原因

#### <div id='p7Verify'><h3 style='color: red;'>p7Verify</h3></div>

1. 方法描述

> p7验签，因各CA验签方法不同，要求调用方必须传入CA的类型

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
Signed_msg | 是 | string | 签名数据
signPlainData | 是 | string | 签名原文
Sign_flag | 是 | int | 0-带原文 1-不带原文
cadeviceType | 是 | string | CA类型 'GDCA'或'NETCA'
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回验证成功，执行失败时返回失败的原因

#### <div id='envelopeEncrypt'><h3 style='color: red;'>envelopeEncrypt</h3></div>

1. 方法描述

> 数字信封加密

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
encryptPlainData | 是 | string | 加密原文
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回加密的base64编码，执行失败时返回失败的原因

#### <div id='envelopeDecrypt'><h3 style='color: red;'>envelopeDecrypt</h3></div>

1. 方法描述

> 数字信封解密，需传入加密原文。首先检查设备的登录状态，若已登录则直接解密，若未登录会弹出登录框先登录再解密

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
encryptPlainData | 是 | string | 加密原文
encryptedData | 是 | string | 加密密文
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回解密的原文，执行失败时返回失败的原因

#### <div id='envelopeDecryptWithoutPlain'><h3 style='color: red;'>envelopeDecryptWithoutPlain</h3></div>

1. 方法描述

> 数字信封解密，无需传入加密原文。首先检查设备的登录状态，若已登录则直接解密，若未登录会弹出登录框先登录再解密

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
encryptedData | 是 | string | 加密密文
timeout | 否 | int | 方法调用超时时间，单位ms，默认为20000ms

3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回解密的原文，执行失败时返回失败的原因

#### <div id='signSealWithPositionAndUpload'><h3 style='color: red;'>signSealWithPositionAndUpload</h3></div>

1. 方法描述

> 坐标定位签章并上传文件，插入GDCA、NETCA的情况下方法默认加盖CA内的公章，标信通签章由用户自行选择。签章前会首先检验设备的登录状态，若已登录则直接签章，若未登录会弹出登录框先登录再签章。

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
ebidsun | 是 | string | 是否为标信通用户，标信通用户传入 EBIDSU，否则传入''
signPdf | 是 | string | PDF文件Base64编码
uploadUrl | 是 | string | 上传URL
pageNum | 是 | int | 签章页码（-1表示最后一页）
x | 是 | int | 相对于起点的X轴偏移量(单位px)，左上角为起点
y | 是 | int | 相对于起点的y轴偏移量(单位px)，左上角为起点
pin | 否 | string | 证书密码
width | 否 | int | 签章图片的宽度 NETCA签章时，该参数有效
height | 否 | int | 签章图片的高度 NETCA签章时，该参数有效


3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回上传URL返回的执行结果，执行失败时返回失败的原因

#### <div id='signSealWithPositionAndReturnB64'><h3 style='color: red;'>signSealWithPositionAndReturnB64</h3></div>

1. 方法描述

> 坐标定位签章并返回盖章后的Base64文件，插入GDCA、NETCA的情况下方法默认加盖CA内的公章，标信通签章由用户自行选择。签章前会首先检验设备的登录状态，若已登录则直接签章，若未登录会弹出登录框先登录再签章。

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
ebidsun | 是 | string | 是否为标信通用户，标信通用户传入 EBIDSU，否则传入''
signPdf | 是 | string | PDF文件Base64编码
pageNum | 是 | int | 签章页码（-1表示最后一页）
x | 是 | int | 相对于起点的X轴偏移量(单位px)，左上角为起点
y | 是 | int | 相对于起点的y轴偏移量(单位px)，左上角为起点
pin | 否 | string | 证书密码
width | 否 | int | 签章图片的宽度 NETCA签章时，该参数有效
height | 否 | int | 签章图片的高度 NETCA签章时，该参数有效


3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回盖章后的Base64文件，执行失败时返回失败的原因


#### <div id='signTextSealWithPositionAndReturnB64'><h3 style='color: red;'>signTextSealWithPositionAndReturnB64</h3></div>

1. 方法描述

> 坐标定位签文本章并返回盖章后的B64文件，插入GDCA、NETCA的情况下方法默认加盖CA内的公章，标信通签章由用户自行选择。签章前会首先检验设备的登录状态，若已登录则直接签章，若未登录会弹出登录框先登录再签章。文本章为根据CA证书持有人生成的签章图片。

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
ebidsun | 是 | string | 是否为标信通用户，标信通用户传入 EBIDSU，否则传入''
signPdf | 是 | string | PDF文件Base64编码
pageNum | 是 | int | 签章页码（-1表示最后一页）
x | 是 | int | 相对于起点的X轴偏移量(单位px)，左上角为起点
y | 是 | int | 相对于起点的y轴偏移量(单位px)，左上角为起点
pin | 否 | string | 证书密码


3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回盖章后的Base64文件，执行失败时返回失败的原因

#### <div id='signForGPOB64'><h3 style='color: red;'>signForGPOB64</h3></div>

1. 方法描述

> 医药耗材定制签章方法，插入GDCA、NETCA的情况下方法默认加盖CA内的公章，标信通签章由用户自行选择。签章前会首先检验设备的登录状态，若已登录则直接签章，若未登录会弹出登录框先登录再签章。

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
ebidsun | 是 | string | 是否为标信通用户，标信通用户传入 EBIDSU，否则传入''
signPdf | 是 | string | PDF文件Base64编码
pin | 是 | string | 传入''时，则默认获取登录设备的缓存密码
num | 是 | int | 签章个数 1---签单位公章，2---单位公章加文本章
s_pageNum | 是 | int | 单位公章盖章页码
s_x | 否 | int | 单位公章相对于起点的X轴偏移量(单位px)，左上角为起点
s_x | 否 | int | 单位公章相对于起点的y轴偏移量(单位px)，左上角为起点
width | 否 | int | 单位公章图片宽度 NETCA签章时，该参数有效
height | 否 | int | 单位公章图片高度 NETCA签章时，该参数有效
t_pageNum | 否 | int | 单位文本章盖章页码
t_x | 否 | int | 单位文本章相对于起点的X轴偏移量(单位px)，左上角为起点
t_y | 否 | int | 单位文本章相对于起点的Y轴偏移量(单位px)，左上角为起点


3. 输出参数

> response对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行录成功，false：执行失败
msg | 执行成功时返回盖章后的Base64文件，执行失败时返回失败的原因

### <div id='javaapi'><h2 style='color: red;'>服务端接口说明</h2></div>

1. CAResponse对象

服务器端接口统一返回CAResponse对象

参数名称 | 描述
---|---
status | 方法执行情况，true：执行成功，false：执行失败
message | 执行成功时返回相应的结果，执行失败时返回失败的原因

#### API概览

接口名称 | 接口功能
---|---
[getRandom](#getRandom) | 服务器端生成随机数 
[verfiyCert](#verfiyCert) | 校验证书的有效性
[verfiySign](#verfiySign) | 验证签名
[getCertInfo](#getCertInfo) | 获取证书基本信息（公共资源定制）

#### <div id='getRandom'><h3 style='color: red;'>getRandom</h3></div>

1. 方法描述

> 服务器端生成随机数 

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
length | 是 | int | 随机数长度

3. 输出参数

> CAResponse对象

#### <div id='verfiyCert'><h3 style='color: red;'>verfiyCert</h3></div>

1. 方法描述

> 校验证书的有效性 

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
cert | 是 | String | 证书的Base64编码

3. 输出参数

> CAResponse对象

#### <div id='verfiySign'><h3 style='color: red;'>verfiySign</h3></div>

1. 方法描述

> 验证签名

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
cert | 是 | String | 证书的Base64编码
sourceValue | 是 | String | 签名原文
signedValue | 是 | String | 签名后的值
sOption | 是 | SignatureOption | 采用的签名方式，具体参看SignatureOption对象说明

> SignatureOption对象说明

签名方法枚举对象
```java
SignatureOption.P1    //P1签名方法
SignatureOption.P7    //P7签名方法
```

3. 输出参数

> CAResponse对象

#### <div id='getCertInfo'><h3 style='color: red;'>getCertInfo</h3></div>

1. 方法描述

> 获取证书属性信息  

2. 输入参数

参数名称 | 必选 | 类型 | 描述
---|---|---|---
cert | 是 | String | 证书的Base64编码
certAttributeOption | 是 | CertAttributeOption | 采用的签名方式，具体参看CertAttributeOption对象说明

> CertAttributeOption对象说明

签名方法枚举对象
```java
CertAttributeOption.SN    //证书序列号
CertAttributeOption.VALID_DATE_FROM    //证书有效期开始时间
CertAttributeOption.VALID_DATE_TO    //证书有效期开始时间
CertAttributeOption.CN    //证书持有人名称
CertAttributeOption.ZZJGDM    //组织机构代码
CertAttributeOption.TRUSTID    //信任服务号
```

3. 输出参数

> CAResponse对象



## <div id='other'><h2 style='color: red;'>其他事项</h2></div>
暂无

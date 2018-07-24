# 东海游戏SDK服务端说明文档


##	接口说明
###	1、用户会话验证

* 1) **请求地址：** https://api.sdk.dhios.cn/open/verifyAccessToken
* 2) **调用方式：** HTTPS Post
* 3) **接口描述：**
验证 accessToken 是否为有效的登录用户会话，若有效则返回其 userId。**“游戏客户端”**通过“SDK 客户端”获取到accessToken，传到**“游戏服务器”**，**“游戏服务器”**到**“SDK 服务器”**验证用户会话accessToken的有效性，获取用户的userId，供游戏使用。</br>**注意：进行接口调用前请确认accessToken是否具备值，如accessToken值为空时请勿调用此接口。**
* 4) **请求方：** 游戏服务器
* 5) **响应方：** SDK 服务器
* 6) **请求内容：** application/x-www-urlencoded格式
* 7）**错误码：   0      正常 
		3001   签名错误 
		3002   token失效
		5000   服务端内部数据错误
* 8）**KEY：运营提供的apiKey


<table>
    <thead>
        <tr>
            <th>字段名称</th>
            <th>字段说明</th>
            <th>类型</th>
            <th>必填</th>
            <th>备注</th> 
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>gameId</td>
            <td>东海运营提供(游戏Id)</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
		<tr>
            <td>subGameId</td>
            <td>东海运营提供(游戏子包Id)</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>accessToken</td>
            <td>accessToken</td>
            <td>string</td>
            <td>是</td>
            <td>accessToken令牌</td>
        </tr>
        <tr>
            <td>sign</td>
            <td>签名参数</td>
            <td>string</td>
            <td>是</td>
            <td>所有参数key(不包括sign)按照A-Z字段升序（去除&符号，key与value用=连接）直接拼接apikey的值之后,md5运算</td>
        </tr>
    </tbody>
</table>

请求例子：   
```php


对参数名按字母升序得到字符串
已排序（值根据文档要求类型填写，示例仅参考）

1. gameId = 1
2. subGameId = 2
3. accessToken = abcdefD

apikey：ABCD(东海提供)

拼接示例：
string=“accessToken=abcdefDgameId=1subGameId=2ABCD"
sign = MD5(string)

结果例如
sign："0554a2df922c7ae7b3a111d8c8f6ebfc"
 
 
Public function notify(){
    $params['gameId'] = 1;//东海运营提供
    $params['subGameId']=1;//东海运营提供
    $params['accessToken'] = 'tokendemo';
    ksort($params);
    $key = 'api_key';//东海运营提供
    $str = str_replace("&","",http_build_query($params).$key);//去除&，升序拼接成：accessToken=accessakkgameId=1subGameId=1api_key
    
    $params['sign'] = md5($str);
    
    $res = curl_https_post（$url,$params） ;//发起https post请求，不解析get请求
    //处理$res数据
}

返回json
{
    "code": 0,
    "msg": "ok",
    "data": {
        "userId": 8,
        "userName": "zxwzxw"
    }
}
```


###	2、支付结果异步通知
* 1）**请求地址：** 由游戏cp方提供
* 2）**调用方式：** HTTP Post
* 3）**接口描述：**
**“游戏客户端”**购买成功后，**“SDK 服务器”**通过该接口通知**”游戏服务器“**。支付成功一定会及时回调通知，支付失败可能不会及时通知（如支付超时等无法及时通知），如果需要知道订单状态，使用支付结果查询结构。
支付通知有失败重试机制，正式环境通知时间间隔为15,15,30,180,1800,1800,1800,1800,3600 单位为秒<br>
<br>支付回调通知，还有一点，可能会由于升级增加参数，需要验证回调签名时，应该允许这种情况。</br>
使用支付key签名；
* 4）**请求方：** SDK 服务器  
* 5）**响应方：** 游戏服务器 
* 6）**请求内容：** application/x-www-urlencoded格式
* 7）**KEY：运营提供的payKey
* 8）**注意：请验证发起调用的服务器ip地址，ip地址会在参数里给到
<table>
    <thead>
        <tr>
            <th>字段名称</th>
            <th>字段说明</th>
            <th>类型</th>
            <th>必填</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>cpOrderId</td>
            <td>CP方订单</td>
            <td>String</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>userId</td>
            <td>用户Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>orderId</td>
            <td>东海平台订单</td>
            <td>String</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>gameId</td>
            <td>游戏Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
	<tr>
            <td>subGameId</td>
            <td>子包Id</td>
            <td>int</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>platform</td>
            <td>平台类型</td>
            <td>string</td>
            <td>是</td>
            <td>1，iOS；2，android</td>
        </tr>
        <tr>
            <td>totalFee</td>
            <td>支付金额</td>
            <td>int</td>
            <td>是</td>
            <td>单位是：分</td>
        </tr>
        <tr>
            <td>orderStatus</td>
            <td>订单状态</td>
            <td>int</td>
            <td>是</td>
            <td>1：已支付，0：未支付</td>
        </tr>
	<tr>
            <td>endtime</td>
            <td>订单完成时间</td>
            <td>int</td>
            <td>是</td>
            <td>UNIX时间戳</td>
        </tr>
	<tr>
            <td>randStr</td>
            <td>随机字符串</td>
            <td>String</td>
            <td>是</td>
            <td>不长于32位</td>
        </tr>
	<tr>
            <td>customInfo</td>
            <td>用户自定义数据</td>
            <td>String</td>
            <td>是</td>
            <td></td>
        </tr>
        <tr>
            <td>sign</td>
            <td>所有参数key(不包括sign)按照A-Z字段升序（去除&符号，key与value用=连接）直接拼接apikey的值之后,md5运算</td>
            <td>string</td>
            <td>是</td>
            <td></td>
        </tr>
    </tbody>
</table>


```php


已排序（参数值根据文档要求类型填写，示例仅参考）

1. cpOrderId = 1   
2. customInfo=2 
3. endtime = 3
4. gameId = 4
5. orderId = 5
6. orderStatus = 6
7. platform = 7
8. randStr = 8
9. subGameId = 9
10. totalFee = 10
11. userId = 11

apikey：ABCD (东海提供)

拼接示例：
string=“cpOrderId=1customInfo=2endtime=3gameId=4orderId=5orderStatus=6platform=7randStr=8subGameId=9totalFee=10userId=11ABCD”

sign = MD5(string)

结果例如:
sign："0554a2df922c7ae7b3a111d8c8f6ebfc"

Public function notify(){
    $params = I('post.');
    unset($params['sign']);
    
    $sign = $_POST['sign'];

    ksort($params);
    $key = 'key';//由东海颁发使用支付key
    $sign = md5(urldecode(str_replace("&","",http_build_query($params).$key)));
    if($sign==$_POST['sign']){
    	//商户信息，订单信息验证
	$success = ...;
	if($success){
        	echo 'success';
	}else{
		echo 'failure';
	}
        //CP方数据处理
    }else{
    	echo 'failure';
    }
}
```

返回内容（string）：
<table>
    <thead>
        <tr>
            <th>输出值</th>
            <th>备注</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>success</td>
            <td>成功,表示游戏服务器成功接收了该次充值结果通知</td>
        </tr>
        <tr>
            <td>failure</td>
            <td>失败,表示游戏服务器无法接收或识别该次充值结果通知,如:签名检验不正确、游戏服务器接收失败</td>
        </tr>
    </tbody>
</table>
	
	
	

许可证
==============
DHSDK 使用 MIT 许可证，详情见 LICENSE 文件。


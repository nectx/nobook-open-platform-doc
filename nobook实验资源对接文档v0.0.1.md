# 一、免登录地址接口

------

## 1、接口定义

主要是为第三方用户对接使用 用户登录NOBOOK虚拟实验平台，需要开发者根据用户信息在服务端生成一个免登录url，通过这个url链接，NOBOOK才能知道是哪个用户来访问。 为了确保客户端每次请求到都是最新的免登陆url，客户端每次向服务器发的请求不能是固定的，以避免请求被缓存。 NOBOOK虚拟实验免登录url经过签名，该url地址5分钟失效，请务必在生成地址后立即使用，使用后页面会重定向进入NOBOOK。

同理

开发者服务器端需要开发一个支持重定向的接口实现动态生成免登录url地址，该接口地址配置在客户端，用户通过点击该地址访问应用。

功能说明：
> * 第三方有独立的用户系统
> * 第三方用户在自己平台登录，调用应用接口，直接访问应用
> * 创建应用时需要应用有免密登录权限才能对接
> * 需要根据免密地址需要传递必要的参数请看下方介绍



## 2. 免登录URL生成

### 请求说明
请求方式：get
编码说明：UTF-8
请求URL：https://res-api.nobook.com/api/login/autologin?appid=123&uid=456&timestamp=1501112321&sign=fdasfdDS93ASF8&redirect=https%3a%2f%2fwww.nobook.com%2f

### 2.1 参数说明

| 参数        | 是否必须   |  参数类型  | 限制长度 |参数说明|
| --------    | :-----     | :----     | :----   | :----|
| uid    | 必须     | str     | 32   | 用户唯一性标识，对应唯一一个用户且不可变|
| subject    | 必须     | str     | 32   | 学科标识（phy：物理；chem：化学；biocz：初中生物；biogz：高中生物；sci：小学科学）|
| appid |必须 | str|  16   | 接口appid，应用的唯一标识|
| timestamp|必须| str|255   |1970-01-01开始的时间戳，秒为单位|
| redirect |必须 | str|  255  | 登录成功后的重定向地址,必须进行URL编码|
| sign |必须 | str|  255   | MD5签名 (除redirect参数外将所有的参数值与appkey按参数名升序进行排列）
|


### 2.2 签名生成规则
md5签名的原理如下： 将所有的参数值与appkey按参数名升序进行排列。
Md5(value1+value2+...appkey...+valueN)
appkey在签名中的顺序取决于他在所有参数名中的顺序。

用例：

1. 参数：
uid：1
appid：123456 
timestamp:1523865261
appkey:testappkey
subject:phy
redirect:https%3a%2f%2fwww.nobook.com%2f


2. 参数进行升序排列后生成的签名原串：
123456testappkeyphy15238652611
3. 签名后字符串 : d9adc82b45df90d95df293004ad130b4

4. 签名url ：https://res-api.nobook.com/api/login/autologin?appid=123456&uid=1&timestamp=1523865261&sign=768f8d587d6c605d7a8c1f7b0641e349&redirect=https%3a%2f%2fwww.nobook.com%2f

---

## 3.响应说明

失败 {code: 500, data: "", msg: "错误信息"}

成功 免密登录成功，跳到回调地址redirect



# 二、获取实验列表

------
### 请求说明
请求方式：get
编码说明：UTF-8
请求URL：https://res-api.nobook.com/api/experiment/get?appid=123&subject=phy&timestamp=1501112321&sign=fdasfdDS93ASF8

### 参数说明

| 参数        | 是否必须   |  参数类型  | 限制长度 |参数说明|
| --------    | :-----     | :----     | :----   | :----|
| subject    | 必须     | str     | 32   | 学科标识（phy：物理；chem：化学；biocz：初中生物；biogz：高中生物；sci：小学科学）|
| appid |必须 | str|  16   | 接口appid，应用的唯一标识|
| timestamp|必须| str|255   |1970-01-01开始的时间戳，秒为单位|
| sign |必须 | str|  255   | MD5签名（将所有的参数值与appkey按参数名升序进行排列）|



## 响应说明

失败 {code: 500, data: "", msg: "错误信息"}

成功 {code: 200, data:_data,  msg: "ok"}

###_data参数说明
```json
[
    {
        "iconPath":"实验缩略图",
        "name": "实验名称",
        "url": "实验访问地址"
    },
]    
    
```

## 3、phpDemo
```<?php

//配置参数
$appid = '';
$appkey = '';
$subject = 'phy';
$timestamp = time();
$uid = '';
$redirect = urlencode('https://res-api.nobook.com/wuli/?sourceid=452385a5699233a32bc4c6b292800752');


function  sign($array)
{
    ksort($array);
    $string="";
    while (list($key, $val) = each($array)){
        $string = $string . $val ;
    }
    return md5($string);
}


//获取实验列表URL
function getListUrl($subject, $appid, $appkey, $timestamp)
{
    $arr = [
        'subject'=> $subject,
        'appid'=> $appid,
        'appkey'=> $appkey,
        'timestamp'=> $timestamp,
    ];

    $sign = sign($arr);
    $param = [
        'timestamp'=> $timestamp,
        'subject'=> $subject,
        'appid'=> $appid,
        'sign'=> $sign,
    ];

    $url = 'https://res-api.nobook.com/api/experiment/get?'.http_build_query($param);

    return $url;
}


//获取登录Url
function getLoginUrl($uid, $subject, $appid, $timestamp, $appkey)
{
    $arr = [
        'uid'=> $uid,
        'subject'=> $subject,
        'appid'=> $appid,
        'timestamp'=> $timestamp,
        'appkey'=> $appkey,
    ];
    $sign = sign($arr);
    $param = [
        'timestamp'=> $timestamp,
        'uid'=> $uid,
        'subject'=> $subject,
        'appid'=> $appid,
        'sign'=> $sign,
        'redirect'=> 'https://res-api.nobook.com/wuli/?sourceid=452385a5699233a32bc4c6b292800752',
    ];

    $url = 'https://res-api.nobook.com/api/login/autologin?'.http_build_query($param);

    return $url;


}

$getListUrl = getListUrl($subject, $appid, $appkey, $timestamp);

$getLoginUrl = getLoginUrl($uid, $subject, $appid, $timestamp, $appkey);

?>


<h4>实验列表URl</h4>


<p> <a target="_blank" href="<?=$getListUrl?>"><?=$getListUrl?></a> </p>

<h4>登录URl</h4>


<p> <a target="_blank" href="<?=$getLoginUrl?>"><?=$getLoginUrl?></a> </p>
```






---
title: Data Center Doc
layout: index
format: markdown
slug: index.html
---

# 全局字段声明

由于众所周知的原因，暂时不支持通过IP地址请求，需要配置Host才能访问：

    172.17.6.127  video.game.yy.com

测试机IP：172.17.6.127(内网)、113.106.251.82(外网)

返回结果格式统一为：

    {
        "status": 200,  // 业务处理的状态，200为处理成功的统一标识，其它数值按照不同接口有不同定义
        "message": "",  // 业务处理结果消息，只有在处理失败时才需要填充错误信息，处理成功留空白
        "data": {       // 业务返回数据
            ...
        }
    }


# 通过手机号获取验证码

url：http://video.game.yy.com/register/getPhoneCode.do?phone=[PhoneNum]

## 参数：

| ----- |:-----:| -----:|
| `String PhoneNum` 11位的手机号码 |

## 请求示例：
### http://video.game.yy.com/register/getPhoneCode.do?phone=15920871245

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": {
            "phoneValidId": 120
        }
    }

无论这个手机号是否已经注册，都要下行验证码短信，在校验验证码合法性并注册或登录时会处理手机号存在的情况。



# 校验验证码的合法性

url：http://video.game.yy.com/register/checkPhoneCode.do?phoneValidId=[ValidId]&phoneValidCode=[ValidCode]&phone=[PhoneNum]

## 参数：

| ----- |:-----:| -----:|
| `int ValidId` 验证码在服务端的唯一ID |
| `String ValidCode` 六位随机数的验证码 |
| `String PhoneNum` 11位的手机号码 |

## 请求示例：
### http://video.game.yy.com/register/checkPhoneCode.do?phoneValidId=120&phoneValidCode=198382&phone=15920871245

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": null
    }



# 通过手机号码注册时设置密码并登录

url：http://video.game.yy.com/register/register.do?phoneValidId=[ValidId]&phone=[PhoneNum]&psw=[Passwd]

## 参数：

| ----- |:-----:| -----:|
| `int ValidId` 验证码在服务端的唯一ID |
| `String PhoneNum` 11位的手机号码 |
| `String Passwd` 使用MD5加密的密码，由服务端确定算法 |

## 请求示例：
### http://video.game.yy.com/register/register.do?phoneValidId=120&phone=15920871245&psw=SFLKDFGJAKLSDFKNSSDJFGASFLK

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": {
            "id": 10942,
            "nickName": "胡尼克扬1892",
            "location": "广东 珠海",
            "head_url": "http://video.game.yy.com/head/198591.png", // 为空代表使用默认头像
        }
    }

返回鉴别用户是否登录的Header：

    memberId：10000035
    UDBSESSIONID：adfcaee9-68d0-46fa-9906-119bff8f394c

在手机号码注册时，前面已经对验证码进行了校验，所以这个接口只提交验证码的唯一标识，服务端通过这个唯一标识去查找对应的手机号，
看参数中的手机号跟对应的手机是否匹配，匹配则为再次鉴权成功，执行注册。最后在响应Json中返回用户的个人信息，Header中返回登录的cookie。

手机号已注册的情况在这个接口是需要处理的，如果手机号码已经注册，则将这次注册操作视为重置密码的操作，
所以服务端遇到这种情况时应该使用提交的新密码替换旧密码，然后返回登录信息。



# 使用新浪微博进行注册登录

url：http://video.game.yy.com/register/registerByWeibo.do?uid=[Uid]&accessToken=[AccessToken]&name=[Name]&location=[Location]&headUrl=[HeadUrl]

## 参数：

| ----- |:-----:| -----:|
| `String Uid` 新浪微博的账号唯一标识 |
| `String AccessToken` 访问微博账号所使用的鉴权标识 |
| `String Name` 用户昵称 |
| `String Location` 用户的所在位置 |
| `String HeadUrl` 用户的头像地址 |

## 请求示例：
### http://video.game.yy.com/register/registerByWeibo.do?uid=1768721164&access_token=2.00ys3hvBosNWuB4fd0fea8bflE2caC&name=WoWoWo&location=广东广州&headUrl=http://tp1.sinaimg.cn/1768721164/50/40043799298/1

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": {
            "id": 10942,
            "nickName": "胡尼克扬1892",
            "location": "广东 珠海",
            "head_url": "http://video.game.yy.com/head/198591.png", // 为空代表使用默认头像
        }
    }

返回鉴别用户是否登录的Header：

    memberId：10000035
    UDBSESSIONID：adfcaee9-68d0-46fa-9906-119bff8f394c

服务端首先使用提交的微博UID跟AccessToken去请求微博开放平台验证这个用户是否已经授权，访问地址为：

    https://api.weibo.com/2/users/show.json?uid=[Uid]&accessToken=[AccessToken]

如果返回正确的结果，则判断这个UID是否已经注册，如果没有就执行注册，最后在响应Json中返回用户的个人信息，Header中返回登录的cookie。



# 使用手机号进行登录

url：http://video.game.yy.com/login/login.do?phone=[PhoneNum]&psw=[Passwd]

## 参数：

| ----- |:-----:| -----:|
| `String PhoneNum` 11位的手机号码 |
| `String Passwd` 使用MD5加密的密码，由服务端确定算法 |

## 请求示例：
### http://video.game.yy.com/login/login.do?phoneNum=15929281952&psw=LKSFDADFLAGJSLDFAD

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": {
            "id": 10942,
            "nickName": "胡尼克扬1892",
            "location": "广东 珠海",
            "head_url": "http://video.game.yy.com/head/198591.png", // 为空代表使用默认头像
        }
    }

返回鉴别用户是否登录的Header：

    memberId：10000035
    UDBSESSIONID：adfcaee9-68d0-46fa-9906-119bff8f394c



# 获取用户信息

url：http://video.game.yy.com/user/getUserInfo.do?memberId=[MemberId]

## 参数：

| ----- |:-----:| -----:|
| `Param int MemberId` 用户唯一标识 |
| `Header int memberId` Header鉴权的用户唯一标识 |
| `Header String UDBSESSIONID` Header鉴权的用户会话信息 |

## 请求示例：
### http://video.game.yy.com/user/getUserInfo.do?memberId=12984

Header：

memberId：10000035

UDBSESSIONID：adfcaee9-68d0-46fa-9906-119bff8f394c

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": {
            "id": 10942,
            "nickName": "胡尼克扬1892",
            "location": "广东 珠海",
            "head_url": "http://video.game.yy.com/head/198591.png", // 为空代表使用默认头像
        }
    }

这个接口首先鉴别客户端提交的两个Header是否合法，再返回memberId对应用户的信息。



# 退出登录

url：http://video.game.yy.com/user/loginOut.do?memberId=[MemberId]

## 参数：

| ----- |:-----:| -----:|
| `int MemberId` 用户唯一标识 |

## 请求示例：
### http://video.game.yy.com/user/loginOut.do?memberId=1894

## 返回结果：

    {
        "status": 200,
        "message": "",
        "data": null
    }


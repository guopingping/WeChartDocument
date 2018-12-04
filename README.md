微信公众号开发文档
=================================================================================

```
1. 微信公众平台：
    概述：运营者通过公众号为微信用户提供资讯和服务的平台。一对多  
        公众平台开发接口则是提供服务的基础。

    开发者在公众平台网创建公众号--获取接口权限；
       
    OpenID用来识别用户，每个用户针对每个公众号产生一个OpenID；
        多公众号、移动应用之间做用户共通，需前往微信开发平台，将这些公众号和应用绑定到一个开发平台账号下，
    绑定后，一个用户虽然对多个功能公众号和应用有多个不同OpenID，但对这些同一开放平台账号下的公众号和
    应用，只有一个UnionID

    订阅号：针对个人或媒体每天可以群发1条消息，默认不具有自定义菜单；
    服务器号：针对企业或银行每月可以群发4条信息，默认具有自定义菜单；
    运营主体是组织（企业、媒体、公益组织）的，可以申请服务号；
    运营主体是组织和个人的可以申请订阅号，但个人不能申请服务号；

2. 公众平台的两种模式：
    编辑模式：直接使用微信公众平台所提供的后台操作进行用户交互。
            应用场景：不具备开发能力的运营者。
    开发者模式：直接使用接口代码实现用户的交流。

3. 公众号消息会话：
    群发消息：以一定频次向用户群发消息（订阅号每天1次，服务号每月次）
    被动回复消息：用户向公众号发消息
    客服消息：不限量的消息
    模板消息：对用户发送服务通知

4. 公众号内网页：
    网页授权获取用户基本信息：通过接口，可以获取用户的基本信息
    微信JS-SDK：开发者在网页上通过JavaScript代码使用微信原生功能的工具包。录制播放微信语音、监听微信分享、上传手机本地图片、拍照等。

5. 开发者规范
    
    appID:第三方用户调用唯一凭证；
    appsecret:第三方永不唯一凭证密钥；
    grant_type:获取accent_token填写client_credential
    URL:
        一旦提交微信会给URL发送一个get请求，验证接口的有效性。
    nonce：随机数
    echostr：随机字符串
    timestamp：时间戳
    signature：微信加密签名，
        签名生成：
            需要timestamp,token,nonce(随机字符串)；
            将三个字段排序，进行sha1加密；
            加密字符串与signature对比，标识请求来源于微信；
            ```
                const crypto=require('crypto')// 加密模块
                module.exports={
                    authentication:function(weixinData){
                        const token=weixinData.token
                        const signature=weixinData.signature
                        const timestamp=weixinData.timestamp
                        const nonce=weixinData.nonce

                        //排序
                        const sortArr=[token,timestamp,nonce]
                        sortArr.sort() 
                        
                        //转换为字符串
                        const sortArrStr=sortArr.join("")
                        
                        //加密
                        const hashCode=crypto.createHash('sha1')//加密类型
                        const result=hashCode.update(sortArrStr,'utf-8').digest('hex')

                        return result===signature
                    }
                }

            ```

    access_token:
        公众号的全局唯一接口调用凭据。
        公众号和小程序均可以使用AppID和AppSecret调用本接口来获取access_token;
        调用接口时，请登录“微信公众平台-开发-基本配置”提前将服务器IP地址添加到IP白名单中，点击查看设置方法，否则将无法调用成功。小程序无需配置IP白名单;

        存储512MB 有效期2小时 定时刷新
        有效期：expires_in

        get
        https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET

    token：
        验证流程：
            自己的服务器和微信服务器进行绑定，提交的url为token验证的接口，
            微信服务器会把echostr发到自己的服务器，然后我们的服务器进行token验证，看是否匹配。
            匹配后，将echostr返回给微信服务器，服务器即绑定成功。
            自己的服务器和微信服务器的消息手法都是通过验证token那个接口实现的。

    AccessToken中控服务器：
        提供主动刷新和被动刷新机制来刷新accessToken并存储，提供给业务逻辑有效的accessToken；
        避免业务逻辑并发获取access_token，互相覆盖；
    
    API-Proxy服务器：
        转一与微信API对接

    服务器保证5秒内处理回复：success或“”

    http:// https:// 开头 ，支持80端口和443端口

```
```
JS-SDK
    微信公众平台面向网页开发者提供的基于微信内的网页开发工具包。

    使用步骤：
        1、绑定域名；
        2、引入JS文件；
            http://res.wx.qq.com/open/js/jweixin-1.4.0.js或http://res2.wx.qq.com/open/js/jweixin-1.4.0.js

        3、通过config接口注入权限验证配置；
        4、通过ready接口处理成功验证；
        5、通过error接口处理失败验证；
        
```
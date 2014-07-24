# OAuth2 Implicit Grant（简化授权）授权模式

## 适用范围
主要用于无服务器端的应用。涉及的角色包括：用户（Resource Owner）、UA（User Agent）、授权服务器、资源服务器。


## 工作流程

* A. Client（第三方应用）将用户导向认证服务器
* B. 用户决定是否给客户端授权
* C. 如果用户同意给客户端授权，则认证服务器将用户重定向到Client指定的redirect_uri，并在URI的Hash部分包含访问令牌
* D. UA通过JavaScript解析URI的Hash中的访问令牌信息
* E. UA将解析得到的访问令牌发给Client
* F. UA或Client利用访问令牌去资源服务器取资源

注意令牌对用户和浏览器是可见的。

下面是API请求过程（example.com是OAuth服务端）：

```shell
# authorize
curl https://open.example.com/authorize?\
client_id=98b698eb7bc14259b35fc337a1c6a0eb\
&response_type=token\
&redirect_uri=http://client.com/cb\
&scope=user_r,user_w
&state=abc
```
注意几条：
* client_id、response_type和redirect_uri是必选参数
* scope和state都是可选参数
* response_type为固定值token
* 如果传了state参数，则下一步返回必须原封不动带上state参数

成功授权后，跳回redirect_uri所定义的网址：

```
HTTP/1.1 302 Found
Location: http://client.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
          &state=abc&token_type=example&expires_in=3600
```

然后浏览器可以提取上面Location地址中的Hash值得到访问令牌，再通过访问令牌访问资源服务器上的资源。当然还可以把访问令牌发回给Client Server然后存起来，这样避免后续再去授权。不过这没有什么意义，因为通过简化授权模式得到的访问令牌的有效期一般都很短，比如30分钟。

相比授权码模式，少了一个获取Authorization Code（即授权码）的流程，而且不会返回refresh_token。所以这种模式的安全性比较低。安全性低，所以不应该给访问令牌设置过长的有效期。

另外为了安全性考虑，应用从授权回调地址接收到访问令牌后，可以调用下get_token_info接口，然后确认是否对应自身的appKey。
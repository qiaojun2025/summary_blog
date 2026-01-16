## 什么是JWT
[官网](https://www.jwt.io/introduction#what-is-json-web-token)<br>
JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

Although JWTs can be encrypted to also provide secrecy between parties, we will focus on signed tokens. Signed tokens can verify the integrity of the claims contained within it, while encrypted tokens hide those claims from other parties. When tokens are signed using public/private key pairs, the signature also certifies that only the party holding the private key is the one that signed it.
## JWT能做什么
Here are some scenarios where JSON Web Tokens are useful:

- Authorization: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays because of its small overhead and its ability to be easily used across different domains.<br>
- Information Exchange: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.



## 为什么是JWT
### 基于传统的session认证
- 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大
- 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。
- 因为是基于cookie来进行用户识别的，cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。
- 在前后端分离系统中就更加痛苦:如下图所示也就是说前后端分离在应用解耦后增加了部署的复杂性。通常用户一次请求就要转发多次。如果用session 每次携带sessionid 到服务器，服务器还要查询用户信息。同时如果用户很多。这些信息存储在服务器内存中，给服务器增加负担。还有就是CSRF(跨站伪造请求攻击)攻击，session是基于cookie进行用户识别的，cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。还有就是sessionid就是一个特征值，表达的信息不够丰富。不容易扩展。而且如果你后端应用是多节点部署。那么就需要实现session共享机制。不方便集群应用。


### 基于JWT认证
![截图2](pic/jwt-check.png)
1. 认证流程

	- 前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP POST请求。建议的方式是通过SSL加密的传输(https协议)，从而避免敏感信息被嗅探。
	- 后端核对用户名和密码成功后，将用户的id等其他信息作为JWT Payload(负载)，将其与头部分别进行Base64编码拼接后签名，形成3-个JWT(Token)。形成的JWT就是一个形同111,zzz,xxx的字符串。 token head,payload.singurater
	- 后端将JWT字符串作为登录成功的返回结果返回给前端。前端可以将返回的结果保存在1ocalStorage或sessionstorage上，退出登录时前端删除保存的JWT即可。
	- 前端在每次请求时将JWT放入HTTP Header中的Authorization位。(解决XSS和XSRF问题)
	- 后端检查是否存在，如存在验证JWT的有效性。例如，检查签名是否正确;检查Token是否过期;检查Token的接收方是否是自己
	- 验证通过后后端使用JWT中包浛的用户信息进行其他逻辑操作，返回相应结果。
2. JWT优势

	- 简洁(Compact): 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快
	- 自包含(Self-contained):负载中包含了所有用户所需要的信息，避免了多次查询数据库
	- 因为Token是以JSON加密的形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持，
	- 不需要在服务端保存会话信息，特别适用于分布式微服务。

## JWT结构
1. JWT令牌组成
	- 标头(Header)
	- 有效载荷(Payload)
	- 签名(Signature)

	因此，JWT通常如下所示:xxxxx.yyyyy.zzzzzHeader.Payload.Signature

2. Header
Header通常由两部分组成:令牌的类型(即JWT)和所使用的签名算法，例如HMAC SHA256或RSA。它会使用 Base64 编码组成 JWT结构的第一部分。
```
{
	"alg": "HS256",
	"typ": "JWT"
}
```

3. Payload
令牌的第二部分是有效负载，其中包含声明。声明是有关实体(通常是用户)和其他数据的声明。同样的，它会使用 Base64 编码组成JWT 结构的第二部分
```
{
	"sub": "123456",
	"name": "Leo",
	"admin": true
}
```

4. Signature
前面两部分都是使用 Base64 进行编码的，即前端可以解开知道里面的信息。Signature 需要使用编码后的 header 和 payload以及我们提供的一个密钥，然后使用 header 中指定的签名算法(HS256)进行签名。签名的作用是保证 JWT 没有被篡改过,
如：HMACSHA256(base64UrlEncode(header)+","+ base64UrlEncode(payload),secret)

	最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

	在这里大家一定会问一个问题:Base64是一种编码，是可逆的，那么我的信息不就被暴露了吗?

	是的。所以，在JWT中，不应该在负载里面加入任何敏感的数据。在上面的例子中，我们传输的是用户的User ID。这个值实际上不是什么敏 感内容，一般情况下被知道也是安全的。但是像密码这样的内容就不能被放在JWT中了。如果将用户的密码放在了JWT中，那么怀有恶意的第 三方通过Base64解码就能很快地知道你的密码了。因此JWT适合用于向Web应用传递一些非敏感信息。JWT还经常用于设计用户认证捡磯郁ニ基爬授燠釕茳蜃研系，统，甚至实现Web应用的单点登录。
	
![截图3](pic/jwt-structure.png)

5. 放在一起
- 输出是三个由点分隔的Base64-URL字符串，可以在HTML和HTTP环境中轻松传递这些字符串，与基于XML的标准(例如SAML)相比，它更紧凑。
- 简洁(Compact), 可以通过URL，POST 参数或者在 HTTP header 发送，因为数据量小，传输速度快
- 自包含(Self-contained), 负载中包含了所有用户所需要的信息，避免了多次查询数据库
> eyJhbGci0iJIUzI1NiISInR5cCI6IkpXVCN9.
eyJzdWIi0iIxMiMBNTY30DkwIiwibmFtZSI6IkpvaG4
gRG9lIiwiaXNTb2NpYWwi0nRydwv9.
4pcPyMD89olPSyXnrxCiTwXyr4BsezdI1AVTmud2fu4

## 使用JWT
**引入依赖**
```
	<dependency>
		<groupId>com.auth0</groupId>
		<artifactId>java-jwt</artifactId>
		<version>3.4.0</version>
	</dependency>
```


**生成token**
```java
Calendar instance=Calendar.getInstance();
instance.add(Calendar.SECOND，90);
//生成令牌
String token =JWT.create()
.withClaim("username"，"张三")//设置自定义用户名
.withExpiresAt(instance.getTime())//设置过期时间
.sign(Algorithm.HMAC256("token!Q2W#ESRW"));//设置签名 保密 复杂//输出令牌
System.out.println(token):

```

**根据令牌和签名解析数据**
```java
JWTVerifier jwtVerifier = JwT.require(Algorithm.HMAC256("token!Q2W#ESRW")).build();
DecodedJWT decodedJWT =iwtVerifier.verify(token);
System.out.println("用户名:“+ decodedJWT.getClaim("username").asString());
System.out.println("过期时间:"+decodedJwT.getExpiresAt());
```

**常见异常**
```
SignatureVerificationException: 签名不一致异常
TokenExpiredException: 令牌过期异常
AlgorithmMismatchException: 算法不匹配异常
InvalidClaimException: 失效的payload异常
```


## 工具类

```
/**
 * 生成token 	header.payload.sing
 */
public static string getToken(Map<string,string> map){
	Calendar instance=Calendar.getInstance()iinstance.add(Calendar.DarE，amount:7);//默认7天过期
	//创建jwt builder
	JWTCreator.Builder builder=JWT.create();
	//payload
	map.forEach((k,v)->{
		builder.withClaim(k,v);
	}):
	String token= builder.withExpiresAt(instance.getTime())//指定令牌过期时间
					.sign(Algorithm.HMAC256(SING))://sign
	return token;
}

/**
 * 验证token 合法性
 */
public static DecodedJwT verify(String token){
	return JWT.require(Algorithm.HMAC256(SING)).build().verify(token);
}
```
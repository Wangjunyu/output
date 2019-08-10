Title: Neomutt Gmail oauth 尝试
Date: 2019-08-08
Category: neomutt

写在前面：
本以为这是一次普通的尝试，结果掉入了一个深坑...而且还经历了一次和项目发起人的邮件，看到了一些无奈和精神。

Neomutt 在其官方文档中[Optional Features](https://neomutt.org/guide/optionalfeatures)有明确的提到“Preliminary OAUTH support for IMAP, POP, and SMTP is provided via external scripts.”于是自己兴冲冲的开始了相关的配置工作。在调通了代码之后（gmail 授权的坑最后说），发现了一个匪夷所思的报错:`imap_oauth_refresh_command: unknown variable`

我一脸的黑问号...难道是我的xxx又错了？经过了将近一个小时的排查，确认内容无误。心中不免有一丝最不该有的疑问：难道项目出错了？很快我就打消了这种疑虑，我想到了网上的经典笑话，如果你问一个开发：你写的代码是不是有问题，很容易被怼回来，是你不会用吧。如果你问开发：是不是我哪里用的不对？开发容易陷入：我擦，是不是我哪里写错了疑虑...

到现在都记得当时已经是将近午夜12点，答应自己要早睡的我，决定在用户组里发封邮件问一下。之后，实在是不甘心，抱着试试看的态度，下载了源码，跳转到对应的tag下，grep了一下配置中的参数...心中一万只羊驼奔腾起来....竟然没有对应的代码，把分支转到最新，有了...所以，我的第一反应这是一个bug。

之后我又把上游的项目mutt的源码下下来检查了一遍，发现mutt中2018年11月就更新了这个feature。而现在neomutt的源码是2018年7月的稳定版本，谜团逐渐揭开...正好最近业余时间把mutt的英文manual看了一遍（以后会写博客来说这个内容），仔细比对了neomutt和mutt的文档，neomutt的文档是基于mutt写的，和代码一样，增加了一部分自己的feature的内容。

现在问题都清楚了，neomutt的项目是基于mutt做的功能增强，但是开发者的速度已经跟不上mutt的发布速度。mutt在2019年7月还有版本发布，但是neomutt一直停留在2018年7月。然而...neomutt的文档不知为何却跟上了mutt的更新节奏，所以mutt更新的功能在neomutt中并未体现，但是却在neomutt的文档里体现了出来...而这么巧的是，都让我碰上了。

本来执着在命令行的人就不多，用到mutt、neomutt的人就更少了，而会折腾google近期更新（最近3年）的授权功能的人...全世界也没几个人，所以网上几乎没有相关的讨论或者资料...

我发的两封邮件得到的回复如下：
```
> Is it possible for developer release the new neomutt version asap?

The short answer is, "No."

If you want to use the development version, you'll have to build your own NeoMutt (and I don't have the time to help individuals do that).

I'm trying to fix twenty years' worth of poor design in Mutt and it's taking me longer than I'd expected.  After three years of hard work on NeoMutt, I'm still pretty much the only developer.

Cheers,
    Rich / FlatCap
```

```
Hi guys:

I am trying to change my gmail imap password to oauth method.And I
followed the  official guide (
https://neomutt.org/guide/optionalfeatures ) but there is an error
when I start the neomutt :

Error in /home/leon/.neomuttrc, line 94: imap_oauth_refresh_command:
unknown variable
source: errors in /home/leon/.neomuttrc


Then I checked the code on github, at latest released version "NeoMutt
20180716" there is no code include  "imap_oauth_refresh_command" , I
search by "grep -r imap_oauth_refresh_command ." but at the latest
master branch it exists.

And in https://neomutt.org/guide/optionalfeatures#6-%C2%A0oauthbearer-support
there is no warning about the function is not release yet.

Is "Optional Features" on official website default under development?
should we give some warning about this? or is it a bug?

--------
Leon

I got the reason:

in mutt repo updating list the feature mentioned above is updated and
released in 2018-11-25. neomutt latest release is 20180716.
BUT neomutt official website followed mutt repo and also publish
introduction about oauth.

Is it possible for developer release the new neomutt version asap?

https://gitlab.com/muttmua/mutt/raw/stable/UPDATING

1.11.0 (2018-11-25):

  + inotify is used for local mailbox monitoring on Linux.  Configuration flag
    --disable-filemonitor turns this off.
  + OAUTHBEARER support for IMAP, SMTP and POP via
    $imap_oauth_refresh_command, $smtp_oauth_refresh_command, and
    $pop_oauth_refresh_command.
```

就像Rich/FlatCap说的那样,我做了开发版本的编译，于是有了[编译部署案例-mutt开发版本](https://tech.junyu.io/bian-yi-bu-shu-an-li-muttkai-fa-ban-ben.html)这篇博客.

I'm trying to fix twenty years' worth of poor design in Mutt and it's taking me longer than I'd expected.  After three years of hard work on NeoMutt, I'm still pretty much the only developer.

通过作者的这段回复，能感受到一丢丢的怨气，相对小众的开源项目能够坚持持续优化的确是一件很辛苦的事情。作者没有弃坑，还在继续开发，精神可嘉，这也符合开源社群的精神：谁主张，谁负责。

为了支持作者，也为了让后来者少踩一些坑，我准备把之前提到的那篇博客翻译成英文发到社区里面。

## 其他的坑
除了这个小插曲之外，其实在配置授权的过程中还有许多的坑，这里简单记录，供参考。

### Gmail的联通性
ss不支持imap协议，（也可能是我的配置问题，求教大神）,所以建议的方案是类似采用前置机的方案，在可以访问gmail的地方租一台服务器，通过服务器访问。毕竟是命令行工具，好处就是普遍性比较好，不需要图形化桌面，而且ssh访问流量也少。基于imap协议不用在本地存储，足够日常的基本使用。

### 获取对应的id文件
首先你需要在下面的地址做一些基本的配置，并获取 client_id client_secret的内容
https://console.developers.google.com/apis/credentials

### python2 or python3
google api提供的代码都是基于python2的内容。如果你有自己写代码实现功能的需要，有可能会迁移到python3，有一些常见的报错。 因为报错都非常常见，这里仅记录，如果需要详细含义的可以网上搜索一下，不再解释。（这里忽略了print需要加括号这个错误）

* AttributeError: module 'urllib' has no attribute 'urlopen'
	urllib.urlopen -> urllib.request.urlopen
	import urllib  -> import urllib.request

* AttributeError: module 'urllib' has no attribute 'urlencode'
  	urllib.urlencode() -> urllib.parse.urlencode()

* TypeError: POST data should be bytes, an iterable of bytes, or a file object. It cannot be of type str.
  	urllib.parse.urlencode(d).encode("utf-8") 增加encode的内容

### oauth2.py 的应用
`urllib.error.HTTPError: HTTP Error 400: Bad Request`这个报错的原因有可能是token有错误,这里需要注意的是refresh_token初次生成的方式不同
  	需要执行oauth2.py的第一种情况获取refresh的token，再执行带有refresh的参数

### 编译后的报错

* 编译之后在配置文件中，除了官方文档的配置外，别忘了`set imap_user = "your_mail_name@gmail.com"`，否则每次登陆都会提示你输入。
* 另外还遇到了一个问题:Permisson Denied.最后发现是对应的oauth2.py文件的执行权限没有给。配置一下权限即可。
* 另一个隐藏的坑点是 python2 或者 python3，一般默认的系统都是python2，建议自己都跑一下，以防万一

### 需要看邮件
最开始尝试和gmail的连接时，最好把gmail的网页版开着，如果检查了发现找不到问题的时候，可以去网页看看，有可能是gmail的安全机制block了访问，你需要手动在邮件里点击确认一下即可。

## 其他方案

https://gitlab.com/muttmua/mutt/wikis/UseCases/Gmail
gmail传统的配置方案

直接把自己的重要的邮箱密码暴露在前置机的配置文件里是非常不安全的做法，为了解决这个问题也并非没有其他选项。比如利用gpg的认证机制。事实上很多人在oauth的认证之前都是这么做的。
这个方案，看起来也要折腾一下，如果用了gpg认证，肯定不可能仅仅用在验证邮箱密码上，很有可能会需要实验一下：
* 用于收发安全邮件
* 用于github的联通加密
* 去公钥服务器看一看
* 其他可能的应用场景

所以...又得折腾好久，以后再写...

## Debug 思路

其实在这个常识过程还是有点曲折，特别是最后一步的权限问题，我一直怀疑是哪里的配置有问题。还打开了neomutt的debug mode查看日志。对系统工程师来说，日志是非常重要的内容，帮助定位和排查问题。

不过最后的问题是更容易出现的文件权限的问题。

## Reference
* https://github.com/google/gmail-oauth2-tools/blob/master/python/oauth2.py
* [凭据 - neomutt - Google API 控制台](https://console.developers.google.com/apis/credentials)
* [Debugging NeoMutt - NeoMutt](https://neomutt.org/dev/debug)
* [Optional Features - NeoMutt](https://neomutt.org/guide/optionalfeatures.html)
* [Add OAuth2 support for gmail! · Issue #1362 · neomutt/neomutt](https://github.com/neomutt/neomutt/issues/1362)

* https://gitlab.com/muttmua/mutt/raw/stable/UPDATING mutt中在2018-11中更新了相关的功能和对应的文档
* https://marc.info/?t=153893576200027&r=1&w=2 这是mutt 邮件组的讨论

## ChangeLog

* 2019-08-08 新建文件
* blog、github、quip、du、ow 发布
* LeonTalk 备份（后续版本以wiki的更新为准）

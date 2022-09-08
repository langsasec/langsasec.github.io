## 地级市HVV|SQL注入分享

### 前言

> 本文对地级市HVV中SQL注入导致连锁反应的两个案例进行分享

### 案例一

#### 某公司SQL注入到进入后台

##### 挖掘过程

资产里找到了一个汽车检测公司的报告查询系统。

![image-20220901151844201](https://i0.hdslb.com/bfs/album/83946093884124e2877020d2f9c9aac6078fe1eb.png)

看到了参数key，直接sqlmap一把梭，SQL注入，呵呵😂

![image-20220901152119326](https://i0.hdslb.com/bfs/album/61a1fb3c38f4fb5153bb196b876561d57119fcc7.png)

![image-20220901152808296](https://i0.hdslb.com/bfs/album/b3cd46bf30d63b8a940b51dd2bec9b293bbd0e0b.png)

![image-20220901152829896](https://i0.hdslb.com/bfs/album/5af1e0b7844ebf941604fb2cb826977b692870eb.png)

当天基本拿下这个站数据库的所有数据了，就直接提交了，尝试上传shell发现物理路径找不到，且MSSQL我也传不了马，后台也没有找到，就放弃找其他点了。

最后一天的时候，没有其他思路就想着再看看这个站，又想起来数据库名字好像jz_开头。

**极致CMS**!!!!!![image-20220901153310155](https://i0.hdslb.com/bfs/album/5f99a1db9260ae3a719c1156b8918cc15fd9e332.png)

![image-20220901152330743](https://i0.hdslb.com/bfs/album/61c46594d3ef420116df0ab2d63449a3c349e00b.png)

拼接后成功访问！

![image-20220901153511128](https://i0.hdslb.com/bfs/album/833b832f55e1fd097cce3e84260aeaca31c625a3.png)

然后在之前SQL注入的库里面找后台用户名密码，找了几个特征表里都没有，就想直接--dump，锁屏让sqlmap跑，大概吃完饭回来，自动停在了一个地方，而这个地方刚好是一个邮箱＋密码。难道它知道我想要什么？

![image-20220901153950917](https://i0.hdslb.com/bfs/album/1f55ca7b21b261e23415a35cb3098ef9ad001037.png)

输入账户密码，登录成功！

![image-20220901154205218](https://i0.hdslb.com/bfs/album/af1d325159e7ce1653c3033284274da2da0fada2.png)

后续对极致cms的测试不再继续

### 案例二

#### 某大数据公司漏洞真多

##### 挖掘过程

开局一个登录框，弱口令admin/123456，我差点没笑死，竟然登进去了，不过进不去也不会出现在文章里。

![image-20220901154615419](https://i0.hdslb.com/bfs/album/4930d229df66c3e82ef470841a3fc3a0b0aee62c.png)

![image-20220901154814136](https://i0.hdslb.com/bfs/album/95860543116a90f5e73807973b43ccd6f2fc14eb.png)

通过上传发现接口太严了，没办法，换思路。

ThinkPHP，搞起来

![image-20220901155031747](https://i0.hdslb.com/bfs/album/9099a0bb65e4b603f13185f8fc347a88cb192c23.png)

先是队友发现了日志泄露

> 日志泄露：https://blog.csdn.net/weixin_40412037/article/details/113885372

日志泄露不要紧，你这SQL语句摆这里，那就得SQL注入了啊

![image-20220901155321026](https://i0.hdslb.com/bfs/album/0dfd119505a2b44df2c0fa1429283361b28d7ce8.png)

开始懒得没抓包，直接sqlmap用--forms，发现不可以。

抓包发现登录认证靠这个接口，当登录成功后为1，失败为0。

![image-20220901155521652](https://i0.hdslb.com/bfs/album/e7ddfc91cfdd1f998078573eddf7ab4d33d5eee5.png)

![image-20220901155642518](https://i0.hdslb.com/bfs/album/40d26d5302200ff297f3145e6e9ea1a5e0f1f7cf.png)

直接sqlmap跑包吧，寻思啥呢

![image-20220901160507071](https://i0.hdslb.com/bfs/album/ff96f9d7f649f5980152a22249a1652d38710ba4.png)

成功跑出来

![image-20220901160551959](https://i0.hdslb.com/bfs/album/bd94a2f9654fe8a44f025e6b48ccbdbd0fc6e88c.png)

-sql-shell可以且为DBA权限，但无法上传shell，因为当时分已经够了并且自己太懒，就没继续挖。

ThinkPHP应该还有好多可以拿分的点。

![image-20220901160903864](https://i0.hdslb.com/bfs/album/7f183ace12a52034b43c5e6a04f219c591e3b907.png)

### 总结

本文分享了两个案例：一个是前台SQL注入拿到后台，一个是弱口令到框架的日志泄露再回到后台登陆发现SQL注入。思路较为常规，希望对初学者有帮助，重点还是要在信息收集方面下功夫，认真全面才会发现更多的漏洞。
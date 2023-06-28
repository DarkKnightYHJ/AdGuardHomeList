# 序言
在当今越来越数字化的社会中，广告和追踪器成为了许多用户最不喜欢的东西之一。用户访问网站或使用应用程序时，经常会被恼人的广告和追踪器所干扰，甚至可能泄露个人隐私。这就是为什么广告拦截 / 反追踪插件变得越发受欢迎的原因。这些插件利用拦截规则过滤和隐身模式等技术，可以有效地去除不必要的广告和跟踪行为，帮助用户保护个人隐私和网络安全。

AdGuard Home是一款功能丰富的广告拦截软件，它可以防止网站上的广告、跟踪脚本和恶意软件等，同时提供全局DNS解析服务，保护用户的隐私和网络安全。与其他广告拦截 / 反追踪插件不同的是，AdGuard Home可以运行在网络上的任何设备上，为所有连接到网络的设备提供保护。在本篇文章中，我们将介绍如何安装和使用AdGuard Home，以便用户更好地保护自己的网络安全。
# 部署
AdGuard Home部署方法有很多种，其中二进制文件部署和直接编译到OpenWrt系统中是常用的方式。不过，笔者更喜欢使用Docker部署，因为它的更新和维护成本低，具有多样性和灵活性。比如，对于需要进行DNS分流的科学上网需求，使用Docker可以进行多容器部署和管理。以下主要介绍两种Docker网络模式下的部署：
## 部署准备

```markup
#拉取 AdGuard Home Docker镜像
docker pull adguard/adguardhome

#设置 AdGuard Home 的配置文件存储位置
mkdir /etc/AdGuard_Home/
```
## 放行端口

 - 53：DNS 端口。即其他设备访问 AdGuard Home 进行 DNS 解析的默认端口。因为部分系统不支持自定义
   DNS端口，所以不建议自定义。部署前务必要查看是否有其它程序占用。 
   
 - 67, 68： DHCP 端口。除非想代替你路由上的 DHCP服务器，否则用不到。
 - 80: 管理页面默认 HTTP 端口。可忽略，在初始化页面设置管理端口为 3000 端口即可。
 - 443：HTTPS 和 DoH 端口。本地内网环境不需要。
 - 853：DoT 端口。不使用相关功能可忽略。
 - 3000：初始化设置端口。除非通过配置文件去设置，否则必须开启。
```markup
#放行TCP
TCP:53,67,68,853,3000,80,443
#放行UDP
UDP:53,67,68,853,3000,80,443
```


 
 
 ### 1. Bridge网络模式
 Bridge网络模式是Docker默认的网络模式，然而，采用Bridge网络模式部署时存在端口设置、占用和修改困难等问题，限制多且灵活度较差。另外，AdGuard Home在Bridge网络模式下每个端口都有各自的作用，对用户来说也需要了解清楚。
示例：
```bash
 docker run \
--name AdGuard_Home \
-v /etc/AdGuard_Home/:/opt/adguardhome/work \
-v /etc/AdGuard_Home/:/opt/adguardhome/conf \
-p 53:53/tcp  \
-p 53:53/udp \ 
-p 67:67/udp \
-p 68:68/tcp \ 
-p 68:68/udp  \
-p 80:80/tcp \
-p 443:443/tcp \ 
-p 853:853/tcp \ 
-p 3000:3000/tcp \
--restart=always \
-d adguard/adguardhome
```
从示例中可以得知 Ad­Guard Home 所需要用到的端口（已在上文详细解释），但实际情况并不是都会用到，这需要根据自身的需求来决定。如果只是本地局域网使用一般只需要映射53和3000端口（在初始化页面设置管理端口为 3000 端口即可）
```bash
docker run \
	--name AdGuard_Home \
	-v /etc/AdGuard_Home/:/opt/adguardhome/work \
	-v /etc/AdGuard_Home/:/opt/adguardhome/conf \
	-p 53:53/tcp  \
	-p 53:53/udp \ 
	-p 3000:3000/tcp \
	--restart=always \
	-d adguard/adguardhome
```
### 2. Host网络模式
Host网络模式直接使用宿主机的网络，不存在容器端口映射和隔离等问题。容器启动后可以根据需要灵活自如地调整端口，适用于本机(localhost)使用，或直接向外部设备开放服务，如VPS或主路由等。Host网络模式的特性使得它适合在需要直通外网设备的情况下使用。

```bash
docker run -d \
    --name adguardhome \
    --restart=always \
    --network host \
    -v /etc/AdGuard_Home/:/opt/adguardhome/work \
	-v /etc/AdGuard_Home/:/opt/adguardhome/conf \
    adguard/adguardhome
```
## 初始化设置

 1. 进入安装向导

	在浏览器中打开 AdGuard Home 的后台，进入安装向导，点击 “开始配置”。默认后台地址为：​http://IP:3000/​
![在这里插入图片描述](https://img-blog.csdnimg.cn/9503ffe6482a4350b0ffebf252e631ad.png)

 2. 设置网络接口

	将后台的访问端口更改为 3000，避免与 NAS 后台的 80 端口发生冲突，DNS 端口保持为 53 即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a089d336184f47e1beff31e85e13f686.png)

 3. 设置管理员账户

	![在这里插入图片描述](https://img-blog.csdnimg.cn/1a72e0023e4a4170b62d8f715f9beffe.png)

 4. 完成初始化设置

	![在这里插入图片描述](https://img-blog.csdnimg.cn/d6bb81192580473b98805fb0adae48b6.png)
## 后期配置
### 常规设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/cea41670bad54bc0babb72187d802fa1.png)

- 过滤器更新间隔：DNS 过滤清单默认更新间隔，一般为 3 天 - 7 天
- 使用 AdGuard 「浏览安全」网页服务：作用与 Chrome 网页安全性检查类似，开启后，当用户访问存在潜在威胁的网站时，AdGuard 会主动拦截并弹出提示
- 使用 AdGuard 「家长控制」 服务：如果家中有尚未成年的孩子，建议开启，避免访问不良网站
- 强制安全搜索：隐藏 Bing、Google、Yandex、YouTube 网站上 NSFW 等不适宜的内容
- 查询记录保留时间：AdGuard Home 服务端采用 Sqlite 文件数据库存储日志，长时间保留可能会降低运行速度，同时占用大量的储存空间，家庭用户一般保留 24 小时 - 7 天即可
- 统计数据保留时间：用于仪表盘的数据展示，一般保留 24 小时 - 7 天即可

### DNS 设置 
![在这里插入图片描述](https://img-blog.csdnimg.cn/c250d07ed48443838cd3ec1c7d3d6c04.png)
  

 - 上游 DNS 服务器：AdGuard Home 的上游 DNS 服务器，可参考下方推荐列表，一般保留 1 - 2 个即可。AdGuard
   Home 除了可以作为广告过滤网关，如果设置了纯净 DNS 后，还可以避免运营商的 DNS 劫持 
 - BootStrap DNS服务器地址：作为 DoH / DoT DNS 的前置 DNS 解析器，可参考下方推荐列表
- 查询方式、速度限制、EDNS、DNSSEC、拦截模式、DNS 缓存设置、访问设置可根据需要进行调整，一般保持默认设置即可
### DNS 服务器推荐

不同地区连接至 DNS 服务器的速度各有差异，各位可以通过 Ping 测速的方式寻找当地连接延迟最低的 DNS 服务器。更多 DNS 服务器可以在 AdGuard 文档中找到。
|DNS 提供商  |类别  |地址  |
|--|--|--|
|阿里  | IPv4 DNS |	223.5.5.5  |
|阿里  | IPv6 DNS |	2400:3200:baba::1 |
|阿里  | DNS-over-Https|https://dns.alidns.com/dns-query|
|DNSPod	|IPv4 DNS	|119.29.29.29
|DNSPod	|DNS-over-Https	|https://doh.pub/dns-query
|114	|IPv4 DNS	|114.114.114.114

### DNS 封锁清单
![在这里插入图片描述](https://img-blog.csdnimg.cn/fcae2fad061e415e8c00eff61e387622.png)


为了更好地发挥AdGuard Home去广告的功能，仅使用默认的过滤规则是不够的。但是，过多的过滤规则会影响解析速度，因此添加过滤规则需要权衡利弊。主要浏览国内网站的用户可以使用anti-AD + Halflife过滤规则，而需要浏览国外网站的用户则可以根据需要添加AdGuard DNS Filter、Fanboy’s Annoyances List等规则。由于不同的规则存在重叠，可以通过AdGuard Home的拦截日志分析哪些规则使用频率最高，哪些规则拦截频率最低，并进行权衡取舍。

|名称	|简介	|地址|
|--|--|--|
|AdGuard DNS Filter	|AdGuard 官方维护的广告规则，涵盖多种过滤规则	|https://raw.githubusercontent.com/AdguardTeam/FiltersRegistry/master/filters/filter_15_DnsFilter/filter.txt
|EasyList	|Adblock Plus 官方维护的广告规则	|https://easylist-downloads.adblockplus.org/easylist.txt
|EasyList China	|面向中文用户的 EasyList 去广告规则	|https://easylist-downloads.adblockplus.org/easylistchina.txt
|EasyPrivacy	|反隐私跟踪、挖矿规则	|https://easylist-downloads.adblockplus.org/easyprivacy.txt
|Halflife 	|规则涵盖了 EasyList China、EasyList Lite、CJX ’s Annoyance、乘风视频过滤规则，以及补充的其它规则	|https://raw.githubusercontent.com/o0HalfLife0o/list/master/ad.txt
|Xinggsf 乘风过滤	|国内网站广告过滤规则	|https://raw.githubusercontent.com/xinggsf/Adblock-Plus-Rule/master/rule.txt
|Xinggsf 乘风视频过滤	|视频网站广告过滤规则	|https://raw.githubusercontent.com/xinggsf/Adblock-Plus-Rule/master/mv.txt
MalwareDomainList	|恶意软件过滤规则	|https://www.malwaredomainlist.com/hostslist/hosts.txt
Adblock Warning Removal List	|去除禁止广告拦截提示规则	|https://easylist-downloads.adblockplus.org/antiadblockfilters.txt
Anti-AD	|命中率高、兼容性强	|https://anti-ad.net/easylist.txt
Fanboy’s Annoyances List	|去除页面弹窗广告规则	|https://easylist-downloads.adblockplus.org/fanboy-annoyance.txt

同时也在本人Github和Gitee上更新了这些过滤规则：

```bash
https://github.com/DarkKnightYHJ/AdGuardHomeList
https://gitee.com/knightyhj/AdGuardHomeList
```
## 在设备上使用 AdGuard Home DNS
![在这里插入图片描述](https://img-blog.csdnimg.cn/0354ae3d451a4c1aa79bbf7cbfb23d27.png)

在AdGuard Home的**设置指导**页面可以找到相应系统的DNS设置方法。为了避免路由器上使用时出现问题，建议先在电脑或手机上进行设置，并测试是否能正常解析访问一些网站。如果一切正常，仪表盘界面会显示有关统计信息，而查询日志界面会显示解析的详细记录信息。最后再重申，在使用路由器时，请确保已充分理解DNS设置的相关步骤和风险，并进行必要的安全配置。
# 使用效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/925c5e8c4206437cb512f8a7fd2c6d93.png)
# 参考
[AdGuard Home](https://adguard.com/zh_cn/adguard-home/overview.html)
[AdGuard Home 是什么？为什么无法去广告？](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-0.html) 
[AdGuard Home 自建 DNS 防污染、去广告教程 #1 - 安装部署详解(Docker)](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-1.html) 
[AdGuard Home 自建 DNS 防污染、去广告教程 #2 - 优化增强设置详解](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-2.html)

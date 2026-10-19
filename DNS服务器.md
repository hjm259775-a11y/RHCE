# DNS服务器



**简介**

*   产生原因：如果服务器只有ip地址，在一个集群中可能有几十甚至上百上千台设备，作为人类很难记录ip和服务器功能之间的关联，为了简化对服务器的描述，于是产生了域名，域名和ip之间绑定关系，我们可以对某个设备根据功能起一个域名(比如[filesystem.dev](https://filesystem.dev/)就是一个关于文件服务器域名)，这样降低了我们工作中的记忆成本。

*   发展到现在，域名是互联网找那个必不可少的组成部分，它大大的降低了互联网用户对各种应用服务的访问成本，提升网站点击率，视频活跃度等跟互联网商业体系直接挂钩参数。

*   作用：给特定的服务器创建一个容易记忆及使用的域名，方便连接和处理相关数据。
*   连接方式：udp为主 tcp为辅



**工作方式：**

*   DNS (Domain Name System) 是互联网上的一项服务，用于将域名和IP地址进行相互映射，使人更方便的访问互联网

*   正向解析：域名->IP

*   反向解析：IP->域名

*   53号端口



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261019101823794.png" alt="image-20261019101823794" style="zoom: 50%;" />



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261019103507009.png" alt="image-20261019103507009" style="zoom:50%;" />





## 搭建DNS服务器



```
[root@xgz ~]# yum install bind ————————————————————————————————————————安装
```





### 正向解析

```
[root@localhost /]# vim /etc/named.conf ——————————————————————————————————修改主配置文件/etc/named.conf让其监听自己拥有所有ip服务所有来访ip

————————————————————————————————————————————————————————————————————————————————————————————————
 10 options {
 11         listen-on port 53 { any; }; ——————————————————————————————————修改为any，让监听地址改为所有IP和域名（
 12         listen-on-v6 port 53 { ::1; };
 13         directory       "/var/named";
 14         dump-file       "/var/named/data/cache_dump.db";
 15         statistics-file "/var/named/data/named_stats.txt";
 16         memstatistics-file "/var/named/data/named_mem_stats.txt";
 17         secroots-file   "/var/named/data/named.secroots";
 18         recursing-file  "/var/named/data/named.recursing";
 19         allow-query     { any; }; ————————————————————————————————————修改为any，允许所有人访问本机的DNS服务
 20
————————————————————————————————————————————————————————————————————————————————————————————————~~


[root@localhost named]# vim /etc/named.rfc1912.zones —————————————————————在/etc/named.rfc1912.zones加入区域声明

————————————————————————————————————————————————————————————————————————————————————————————————
 29 zone "xgz.com" IN { ——————————————————————————————————————————————————添加，本区域为所有xgz.com后缀的
 30         type primary;
 31         file "/etc/named/xgz.com.zone";
 32         allow-update { none; };
 33 };
————————————————————————————————————————————————————————————————————————————————————————————————~~


[root@localhost named]# cd /var/named/
[root@localhost named]# ll
总计 0
drwxrwx--- 2 named named  6  9月 7日 08:00 data
drwxrwx--- 2 named named  6  9月 7日 08:00 dynamic
lrwxrwxrwx 1 root  named 18  9月 7日 08:00 named.ca -> ../../etc/named.ca
lrwxrwxrwx 1 root  named 33  9月 7日 08:00 named.empty -> ../../usr/share/named/named.empty
lrwxrwxrwx 1 root  named 37  9月 7日 08:00 named.localhost -> ../../usr/share/named/named.localhost
lrwxrwxrwx 1 root  named 36  9月 7日 08:00 named.loopback -> ../../usr/share/named/named.loopback
drwxrwx--- 2 named named  6  9月 7日 08:00 slaves
[root@localhost named]# cp -p named.localhost xgz.com.zone ————————————————在/var/named目录中创建正向解析的数据文件，通过name.localhost模板文件复制一份区域文件，-p保证权限和用户组与模板一致

[root@localhost named]# vim xgz.com.zone
——————————————————————————————————————————————————————————————————————————————————————————————————~~
$TTL 1D
xgz.com.        IN SOA  ns.xgz.com. 242348535.qq.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
xgz.com. NS ns.xgz.com. ——————————————————————————指定 xgz.com 这个域名的权威 DNS 服务器是谁
ns.xgz.com. A 192.168.44.99 ——————————————————————ns.xgz.com 就在 192.168.44.99 这台机器上（也就是我自己）
www A 11.11.11.11 ————————————————————————————————当用户查 www.xgz.com 时，服务器就告诉他：“去访问 11.11.11.11 吧。”
bbs A 22.22.22.22
——————————————————————————————————————————————————————————————————————————————————————————————————~~

[root@localhost named]# systemctl enable --now named.service

[root@localhost named]# nslookup www.xgz.com 192.168.44.9 ————————————————————可以看到，这边问出来IP了
Server:         192.168.44.9
Address:        192.168.44.9#53

Name:   www.xgz.com
Address: 11.11.11.11

```







|  记录类型  | 全称                            | 核心作用                                                    | 使用场景                                                     | 关键特点 / 限制                                              |
| :--------: | :------------------------------ | :---------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|     A      | Address (IPv4 地址记录)         | 将域名映射到IPv4 公网 IP                                    | 网站、Nextcloud 私有云、Web 业务、普通服务器解析             | 直接填写 IP；同一主机记录可多条 A 记录实现简易轮询；不能和 CNAME 共存 |
|    AAAA    | IPv6 Address 记录               | 将域名映射到IPv6 地址                                       | 服务器开通 IPv6 网络、支持 IPv6 访问站点                     | 国内普及率低，需运营商支持 IPv6 网络                         |
|   CNAME    | Canonical Name 别名记录         | 将当前域名指向另一个域名，不直接绑定 IP                     | 弹性云主机（IP 常变动）、CDN 加速、批量站点统一指向          | 1. 同一主机不能同时有 A/AAAA 记录 2. 不能指向泛域名 3. 无法搭配 MX 记录 |
|     MX     | Mail Exchange 邮件交换记录      | 指定接收邮件的服务器，搭建企业邮箱                          | 企业邮箱、自建邮件服务                                       | 1. 每条记录带优先级数字（数字越小，邮箱服务器优先级越高） 2. 只能填写域名，不能填 IP |
|    TXT     | Text 文本记录                   | 存储自定义文本字符串                                        | 1. SSL 证书域名所有权验证 2. 邮箱防垃圾邮件：SPF/DKIM/DMARC3. 业务域名校验 | 文本内容有长度限制，多条 TXT 可共存                          |
|     NS     | Name Server 域名服务器记录      | 指定负责解析该域名的 DNS 服务器                             | 域名拆分解析、自定义第三方 DNS、子域名独立解析               | 主域名必须配置 NS 记录，修改后 DNS 全局同步较慢              |
|    SRV     | Service Location 服务定位记录   | 标记内网 / 外网服务的地址、端口、权重                       | LDAP 域登录、即时通讯、数据库集群、内部业务服务              | 格式固定，包含服务名、协议、优先级、端口、目标域名           |
|    CAA     | Certificate Authority 授权记录  | 限制哪些 CA 机构可以给当前域名颁发 SSL 证书                 | 网站 HTTPS 安全加固，防止他人滥发证书劫持域名                | 安全防护类记录，企业生产环境推荐配置                         |
|    PTR     | Pointer 反向解析记录            | 通过 IP 反向查询对应的域名                                  | 邮件服务器（防止发垃圾邮件）、服务器审计溯源                 | 由 IP 运营商 / 机房管理，用户无法在域名控制台直接添加        |
|    SOA     | Start of Authority 起始授权记录 | 存储域名 DNS 基础管理信息（刷新时间、管理员邮箱、过期时间） | 所有域名默认自带，无需手动新增                               | 每个域名仅 1 条，是 DNS 解析的基础元数据                     |
| * (泛域名) | 泛解析记录                      | 所有未单独配置的子域名统一指向同一地址                      | 测试站点、多子域名统一跳转                                   | 自定义主机记录优先级高于泛域名；泛域名不能用于 MX、NS        |







### 反向解析





```
[root@localhost named]# vim /etc/named.conf ——————————————————————————在/etc/named.rfc1912.zones文件中添加反向解析功能
——————————————————————————————————————————————————————————————————————————————————————————————————————
 31         recursion yes;
 32         forward only;
 33         forwarders {8.8.8.8;114.114.114.114;};
 34         dnssec-validation no;
——————————————————————————————————————————————————————————————————————————————————————————————————————



[root@localhost named]# vim /etc/named.rfc1912.zones ————————————————在/var/named目录中创建反向解析的数据文件
——————————————————————————————————————————————————————————————————————————————————————————————————————
 41 zone "44.168.192.in-addr.arpa" IN {
 42         type primary;
 43         file "/var/named/192.168.44.arpa";
 44         allow-update { none; };
 45 };
——————————————————————————————————————————————————————————————————————————————————————————————————————


[root@localhost named]# cp -p named.loopback /var/named/192.168.44.arpa

[root@localhost named]# vim 192.168.44.arpa
——————————————————————————————————————————————————————————————————————————————————————————————————————
$TTL 1D
@       IN SOA  ns.xgz.com. 242348535.qq.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.xgz.com.
ns      A       192.168.44.9
9       PTR     ns.xgz.com.
10      PTR     www.xgz.com.
11      PTR     bbs.xgz.com.
——————————————————————————————————————————————————————————————————————————————————————————————————————



[root@localhost named]# nslookup 192.168.44.11 127.0.0.1 ————————————————————————————可以看到，也出来了
11.44.168.192.in-addr.arpa      name = bbs.xgz.com.

```







```
客户端配置dns为172.25.254.100，client为会话文件名
[root@client ~]# nmcli connection modify client ipv4.dns 172.25.254.100
```







### 主从DNS





注意：

*   时间同步必须保持一致性
*   bind最好使用同一版本







*   主服务器

```
[root@localhost named]# vim /etc/named.rfc1912.zones
————————————————————————————————————————————————————————————————————————————————————————————————
 29 zone "xgz.com" IN {
 30         type primary;
 31         file "/var/named/xgz.com.zone";
 32         allow-transfer { 192.168.44.99; }; ————————————————————————允许把你这台主 DNS 服务器上的“区域数据”，同步（复制）给 IP 为 192.168.44.99 的从 DNS 服务器。
 33 };
 41 zone "44.168.192.in-addr.arpa" IN {
 42         type primary;
 43         file "/var/named/192.168.44.arpa";
 44         allow-transfer { 192.168.44.99; };
————————————————————————————————————————————————————————————————————————————————————————————————



[root@localhost named]# vim /var/named/xgz.com.zone ——————————————————————————————修改正向解析区域文件
————————————————————————————————————————————————————————————————————————————————————————————————
$TTL 1D
xgz.com.        IN SOA  ns.xgz.com. 242348535.qq.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum

xgz.com.        IN NS   ns.xgz.com.
xgz.com.        IN NS   slave.xgz.com. ————————————————————————加上这个
slave		   IN  A   192.168.44.99 ————————————————————————加上这个
ns.xgz.com.     IN A    192.168.44.9
www             IN A    11.11.11.11
bbs             IN A    22.22.22.22
————————————————————————————————————————————————————————————————————————————————————————————————



[root@localhost named]# vim /var/named/192.168.44.arpa ————————————————————————修改反向解析区域文件
————————————————————————————————————————————————————————————————————————————————————————————————
$TTL 1D
@       IN SOA  ns.xgz.com. 242348535.qq.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.xgz.com.
        NS      slave.xgz.com.
slave   A       192.168.44.99 ————————————————————————加上这个
ns      A       192.168.44.9
9       PTR     ns.xgz.com.
10      PTR     www.xgz.com.
11      PTR     bbs.xgz.com.
————————————————————————————————————————————————————————————————————————————————————————————————

```







































## 配置文件



### /etc/named.conf

主配置文件

```
[root@localhost /]# vim /etc/named.conf

————————————————————————————————————————————————————————————————————————————————————————————————————————
1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9
 10 options {
 11         listen-on port 53 { any; }; ——————————————————————————————————修改为any，让监听地址改为所有IP和域名（
 12         listen-on-v6 port 53 { ::1; };
 13         directory       "/var/named";
 14         dump-file       "/var/named/data/cache_dump.db";
 15         statistics-file "/var/named/data/named_stats.txt";
 16         memstatistics-file "/var/named/data/named_mem_stats.txt";
 17         secroots-file   "/var/named/data/named.secroots";
 18         recursing-file  "/var/named/data/named.recursing";
 19         allow-query     { any; }; ————————————————————————————————————修改为any，允许所有人访问本机的DNS服务
 20
 21         /*
 22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable
 24            recursion.
 25          - If your recursive DNS server has a public IP address, you MUST enable access
 26            control to limit queries to your legitimate users. Failing to do so will
 27            cause your server to become part of large scale DNS amplification
 28            attacks. Implementing BCP38 within your network would greatly
 29            reduce such attack surface
 30         */
 31         recursion yes; ———————————————————————————————————————————————yes是递归模式，no是迭代模式
 32
 33         dnssec-validation yes;
 34
 35         managed-keys-directory "/var/named/dynamic";
 36         geoip-directory "/usr/share/GeoIP";
 37
 38         pid-file "/run/named/named.pid";
 39         session-keyfile "/run/named/session.key";
 40
 41         /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
 42         include "/etc/crypto-policies/back-ends/bind.config";
 43 };
 44
 45 logging {
 46         channel default_debug {
 47                 file "data/named.run";
 48                 severity dynamic;
 49         };
 50 };
 51
 52 zone "." IN {
 53         type hint;
 54         file "named.ca";
 55 };
 56
 57 include "/etc/named.rfc1912.zones";
 58 include "/etc/named.root.key";
 59

————————————————————————————————————————————————————————————————————————————————————————————————————————
```





**DNS服务器工作模式：**

*   递归模式：别人来问地址，本DNS一个一个跑腿问根域名服务器。。顶级域名服务器。。最后把IP地址交给别人

*   迭代模式：别人来问地址，本DNS给别人根域名服务器地址，自己去找吧

    （本地DNS是递归模式，全球的根域名DNS服务器就是迭代模式，二者相辅相成）





### /etc/named.conf的if线

（本地DNS经常干的事，因为本地没那么多域名，只能去问）

```
 31         recursion yes; ———————————————————————————————————————————————yes是递归模式，no是迭代模式
 32
   		   forward only; —————————————————————————————————————————————————开启dns转发功能，当遇到本机不能提供解析的域名会转发给上级域名解析服务器，
		   forwarders {8.8.8.8;114.114.144.114;}; ———————————————————————指定上级域名解析服务器
 
 33         dnssec-validation no; ————————————————————————————————————————关闭验证
```





### /etc/named.rfc1912.zones

当前域名解析的服务的声明文件

（域名的区域声明信息所在文件）



```
[root@localhost /]# vim /etc/named.rfc1912.zones

——————————————————————————————————————————————————————————————————————————————————————————————————————

# 关于localhost.localdomain的正向解析声明
 17 zone "localhost.localdomain" IN {
 18         type primary; ————————————————————————————————————————————如果是从服务器就是type slave;
 19         file "named.localhost"; ——————————————————————————————————当前区域的详细数据文件
 20         allow-update { none; }; ——————————————————————————————————是否允许更新
 21 };
 22
 # 关于localhost的正向解析声明
 23 zone "localhost" IN {
 24         type primary;
 25         file "named.localhost";
 26         allow-update { none; };
 27 };
 28
 # ipv6反向解析
 29 zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
 30         type primary;
 31         file "named.loopback";
 32         allow-update { none; };
 33 };
 34
 35 zone "1.0.0.127.in-addr.arpa" IN {
 36         type primary;
 37         file "named.loopback";
 38         allow-update { none; };
 39 };
 40
 41 zone "0.in-addr.arpa" IN {
 42         type primary;
 43         file "named.empty";
 44         allow-update { none; };
 45 };

——————————————————————————————————————————————————————————————————————————————————————————————————————
```











### /var/named

目录所有的域名解析数据文件都在该目录中



```
[root@localhost /]# cd /var/named/
[root@localhost named]# ll
总计 0
drwxrwx--- 2 named named  6  9月 7日 08:00 data
drwxrwx--- 2 named named  6  9月 7日 08:00 dynamic
lrwxrwxrwx 1 root  named 18  9月 7日 08:00 named.ca -> ../../etc/named.ca
lrwxrwxrwx 1 root  named 33  9月 7日 08:00 named.empty -> ../../usr/share/named/named.empty
lrwxrwxrwx 1 root  named 37  9月 7日 08:00 named.localhost -> ../../usr/share/named/named.localhost ——————正向解析的模板文件
lrwxrwxrwx 1 root  named 36  9月 7日 08:00 named.loopback -> ../../usr/share/named/named.loopback ————————反向解析的模板文件
drwxrwx--- 2 named named  6  9月 7日 08:00 slaves ————————————————————————————————————该目录会保存从主服务器下载的域名解析数据文件




[root@localhost named]# cat named.localhost
————————————————————————————————————————————————————————————————————————————————————————————————————
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
————————————————————————————————————————————————————————————————————————————————————————————————————

```
























# Web服务器

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261016212737666.png" alt="image-20261016212737666" style="zoom: 67%;" />

迅速过一下背景介绍







Web网络服务是一种被动访问的服务程序，即只有接收到互联网中其他主机发出的请求后才会响应，最终用于提供服务程序的Web服务器会通过HTTP（超文本传输协议）或HTTPS（安全超文本传输协议）把请求的内容传送给用户









在 http 通信中，每个 http 请求报文都包含一个方法，用以告诉 web 服务器端需要执行哪些具体的动作，这些动作包括：获取指定 web 页面、提交内容到服务器、删除服务器上资源文件等。

| 序号 | 请求方法 | 核心用途                                    | 核心特点                                                     | 典型适用场景                                               |
| :--- | :------- | :------------------------------------------ | :----------------------------------------------------------- | :--------------------------------------------------------- |
| 1    | GET      | 请求 / 查询指定资源，返回完整的响应正文主体 | 参数拼接在 URL 中，安全只读、可缓存，无请求体，仅做数据读取不修改服务端资源 | 打开网页、查询列表数据、普通接口 GET 请求、资源下载        |
| 2    | HEAD     | 仅获取响应头，不返回页面 / 接口正文内容     | 逻辑完全等同于 GET，但服务端仅返回响应头、无响应体，快速探测资源状态 | 运维检测服务存活、校验文件大小、快速判断接口是否正常       |
| 3    | POST     | 向服务端提交业务数据，完成新增 / 修改类操作 | 数据全部放在请求体中，可新增 / 修改服务端资源，不可缓存，无长度限制 | 提交登录表单、上传文件、新增用户 / 订单 / 文章等业务数据   |
| 4    | PUT      | 全量更新 / 替换服务端指定资源的完整内容     | 用请求体数据完整覆盖目标资源，操作幂等，多次执行结果一致     | 全量更新用户信息、修改完整的资源配置、覆盖已有文档内容     |
| 5    | DELETE   | 请求服务端删除指定的目标资源                | 直接删除服务端对应资源，操作幂等，多次执行结果一致           | 删除用户账号、删除文章 / 订单、清理服务端无效资源          |
| 6    | CONNECT  | 建立隧道式的代理连接                        | HTTP/1.1 协议预留，用于将普通 TCP 连接转为管道式代理连接     | HTTPS 代理、内网穿透、翻墙代理、跨网访问隧道搭建           |
| 7    | OPTIONS  | 查询服务端的性能、支持的请求方法与跨域规则  | 预检类请求，不操作业务数据，仅获取服务端的接口能力信息       | 前端跨域请求自动预检、查询接口支持的请求方法、校验跨域权限 |
| 8    | TRACE    | 回显服务端收到的完整请求报文，用于测试诊断  | 服务端原样返回接收到的全部请求内容，用于链路排查与调试       |                                                            |





------------------------------------------------------------------------------------------

状态代码：由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值

1xx：指示信息 —— 表示请求已接收，继续处理
2xx：成功 —— 表示请求已被成功接收、理解
3xx：重定向 —— 要完成请求必须进行更进一步的操作
4xx：客户端错误 —— 请求有语法错误或请求无法实现
5xx：服务器端错误 —— 服务器未能实现合法的请求

常见状态代码、状态描述的说明如下：

200 OK：客户端请求成功
400 Bad Request：客户端请求有语法错误，不能被服务器所理解
401 Unauthorized：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头
403 Forbidden：服务器收到请求，但是拒绝提供服务
404 Not Found：请求资源不存在，举个例子：输入了错误的URL

------------------------------------------------------------------------------------------------------------





——————————————————————————==**以上看看就行了**==————————————————————————————









## 简介



*   作用：基于http或https，接收用户侧发起数据请求，并对请求数据加以处理后，对用户返回请求相关的内容。

*   被动：web服务不会主动地去连接客户端，它只会等待用户的请求，并处理。





服务器主机：

*   虚拟主机：就是将一台主机的资源分割成多份给到不同web服务。
*   VPS：云服务早期的一种形态，他将集群构成资源自由分配给购买云服务器的客户，云服务器扩容上存在难度。
*   ECS：当下云服务商主要使用的形式，他的特点除了具备vps的所有有点意外，还能够弹性缩容扩容。
*   独立服务器：一台独立主机





URI和URL

*   URI：统一资源标识
    *   URN：统一资源名称，就是某个物品或者某个资源的唯一名称
    *   URL：统一资源定位符，网络资源的唯一地址

*   URL格式：http、https

    协议头://域名或者ip+/资源路径





**HTTP/HTTPS**

请求方法：

*   get：主要是用资源下载，使用get请求时可以上传有限的数据。
*   post：主要是用于上传一部分数据，用于获取服务器上的数据。
*   put： 全量更新。
*   delete： 删除服务器上资源。
*   patch：局部更新。



状态代码：由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值

1xx：指示信息 —— 表示请求已接收，继续处理
2xx：成功 —— 表示请求已被成功接收、理解
3xx：重定向 —— 要完成请求必须进行更进一步的操作
4xx：客户端错误 —— 请求有语法错误或请求无法实现
5xx：服务器端错误 —— 服务器未能实现合法的请求

常见状态代码、状态描述的说明如下：

200 OK：客户端请求成功
400 Bad Request：客户端请求有语法错误，不能被服务器所理解
401 Unauthorized：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头
403 Forbidden：服务器收到请求，但是拒绝提供服务
404 Not Found：请求资源不存在，举个例子：输入了错误的URL









## 使用nginx搭建web网站



*   介绍：目前世界上使用最广泛web服务器，许多大厂使用的Tengine web服务器本质上都是变种的nginx。他可以实现正向代理，反向代理，负载均衡，动静分离。
*   特点：高性能，低功耗，异步高并发。
*   nginx架构
    *   master：指挥者，他则调度worker及资源。
    *   worker：真正干活的工作者。



**正向代理和反向代理**

正向代理的特点：

​	客户端隐匿性。在服务器眼里，真正访问它是那台代理服务器。

反向代理的特点：

​	服务器隐匿性。在客户端眼里，他只知道他访问了服务器，但是不知道服务器后方真实处理数据的设备是哪个。

​	客户端只与Nginx代理服务器通信，Nginx将请求分发给后端的真实服务器，并将响应返回给客户端，常用于负载均衡、动静分离、安全防护等。





Nginx特有命令（常用）

```
nginx -s reload ——————————————————————————当修改了配置文件但是不想重启服务器，可以用
nginx -t —————————————————————————————————检查配置文件的语法是否有错
nginx -v —————————————————————————————————查看nginx 版本号
nginx -s reload ——————————————————————————关闭
```

注意：尽量不要用nginx启动，不然systermctl就无法控制他了





### 实验



安装

```
[root@xgz ~]# yum install nginx
[root@xgz ~]# systemctl enable --now nginx
[root@xgz ~]# systemctl status nginx
先下载、运行、查看状态
```



查看网络使用情况，这东西挺有用的

```
[root@xgz ~]# netstat -lntup
查看网络使用情况（-l表示查看正在监听的程序，-n显示数字ip而非域名，-t显示tcp的程序，-u显示udp程序，-p显示这些程序的进程id）
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261017150928577.png" alt="image-20261017150928577" style="zoom:67%;" />





#### 配置文件



目录

```
[root@xgz ~]# cd /etc/nginx
[root@xgz nginx]# ll
总计 68
drwxr-xr-x 2 root root    6  8月31日 08:00 conf.d ——————————————————————————————————子配置文件的目录
drwxr-xr-x 2 root root    6  8月31日 08:00 default.d ———————————————————————————————默认配置的目录
-rw-r--r-- 1 root root 1077  8月31日 08:00 fastcgi.conf ————————————————————————————带gi的文件都是转发给开发者写好的web服务框架使用的转发文件
-rw-r--r-- 1 root root 1077  8月31日 08:00 fastcgi.conf.default ————————————————————带default后缀的文件全部都是备份文件能起到配置模版的作用
-rw-r--r-- 1 root root 1007  8月31日 08:00 fastcgi_params
-rw-r--r-- 1 root root 1007  8月31日 08:00 fastcgi_params.default
-rw-r--r-- 1 root root 2837  8月31日 08:00 koi-utf
-rw-r--r-- 1 root root 2223  8月31日 08:00 koi-win
-rw-r--r-- 1 root root 5349  8月31日 08:00 mime.types ——————————————————————————————配置支持的媒体类型
-rw-r--r-- 1 root root 5349  8月31日 08:00 mime.types.default
-rw-r--r-- 1 root root 1983  8月31日 08:00 nginx.conf ——————————————————————————————主配置文件
-rw-r--r-- 1 root root 2656  8月31日 08:00 nginx.conf.default
-rw-r--r-- 1 root root  636  8月31日 08:00 scgi_params
-rw-r--r-- 1 root root  636  8月31日 08:00 scgi_params.default
-rw-r--r-- 1 root root  664  8月31日 08:00 uwsgi_params
-rw-r--r-- 1 root root  664  8月31日 08:00 uwsgi_params.default
-rw-r--r-- 1 root root 3610  8月31日 08:00 win-utf
```





配置文件

```
[root@xgz nginx]# vim nginx.conf

——————————————————————————————————————————————————————————————————————————————————————————————
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx; ————————————————————————————————————————————————————————————————当nginx启动时会用系统中nginx账号
worker_processes auto; —————————————————————————————————————————————————————worker进程数量默认情况下auto对应了你的cpu数量
error_log /var/log/nginx/error.log notice; —————————————————————————————————错误日志保存的地址（正确日志也在那）
pid /run/nginx.pid; ————————————————————————————————————————————————————————nginx运行起来以后可以通过查看/run/nginx·pid来获取nginx的pid

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf; ———————————————————————————————————模组的子配置文件

events {
    worker_connections 1024;
}

http { —————————————————————————————————————————————————————————————————————访问日志格式（可自行修改）
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main; ———————————————————————————访问日志的地址

    sendfile            on; 
    tcp_nopush          on; ————————————————————————————————————————————————发送件时，尽量积攒数据包然后统一发送
    keepalive_timeout   65; ————————————————————————————————————————————————打开网页后65秒内不会产生新的http连接
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf; ——————————————————————————————————————子配置文件的目录（include代表把子配置文件拉到36行这里来）

    server { ———————————————————————————————————————————————————————————————子配置按照这个格式写
        listen       80; ———————————————————————————————————————————————————监听ipv4的80端口（http默认端口）
        listen       [::]:80; ——————————————————————————————————————————————监听ipv6的80端口
        server_name  _; ————————————————————————————————————————————————————就是在浏览器中访问的域名或ip，_下划线的作用是用来兜底，当访问的流量所访问的server_name不存在于已建立的server，那么这个_就负责兜底解析
        root         /usr/share/nginx/html; ————————————————————————————————————————————Web服务资源路径，web服务的默认发布目录（URL上面域名后面的路径就是在这个目录下的）
        index		index.html ————————————————————————————————————————————————————————默认发布文件，本质就是在浏览器中输入域名后直接回车会打开的页面
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl;
#        listen       [::]:443 ssl;
#        http2        on;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#    }

}
——————————————————————————————————————————————————————————————————————————————————————————————————————————

```











#### 实践



##### 创建虚拟主机

*   创建虚拟主机

```
[root@xgz ~]# vim /etc/nginx/nginx.conf

————————————————————————————————————————————————————————————————————————————————————————————————————————
35     include /etc/nginx/conf.d/*.conf;
 36     server{ ——————————————————————————————————————————————————————自己添加的
 37         listen 80;
 38         server_name 192.168.44.99;
 39         root /root/web;
 40         index index.html;
 41     }
 42     server{ ——————————————————————————————————————————————————————自己添加的
 43         listen 90;
 44         server_name 192.168.44.99;
 45         root /root/web1;
 46         index haha.html;
 47     }
 48
 49     server {
 50         listen       80;
 51         listen       [::]:80;
 52         server_name  _;
 53         root         /usr/share/nginx/html;
 54
 55         # Load configuration files for the default server block.
 56         include /etc/nginx/default.d/*.conf;
 57     }
——————————————————————————————————————————————————————————————————————————————————————————————————————————



[root@xgz ~]# chmod 755 /root ———————————————————————————————————————在root文件夹下的，nginx用的普通用户身份，需要加权限
[root@xgz ~]# ll
总计 4
-rw-------. 1 root root 974 10月13日 11:21 anaconda-ks.cfg
drwxr-xr-x  2 root root  24 10月17日 19:05 web
drwxr-xr-x  2 root root  23 10月17日 19:03 web1
[root@xgz ~]# tree web
web
└── index.html

1 directory, 1 file
[root@xgz ~]# tree web1
web1
└── haha.html

1 directory, 1 file
[root@xgz ~]# cat web/index.html
hello world!
[root@xgz ~]# cat web1/haha.html
dierge
```



<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261017193132954.png" alt="image-20261017193132954" style="zoom:67%;" />

强而有力！强而有力啊！（当然，这东西还只能在区域网里面玩）







##### 带密码

*   创建虚拟主机（带密码）

```
[root@xgz ~]# yum install httpd-tools -y
[root@xgz ~]# useradd nailong
[root@xgz ~]# passwd nailong ——————————————————————————————————————————————————nailong的密码是123
[root@xgz ~]# htpasswd -c /etc/nginx/passwd nailong ———————————————————————————在/etc/nginx/passwd路径下创建密码文件，nailong这边密码是abc（想创建第二个用户的话不需要-c了）

[root@xgz ~]# vim /etc/nginx/nginx.conf
——————————————————————————————————————————————————————————————————————————————————————————————————————————————
 36     server{
 37         listen 80;
 38         server_name 192.168.44.99;
 39         root /root/web;
 40         index index.html;
 41         auth_basic on; ————————————————————————————————————————————————————开启密码功能
 42         auth_basic_user_file /etc/nginx/passwd; ———————————————————————————此路径为密码本路径，将密码本设置给网站
 43     }
——————————————————————————————————————————————————————————————————————————————————————————————————————————————

[root@xgz ~]# systemctl restart nginx.service
```

**在这个 Nginx 实验里，`useradd nailong` 完全是多余的**，根本不需要系统里有这个用户

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261017200617506.png" alt="image-20261017200617506" style="zoom:67%;" />



核电，轻而易举啊！🤪（密码输入的是abc）





##### 用于下载



```
[root@xgz nginx]# vim nginx.conf

——————————————————————————————————————————————————————————————————————————————————————————————————————————
 50     server{
 51         listen 81;
 52         server_name  192.168.44.99;
 53         root /root/web;
 54         autoindex on; ————————————————————————————————————打开网页可以下载/root/web该路径下的文件了
 55     }

——————————————————————————————————————————————————————————————————————————————————————————————————————————

[root@xgz nginx]# systemctl restart nginx.service
```







##### 基于域名



```
#1 . 修改windows本地域名解析文件
C:\Windows\System32\drivers\etc\hosts
在其中添加自定义的域名解析：
xxx.xxx.xxx.100 www.xxx.com

2.创建一个子配置文件必须是xxx.conf
[root@server web]# vim /etc/nginx/conf.d/lyh.conf

——————————————————————————————————————————————————————————————————————————————————————
1 server{
2 	listen 80;
3 	server_name www.xgz.com;
4 	root /etc;
5 	autoindex on;
6 }
——————————————————————————————————————————————————————————————————————————————————————
```



注意：

​	精确匹配：server_name [www.nginx.com](https://www.nginx.com/) ;
​	左侧匹配：server_name *.[nginx.com](https://nginx.com/) ;
​	右侧匹配：server_name [www.nginx](https://www.nginx/).* ;
​	正则匹配：server_name ~^*$ ;









##### localtion

**`location` 路由匹配**（location和server同时出现root，以location的为准）







location 之后跟着的是资源路径 

​	当 location中属性是 root时，要访问的资源的绝对路径是：root的路径 + url中资源路径

​	当location中属性是alias时，要访问的资源的绝对路径是：ropt的路径 + (url中资源路径 - location 后 资源路径)





location匹配规则:

= 精确匹配；location = /image/ {} 可匹配的url→ http://xxx.xxx.xxx.xxx/image/

------------------------------------------------------------------------------------------------------------------------------------

~ 正则匹配，区分大小写；location ~ ^/image/.*可匹配的url→http://xxx.xxx.xxx.xxx/image/... 

*正则匹配，不区分大小写；location ∗/image/.∗可匹配的\*u**r**l\*→*h**ttp\*://\*xxx\*.*xxx\*.*xxx\*.*xxx\*/*ima**g**e\*/... ∗正则匹配，不区分大小写；\*l**oc**a**t**i**o**n\* ∗/\*ima**g**e\*/.∗ 可匹配的url→ http://xxx.xxx.xxx.xxx/IMAGE/...*

------------------------------------------------------------------------------------------------------------------------------------------------
/xx 前缀匹配 他的优先级低于正则匹配 location /image {} http://xxx.xxx.xxx.xxx/image/...

^~ /xx 前缀匹配，不是正则匹配，他跟上方的前缀匹配唯一的区别是 他的优先级高于正则匹配；

/ 通用匹配，如果没有其它匹配，任何请求都会匹配到 location / {} 可匹配的url→ http://xxx.xxx.xxx.xxx/...

注：匹配优先级：= > ^~ > (~ | ~*) > /xx > /











##### root和alias





场景一：使用 `root` 指令（会多拼一层）

```
server {
    listen 81;
    server_name 192.168.44.99;

    location /image/ {
        root /data/pics/; 
    }
}
```



**Nginx 的查找逻辑：**
`root` 路径 + 完整的 URL 路径

-   `root` 路径 = `/data/pics/`
-   完整 URL 路径 = `/image/1.jpg`
-   **最终硬盘找的路径：** `/data/pics/image/1.jpg`

 **结果：** 你必须创建 `/data/pics/image/` 这个目录，把图片放进去。如果服务器上只有 `/data/pics/1.jpg`，就会报 404 找不到。

-   **用户浏览器输入的网址：** `http://192.168.44.99:81/image/1.jpg`

------

场景二：使用 `alias` 指令（精准替换，推荐）

```
server {
    listen 81;
    server_name 192.168.44.99;

    location /image/ {
        alias /data/pics/; 
    }
}
```



**Nginx 的查找逻辑：**
用 `alias` 路径，**直接替换**掉 `location` 匹配到的部分。

-   `location` 匹配了 `/image/`
-   把 URL 中的 `/image/` 直接换成 `/data/pics/`
-   **最终硬盘找的路径：** `/data/pics/1.jpg`

 **结果：** 只要服务器上有 `/data/pics/1.jpg`，用户就能访问到。

-   **用户浏览器输入的网址：** `http://192.168.44.99:81/image/1.jpg`






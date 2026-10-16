# 远程连接



*   telnet（telnet是**明文传输**数据的远程控制协议）

*   ssh及openssh（ssh是**加密通信**的远程控制协议）

​	ssh是一个远程连接通信协议

​	openssh 是实现了ssh协议的一个具体的软件





## Telnet



### 实验



客户端

```
[root@localhost ~]# yum install telnet
[root@localhost ~]# yum install telnet-server ————————————————————————————————————————下载telnet

[root@localhost ~]# telnet 192.168.44.99 —————————————————————————————————————————————远程连接
xgz login: root
Password:
[root@xgz ~]#
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016102708056.png" alt="image-20261016102708056" style="zoom:67%;" />

明文传输，密码暴露在外，吓哭了😨



## SSH





### 实验

*   密码验证登录

```
[root@localhost ~]# ssh root@192.168.44.99 ——————————————————————————————————————远程连接，走密码验证（root是要登录的账户）

[root@localhost ~]# ssh -V ——————————————————————————————————————————————————————查看版本信息
OpenSSH_9.9p1, OpenSSL 3.5.5 27 Jan 2026

[root@localhost ~]# vim /etc/hosts ——————————————————————————————————————————————修改本地域名解析文件
[root@localhost ~]# ssh root@www.xgz.com ————————————————————————————————————————域名远程连接

```







*   免密登录（客户端）

```
[root@localhost ~]# cd ~/.ssh

[root@localhost .ssh]# ssh-keygen ————————————————————————————————————————————————————————————用来生成 SSH 密钥对

Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519): ———————————————————密钥存在了这个路径
Enter passphrase for "/root/.ssh/id_ed25519" (empty for no passphrase): ————————要给密钥设密码吗（不设就回车）
Enter same passphrase again: ———————————————————————————————————————————————————再输一遍
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:ubQF7mBOpxxfNrcrUgyFqAidO0xOvMLbyqhBbalSH6M root@localhost.localdomain
The key's randomart image is:
+--[ED25519 256]--+
| o .   . .       |
|. *   . . .      |
|.* + .  ..       |
|..O o  ..o       |
| o+=o = So= .    |
|.oo+ B O *oo .   |
|=.E . + =.  .    |
|o+      . .  .   |
|o        . ..    |
+----[SHA256]-----+
[root@localhost .ssh]# ls
id_ed25519  id_ed25519.pub  known_hosts  known_hosts.old ————————————————————————一个是私钥，一个是公钥，要把公钥给服务器
[root@localhost .ssh]# cd ~

[root@localhost ~]# ssh-copy-id root@192.168.44.99 ——————————————————————————————————————————————把你本地的公钥（id_ed25519.pub）复制到远程服务器 192.168.44.99 的 /root/.ssh/authorized_keys 文件里。

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.44.99's password:
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'root@192.168.44.99'"
and check to make sure that only the key(s) you wanted were added.
```

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016114649466.png" alt="image-20261016114649466" style="zoom:67%;" />





### 理论







#### 认证前

**1，ssh@xxx.xxx.xxx.xxx后、输入密码文本弹出前发生的事**

<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016150810807.png" alt="image-20261016150810807" style="zoom:80%;" />



1.  **固定钥匙（host公钥 / host私钥）**：服务器出厂就自带的，永久不变，用来**证明“我是这台服务器，不是冒牌货”**。
2.  **临时钥匙（临时公钥 / 临时私钥）**：为了这一次连接，双方现造现用的，用完就扔，用来**商量出一个只有你俩知道的“共享密码”**。



**3.1 你造临时钥匙，把公钥寄出去**
你的电脑现场生成一对“临时钥匙”：临时公钥 c、临时私钥 c。
你把**临时公钥 c** 通过网络寄给服务器。（临时私钥 c 永远留在你电脑里，不寄。）

**3.2 服务器造临时钥匙，算出共享密码**
服务器收到你的临时公钥 c 后，也现场生成一对“临时钥匙”：临时公钥 s、临时私钥 s。
然后服务器在本地算：
**服务器的临时私钥 s + 你的临时公钥 c = 共享密码**
（这个共享密码，服务器自己记住了，没发给任何人。）

**3.3 服务器签个名，证明自己是真身**
服务器在本地生成一个哈希值 H（相当于这次通信的摘要），然后拿出自己的**固定私钥（host私钥）**，给 H 签了个名。
（这个签名，只有用服务器的固定公钥才能验，别人伪造不了。）

**3.4 服务器把三样东西寄回给你**
服务器通过网络寄回：

1.  服务器的**临时公钥 s**
2.  服务器的**固定公钥（host公钥）**
3.  **签了名的 H**

**3.5 你在本地算共享密码**
你的电脑收到**服务器的临时公钥 s** 后，在本地算：
**你自己的临时私钥 c + 服务器的临时公钥 s = 共享密码**
（奇迹发生了：你算出来的共享密码，和服务器 3.2 算出来的一模一样。）

**3.6 你在本地验证签名**
你的电脑打开本地的 `~/.ssh/known_hosts` 文件，取出你第一次连它时存下来的**服务器固定公钥**。
用这把公钥，去解服务器寄来的“签了名的 H”：

-   解得开 → 证明对方真的是那台服务器，不是黑客冒充的。
-   解不开 → 报警，断开连接。

**3.7 双方启用共享密码，开始加密聊天**
你的电脑和服务器说：“好了，暗号对完了。”
从这一刻起，双方用刚才算出来的“共享密码”加密所有通信。
之后 SSH 让你输密码，就是在**这条已经加密的隧道里**传输的。



------------------------------------------

就是说，我给对面临时公钥，对面会结合自己的临时公钥和临时私钥算出来共享密钥，

之后，对面会把他的临时公钥、固定公钥、用固定私钥签名的H给我，我也可以用他给我的临时公钥结合自己的临时公钥和临时私钥算出来共享密钥，

最后，看固定公钥（第一次交流后就存在我这里了）是否匹配这个有固定私钥签名的H，不匹配就说明有人冒充

（会用共享密钥对数据进行加密）

------------------------------------------------------



结论：也就是说，只要算出那个共享密钥一致，并且自己这边的服务器公钥还对的上对面发来用私钥签的纸，就代表对方不是冒牌的





#### 认证



两种认证方式



##### 密码认证



```
这是shadow文件中的一个加密码
$6 他是哈希算法的类型 哈希算法 常见有 md5 sha512 sha256 sha334
$umLFOxV6OrqDdtqL # 盐值，混淆原始密码的一段随机数据(将盐值和和原始密码混合在一起，然后进行哈希加密变成下面这一坨)
$OPRt8sPIcjT7gfMof5G2Lsiy2/wL5Le/C1dSBoF5vZteyyhP7Yq /Ww3NqtbOFy1veYK2DT0Z.oT00BEzbp05J.
```





<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016152000529.png" alt="image-20261016152000529" style="zoom:67%;" />





##### 密钥认证



其实就是第一次连接时，客户端把公钥给了服务器，服务器记住了之后，以后认证时客户端发送公钥，服务器就只要核对下看看有没有就行了



<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016183641691.png" alt="image-20261016183641691" style="zoom:67%;" />



<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261016192550093.png" alt="image-20261016192550093" style="zoom:80%;" />

右边是客户端，左边是服务器，这两个文件是一样的

（所以说，要是手动把客户端的公共密钥添加到左边文件中，甚至可以实现无需输入密码即可连接）













### 配置文件





```
[root@xgz .ssh]# cd /etc/ssh/sshd_config

[root@xgz ssh]# cd ~/.ssh
```





/etc/ssh/sshd_config

```
AllowUsers root ————————————————————————————————————————这一行可以自己添加的，白名单只有root

21 Port 22 —————————————————————————————————————————————ssh服务程序的默认端口号
22 AddressFamily any
23 ListenAddress 0.0.0.0 ———————————————————————————————ssh服务程序 监听的ipv4
24 ListenAddress :: ————————————————————————————————————ssh服务程序 监听的ipv6

————————————————————————————————————————————————————————主机私钥的存放位置
26 HostKey /etc/ssh/ssh_host_rsa_key
27 HostKey /etc/ssh/ssh_host_ecdsa_key
28 HostKey /etc/ssh/ssh_host_ed25519_key

39 LoginGraceTime 2m ———————————————————————————————————登录宽容时间（2m不输密码就断开）
40 PermitRootLogin prohibit-password ———————————————————是否允许root账户登录 | 只允许密码 只允许公钥（为什么禁用密码登录不起作用，应为另一个子文件/etc/ssh/sshd_config.d里面允许密码登录，并且优先级更高）
41 StrictModes yes
42 MaxAuthTries 6
43 MaxSessions 10 ——————————————————————————————————————同时允许几个客户端连接到当前虚拟机

49 AuthorizedKeysFile .ssh/authorized_keys —————————————选择基于密钥验证时，客户端生成一对公私钥之后，会将公钥放到.ssh/authorizd_keys里面

65 #PasswordAuthentication yes —————————————————————————是否允许普通账户密码登录
66 #PermitEmptyPasswords no ————————————————————————————是否允许空密码登录
```





| 参数类别（上面第四十行）                 | 是否允许ssh登录 | 登录方式   | 交互shell  |
| :--------------------------------------- | :-------------- | :--------- | :--------- |
| yes                                      | 允许            | 无限制     | 无限制     |
| no                                       | 不允许          | 无         | 无         |
| prohibit-password                        | 允许            | 禁用密码   | 无限制     |
| forced-commands only（不常用，不要看了） | 允许            | 仅允许密钥 | 授权的口令 |






















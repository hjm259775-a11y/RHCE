# SELinux



作用：

*   SELinux域限制：对服务程序的功能进行限制，以确保服务程序做不了出格的事情

*   SELinux安全上下：对件资源的访问限制，确保件资源只能被其所属的服务程序访问



/etc/selinux/config文件里的

| 模式           | 中文叫法      | 核心行为                                                     | 适用场景                         | 如何切换（临时）                                        |
| :------------- | :------------ | :----------------------------------------------------------- | :------------------------------- | :------------------------------------------------------ |
| **Enforcing**  | 强制模式      | **拦截**并**记录**。严格执行安全策略，只要违反规则，直接拒绝（如报 `Permission denied`），并写入日志。 | 生产环境（默认且最安全）         | `setenforce 1`                                          |
| **Permissive** | 宽容/警告模式 | **只记录**，**不拦截**。即使违反策略，也放行操作，但会在日志里疯狂输出警告（方便排错）。 | 调试排错阶段（想看是什么被拦了） | `setenforce 0`                                          |
| **Disabled**   | 彻底关闭      | **完全不生效**。内核根本不加载 SELinux 模块，系统毫无察觉。  | 学习实验环境/极度追求性能的场景  | 修改 `/etc/selinux/config` 为 `disabled` 后**必须重启** |





*   开启selinux

```
[root@xgz ~]# vim /etc/selinux/config
————————————————————————————————
 22 SELINUX=permissive
————————————————————————————————
[root@xgz ~]# reboot
[root@xgz ~]# vim /etc/selinux/config
————————————————————————————————
 22 SELINUX=enforcing
————————————————————————————————
 [root@xgz ~]# reboot

```



*   关闭selinux

```
[root@xgz ~]# vim /etc/selinux/config
————————————————————————————————
 22 SELINUX=disabled
————————————————————————————————
 
```







*   临时修改

```
[root@xgz ~]# setenforce 0 ——————————————————————改为Permissive
setenforce: SELinux is disabled
[root@xgz ~]# setenforce 1 ——————————————————————改为Enforcing
setenforce: SELinux is disabled
[root@xgz ~]# getenforce
Disabled
```









```
[root@xgz ~]# yum install setools-console -y ————————————————————————————下载安装
```











## 安全上下文



### ls -Z



*   查看安全上下文

```
[root@xgz ~]# ls -Z anaconda-ks.cfg
system_u: # 用户身份
object_r: # 角色身份 以上两个字段几乎不参与selinux限制，他们的作用标记文件的身份和角色
selinux_config_t: # 安全上下文的类型 他决定了该资源能否被特定进程所访问
s0 /etc/selinux/config # 安全级别
```





### chcon

**暂时修改**

*   修改安全上下文

```
chcon [-R] [-t type] [-u user] [-r role] [-v] 文件名
```

| 参数 | 说明                 |
| :--: | :------------------- |
|  -R  | 递归修改             |
|  -t  | 修改文件的上下文类型 |
|  -u  | 修改文件的上下文用户 |
|  -r  | 修改文件的上下文角色 |



*   修改安全上下文（更方便于将范例文件的安全上下文完全复制到别的文件上）

```
chcon -R --reference=范例文件 文件名
```









因为文件的安全上下文类型太多了除了安全研究员一般人很难知道这些类型被哪些进程访问是合规的。

所以我们在使用修改安全上下文类型功能时会参考该进程本身就可以访问的资源他们具备怎样安全上下文类型。

（就是看看进程正常访问的资源是什么安全上下文类型就行了，然后你就复制给自己创建的目录和文件）





```
[root@server ~]# chcon -Rt httpd_sys_content_t /web/ ——————————————————————————————递归修改/web/中的安全上下文类型 
[root@server ~]# ls -Z /web/
unconfined_u:
object_r:
httpd_sys_content_t:
s0 index.html
```





### seinfo



```
[root@xgz ~]# seinfo -u ——————————————————————————————————查看所有用户
Users: 8
   guest_u
   root
   staff_u
   sysadm_u
   system_u
   unconfined_u
   user_u
   xguest_u
[root@xgz ~]# seinfo -r ——————————————————————————————————查看所有角色
[root@xgz ~]# seinfo -t | wc -l ——————————————————————————查看所有类型
```







### restorecon



*   让文件的SELinux类型恢复为默认的SELinux类型

*   默认的SELinux类型与semanage命令有关，其参考semanage命令所查询的默认SELinux类型



```
restorecon [-Rv] 文件或目录


[root@server ~]# restorecon -R /web2
[root@server ~]# ls -Zd /web2
system_u:object_r:default_t:s0 /web2
```







### semanage



*   用于管理SELinux的策略,查询/修改/增加/删除文件的默认SELinux安全上下文，管理网络端口、消息接口等



*   安装

```
[root@server ~]# yum install policycoreutils-python-utils -y
```



格式1：

```
semanage    选项    参数    文件
```



选项：login  |  user  |  port  |  interface  |  fcontext  |  translation  |  boolean ，注意：fcontext查询默认安全上下文（重要）

| port     | 查找端口号的安全上下文 |
| -------- | ---------------------- |
| fcontext | 查找文件的安全上下文   |





参数

| 参数   | 英文全称   | 核心作用                                                 |
| :----- | :--------- | :------------------------------------------------------- |
| **-l** | list       | 列出（查询）现有的规则                                   |
| **-a** | add        | 添加一条新的规则                                         |
| **-m** | modify     | 修改一条已经存在的规则                                   |
| **-d** | delete     | 删除一条规则                                             |
| **-D** | Delete all | 删除**所有**本地自定义的规则（慎用）                     |
| **-t** | type       | 指定安全上下文的类型（Type），比如 `httpd_sys_content_t` |
| **-p** |            | 协议类型                                                 |





示例：

```
# 1.修改nginx的主配置文件 让server监听8888 此时浏览器验证 172.25.254.100:8888服务无法打开
[root@server ~]# vim /etc/nginx/nginx.conf
....
38      server {
39              listen          8888;
....
[root@server ~]# ngxin -s reload
# 2.使用semanage 对http port 进行修改 将8888加入允许列表
# 2.1 查看http允许的端口范围
[root@server ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
# 2.2 将8888 加入到允许的端口中
[root@server ~]# semanage port -a -t http_port_t -p tcp 8888
# 2.3 添加后观察是否成功
[root@server ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      8888, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
# 3.重启nginx观察是否可以访问172.25.254.100:8888
systemctl restart nginx
```



示例：

```
[root@server ~]# vim /etc/ssh/sshd_config  # 修改ssh的端口号为2222
Port 2222

[root@server ~]# systemctl restart sshd   # 重启服务失败
Job for sshd.service failed because the control process exited with error code.
See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.

[root@server ~]# semanage port -l | grep ssh_port_t  # 查看ssh的端口号策略
ssh_port_t                     tcp      22
[root@server ~]# semanage port -a -t ssh_port_t -p tcp 2222  # 策略中添加新端口
[root@server ~]# semanage port -l | grep ssh_port_t
ssh_port_t                     tcp      2222, 22
[root@server ~]# systemctl restart sshd

# win11中使用   ssh  root@192.168.88.130:2222  进行访问
```












































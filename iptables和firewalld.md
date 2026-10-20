# 防火墙



*   硬件防火墙：由厂商设计好的主机硬件，其操作系统主要以提供数据包数据的过滤机制为主，并去掉不必要的功能

*   软件防火墙：保护系统络安全的一套软件（或称为机制），如Netfilter（数据包过滤机制)





作用：

*   拒绝让Internet的数据包进入主机的某些端口

*   拒绝某些来源IP的数据包进入
*   拒绝让带有某些特殊标志（flag）的数据包进入，如：带有SYN的主动连接标志
*   分析MAC地址决定是否连接

## iptables

默认所有动作都是允许的，要是什么都没有也是放行





<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261020090623247.png" alt="image-20261020090623247" style="zoom: 80%;" />



**五链：**

-   INPUT链：数据包流入时，即数据包从内核流入用户空间
-   OUTPUT链：向外发送数据包(流出)时，即数据包从用户空间流出到内核空间，一般不配置
-   FORWARD链：处理数据包转发时，即在内核空间中，从一个网络接口进入，到另一个网络接口去（转发过滤）
-   PREROUTING链：在对数据包作路由选择之前，即互联网进入局域网
-   POSTROUTING链：在对数据包作路由选择之后，即局域网出互联网







**四表**（越靠前优先级越高）

-   raw表：决定是否进入连接跟踪。 new establish related
-   mangle：修改数据包内容/标记。
-   nat：做地址转换（SNAT/DNAT）。
-   filter：最终的防火墙决策。







**iptables 流量处理动作**

-   ACCEPT：允许流量通过
-   REJECT：拒绝流量通过，拒绝后回复拒绝信息
-   LOG：记录日志信息
-   DROP：拒绝流量通过，流量丢弃不响应





### 命令



*   安装

```
dnf install iptables-nft-services
systemctl disable --now firewalld
systemctl mask firewalld
systemctl enable --now iptables
```





**原则**

· iptables 命令根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配，若匹配成，则iptables 会根据策略规则所预设的动作来处理这些流量，由于策略规则的匹配顺序是从上至下，则要把较为严格、优先级较高的策略规则放到前面，以免发生错误

（自上而下，哪条匹配上了就不看下面的了）



**格式：**

· 简化：

```
iptables [-t 表名] 选项 [链名] [条件] [-j 控制动作]
iptables [操作] [链名] [条件] -j [动作]
```



· 详细：

```
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> <--sport 源端口> <-d 目标IP/目标子网> <--dport 目标端口> -j 控制动作
```

-   **`iptables -t 表名`**：指定操作哪张表（filter/nat/mangle/raw）。

-   **`<-A/I/D/R>`**：**必须选一个**。

    -   `-A` 追加到末尾
    -   `-I` 插入到头部
    -   `-D` 删除某条
    -   `-R` 替换某条

-   **`规则链名`**：你要操作的链（INPUT/FORWARD/OUTPUT等）。进站。出站。过境。

    **`[规则号]`**：**可选**。如果你用 `-I`（插入）、`-D`（删除）、`-R`（修改），通常要写个数字，比如 `1` 代表第一行。

    **`<-i/o 网卡名>`**：**二选一**。

    -   `-i` (input)：流量从哪块网卡**进来**（如 `-i eth0`）。
    -   `-o` (output)：流量从哪块网卡**出去**。

-   **`-p 协议名`**：匹配什么协议（如 `tcp`、`udp`、`icmp`）。

    **`<-s 源IP/源子网>`**：**可选**。匹配谁发来的（如 `-s 192.168.1.100`）。

    **`<--sport 源端口>`**：**可选**。发送方的端口（一般很少限制源端口）。

    **`<-d 目标IP/目标子网>`**：**可选**。匹配发给谁的（如 `-d 192.168.1.10`）。

    **`<--dport 目标端口>`**：**可选**。访问的服务端口（如 `--dport 80`，**最常用**）。

-   **`-j 控制动作`**：**必填**。怎么处理（`ACCEPT`放行、`DROP`丢弃、`REJECT`拒绝）。





```
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited ——————————————————————————在最后设置一条拒绝所有
```









这张表越看越混😨

| 参数          | 作用                                                         |
| :------------ | :----------------------------------------------------------- |
| **-t**        | 对指定的表进行操作，table必须是raw，nat，filter，mangle中的一个，默认是filter |
| **-p**        | 指定要匹配的数据包协议类型                                   |
| **-s**        | 匹配源地址IP/MASK，若有 ! 表示取反                           |
| **-d**        | 匹配目的地址IP/MASK                                          |
| **-i 网卡名** | 匹配从这块网卡流入的数据                                     |
| **-o 网卡名** | 匹配从这块网卡流出的数据                                     |
| **-L**        | 列出规则链上的所有规则，如果没有指定链，列出表上所有链的所有规则 |
| **-A**        | 在规则链的末尾加入新规则                                     |
| **-I num**    | 在规则链的头部加入新规则                                     |
| **-D num**    | 删除指定规则                                                 |
| **-R num**    | 替换/修改第几条规则                                          |
| **-P**        | 设置默认策略                                                 |
| **-F**        | 清空所有规则(F:flush是“冲洗、冲掉”的意思)                    |
| **-N**        | 创建新规则链                                                 |
| **-X**        | 删除指定规则链，这个链必须没有被其它任何规则引用，而且这条链上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链 |
| **-E**        | 用指定的新名字去重命名指定的链                               |
| **-Z**        | 把指定链，或者表中的所有链上的所有计数器清零                 |
| **-j**        | 满足某条件时该执行什么样的动作                               |
| **-h**        | 显示帮助信息                                                 |













*   查看

```
[root@server ~]# iptables -nvxL --line
# -n 显示源   -v 详细信息  -x 自动转换为KB/MB等单位  -L写在最后列出所有规则  --line 增加行号

[root@server ~]# iptable -nL : n表示显示数字（不把 IP 解析成域名） -L表示查看表详细规则
```



*   永久保存

```
[root@server ~]# iptables -F # 清空规则
[root@server ~]# iptables -nL # 查看规则链
[root@server ~]# systemctl restart iptables
[root@server ~]# iptables -nL # 由于未保存，则恢复原状

[root@server ~]# iptables -F                      # 再次尝试清空规则链
[root@server ~]# service iptables save            # 保存服务状态
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[root@server ~]# systemctl restart iptables       # 重启服务
[root@server ~]# iptables -nL                     # 查看
```

```
[root@server ~]# service iptables save ——————————————————————————————在修改完iptables后执行，可永久保存
```











*   修改iptables（临时）

```
iptable -nL : n表示显示数字（不把 IP 解析成域名） -L表示查看表详细规则

iptables [-t 指定表] -A/-I/-D/-R 指定链 [-s 来源ip] -p 协议 [--dport 目标端口] -j [ACCEPT/REJECT/DROP]

# 拒绝 172.25.254.200的规则
iptables -I INPUT 1 -s 172.25.254.200 -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -s 172.25.254.200 -p tcp --dport 22 -j REJECT

-A:表示在某条链上 添加 规则
-I num:表示在某条链上 插入 规则
-D num:表示删除某条链上的某个规则
-R num:表示修改某条链上 第几个规则

-s ：允许/拒绝 ip
-p ：在规则中添加允许/拒绝的协议
--dport ：允许/拒绝的端口
-j ACCEPT/DROP/REJECT：表示接受/丢弃/拒绝

-P ACCEPT/DROP：修改某条链是否允许或者丢弃流量
-F ：清空所有规则
```







示例：（搭建web服务器，设置任何人都能通过80端口访问http）

```
[root@xgz www]# vim /etc/nginx/nginx.conf
————————————————————————————————————————————————————————————————————————————————————————————
server{
        listen 80;
        server_name 192.168.44.99;
        root /www;
        index lelele.html;
}
————————————————————————————————————————————————————————————————————————————————————————————
[root@xgz www]# systemctl restart nginx.service ——————————————————————开启

[root@xgz www]# iptables -F ——————————————————————————————————————————先把原本的设置临时删了
[root@xgz www]# iptables -A INPUT -p tcp --dport 80 -j ACCEPT ————————加入允许80端口
[root@xgz www]# iptables -nL —————————————————————————————————————————查看下
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80

[root@xgz www]# setenforce 0 —————————————————————————————————————————临时关下SELinux


```







示例：（禁止某主机（192.168.44.9）ssh远程登录）

```
[root@xgz ~]# iptables -F
[root@xgz ~]# iptables -A INPUT -p tcp -s 192.168.44.9 --dport 22 -j REJECT
```



示例：（屏蔽所有ping）

```
[root@xgz ~]# iptables -A INPUT -p icmp -j REJECT
```













## firewalld



-   firewalld可以动态修改单条规则，不需要像iptables那样，修改规则后必须全部刷新才可生效
-   firewalld默认动作是拒绝，则每个服务都需要去设置才能放行，而iptables里默认是每个服务是允许，需要拒绝的才去限制
-   iptables防火墙类型为静态防火墙firewalld 防火墙类型为动态防火墙
-   firewalld和iptables一样自身并不具备防火墙功能，它们的作用都是用于维护规则，而真正使用规则干活的是内核防火墙模块
-   firewalld 加入了区域（zone）概念





*   启动！

```
[root@xgz ~]# systemctl disable --now iptables.service

[root@xgz ~]# systemctl unmask firewalld.service

[root@xgz ~]# systemctl enable --now firewalld.service
```









### 命令





firewall-cmd的子命令虽然有102个 但是无非就是



端口 (port) 

ip (source) 

服务(service) 

接口(interface) 

区域(zone) 



增(add)

删(remove) 

改 (set,change) 

查(info,list,query,get)



|             参数              | 作用                                                 |
| :---------------------------: | :--------------------------------------------------- |
|      --get-default-zone       | 查询默认的区域名称                                   |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效                         |
|          --get-zones          | 显示可用的区域                                       |
|        --get-services         | 显示预先定义的服务（系统自带的）                     |
|      --get-active-zones       | 显示当前正在使用的区域与网卡名称                     |
|         --add-source=         | 将源自此 IP 或子网的流量导向指定的区域               |
|       --remove-source=        | 不再将源自此 IP 或子网的流量导向某个指定区域         |
|  --add-interface=<网卡名称>   | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联                             |
|          --list-all           | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
|       --list-all-zones        | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
|    --add-service=<服务名>     | 设置默认区域允许该服务的流量                         |
|   --add-port=<端口号/协议>    | 设置默认区域允许该端口的流量                         |
|   --remove-service=<服务名>   | 设置默认区域不再允许该服务的流量                     |
|  --remove-port=<端口号/协议>  | 设置默认区域不再允许该端口的流量                     |
|           --reload            | 将硬盘中的设定加载到内存中                           |



```
[root@server ~]# firewall-cmd --zone=block --list-all --permanent ————————————————————最后加上--permanent写入数据库
```







```
firewall-cmd --set-default-zone=drop ——————————————————————————————设置当前区域为丢弃区，此时防火墙会丢弃所有的来访流量

firewall-cmd --zone=trusted --add-source=172.25.254.2 —————————————将指定ip添加到信任区，添加进来的这台设备可以访问当前设备任意端口

firewall-cmd --zone=trusted --remove-source=172.25.254.2 ——————————如果不再想要指定ip存在于信任区，可以使用remove移出
```













**富规则**：它的主要作用是精细化控制，如对指定ip能访问的端、服务、icmp进具体控制

------------------------------------------------------



<img src="C:/Users/xgz24/AppData/Roaming/Typora/typora-user-images/image-20261020171620352.png" alt="image-20261020171620352" style="zoom:50%;" />

rich rules就是富规则







😨这期神了😨

````
添加规则
firewall-cmd [--zone=<区域>] --add-rich-rule='<规则>' [--timeout=<时间>]

删除规则
firewall-cmd [--zone=<区域>] --remove-rich-rule='<规则>'

查询规则是否存在
firewall-cmd [--zone=<区域>] --query-rich-rule='<规则>'


[ family="ipv4" | family="ipv6" ]
[ priority="<数字>" ]    # 数字范围 -32768 - 32767, 默认为0
[ source [ address="<IP地址或CIDR网段>" | mac="<MAC地址>" | ipset="<IP集名称>" ] [ invert="true" | invert="false" ] ]
[ destination address="<IP地址或CIDR网段>" [ invert="true" | invert="false" ] ]

# ---------- 以下七种匹配条件（必须且只能选一种） ----------（括号和竖线不是命令，只是方便查看
(
    service name="<服务名>" | ——————————————————————————————————一种
    port port="<端口号或范围>" protocol="tcp" | —————————————————二种
    port port="<端口号或范围>" protocol="udp" |
    port port="<端口号或范围>" protocol="sctp" |
    port port="<端口号或范围>" protocol="dccp" |
    source-port port="<端口号或范围>" protocol="tcp" | ——————————三种
    source-port port="<端口号或范围>" protocol="udp" |
    source-port port="<端口号或范围>" protocol="sctp" |
    source-port port="<端口号或范围>" protocol="dccp" |
    protocol value="<协议名称或协议号>" | ————————————————————————四种
    icmp-block name="<ICMP类型名称>" | ——————————————————————————五种
    masquerade | ———————————————————————————————————————————————六种
    forward-port port="<外部端口号>" protocol="tcp" to-port="<内部端口号>" to-addr="<内部IP地址>" | ————————七种
    forward-port port="<外部端口号>" protocol="udp" to-port="<内部端口号>" to-addr="<内部IP地址>"
)




# ---------- 可选日志和审计 ----------
[ log
    [ prefix="<自定义前缀文本>" ]
    [ level="emerg" | level="alert" | level="crit" | level="error" | level="warning" | level="notice" | level="info" | level="debug" ]
    [ limit value="<数字>/<单位>" ]   # 单位可以是 s(秒)、m(分)、h(时)，例如 "5/m"
]
[ audit ]


# ---------- 以下四种动作（必须且只能选一种） ----------
(
    accept |
    reject [ type="<拒绝类型>" ] |    # 拒绝类型可选：icmp-net-unreachable, icmp-host-unreachable, icmp-port-unreachable, icmp-proto-unreachable, icmp-net-prohibited, icmp-host-prohibited, icmp-admin-prohibited
    drop |
    mark [ set="<标记值>" ] [ mask="<掩码值>" ]    # 标记值和掩码为数字，用于后续策略路由
)
````

这是地球上最丑陋的命令形式，没有之一，所以你还是在文本编辑器里写好了再粘贴过来吧







*   示例（允许（accept）来自 IP 地址 172.25.254.200 的流量，访问本机的 22 端口（SSH），使用的协议是 TCP）

```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="172.25.254.200" port port="22" protocol="tcp" accept'
```


































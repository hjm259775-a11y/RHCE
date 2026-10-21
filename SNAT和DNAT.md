

# SNAT和DNAT





SNAT：源地址转换

DNAT：目标地址转换





1.  **第 1 台机器（Client / 客户端）：**
    -   **IP：** `172.25.254.200`
    -   **网络模式：** **NAT模式**（模拟外部网络，也就是“公网”）。
    -   **作用：** 假设它是外网的一台客户机，用来去访问服务器。
2.  **第 2 台机器（Server / 服务器 / 网关）：**
    -   **IP：** `172.25.254.100`（对外） 和 `192.168.10.100`（对内）。
    -   **网络模式：** **双网卡**（一块连 NAT 外部网络，一块连仅主机内部网络）。
    -   **作用：** 它是核心！它扮演“路由器”或“防火墙”的角色，也就是做 **NAT 转换**的地方。它一边连着外网（172网段），一边连着内网（192网段）。
3.  **第 3 台机器（hostonly / 内网主机）：**
    -   **IP：** `192.168.10.200`
    -   **网络模式：** **仅主机模式**（模拟内部网络，也就是“私网”）。
    -   **作用：** 假设它是公司内网深处的一台服务器（比如你的数据库或私有云），它没有公网 IP，不能直接连外网。









## SNAT 源地址转换

```


# 1.设定slave的网络
# 1.1.将网络适配器修改为host-only
# 1.2.使用nmcli设定网络
nmcli con mod ens160 ipv4.method manual ipv4.address xxx.xxx.xxx.200/24 ipv4.gateway xxx.xxx.xxx.2 ipv4.dns 8.8.8.8
nmcli con up ens160

# 2.设定server网络
[root@server ~]# nmcli connection add type ethernet ifname ens224 con-name ens224 ipv4.method manual ipv4.addresses xxx.xxx.xxx.100/24 ipv4.gateway xxx.xxx.xxx.2 ipv4.dns 8.8.8.8
[root@server ~]# nmcli connection up ens224

# 3.设定client ：因为本身网络就符合设定所以此处无需修改

# 4.开启server内核转发
[root@server ~]# echo "net.ipv4.ip_forward=1">> /etc/sysctl.conf
# 4.1 是配置生效
[root@server ~]# sysctl -p
net.ipv4.ip_forward = 1
# 4.2 查看系统规则是否真的生效
[root@server ~]# sysctl -a | grep ip_forward
net.ipv4.ip_forward = 1
net.ipv4.ip_forward_update_priority = 1
net.ipv4.ip_forward_use_pmtu = 0

# 5.在server添加源地址伪装的规则，只能添加到postrouting这条链上，添加到prerouting可能会导致流量出口错误
[root@server ~]# systemctl disable --now firewalld.service
[root@server ~]# iptables -t nat -A POSTROUTING -o enp0s6 -j SNAT --to-source 1.2.3.4
[root@server ~]# iptables -t nat -nL
...
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
SNAT       0    --  0.0.0.0/0            0.0.0.0/0            to:1.2.3.4
```







DNAT目标地址转换

```
6.将进入路由器的流量转移到指定的设备上
[root@server ~]# iptables -t nat -A PREROUTING -i enp0s5 -j DNAT --to-dest 192.168.10.200
```







## firewalld的SNAT和DNAT

```
# SNAT
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s xxx.xxx.xxx.xxx -o eth0 -j MASQUERADE
firewall-cmd --reload

# DNAT
firewall-cmd --add-forward-port=port=8080:proto=tcp:toport=80:toaddr=xxx.xxx.xxx.xxx --permanent
firewall-cmd --reload
```
















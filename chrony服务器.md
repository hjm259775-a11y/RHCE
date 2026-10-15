# chrony服务器



## 前情提要

Linux的两个时钟

​	**硬件时钟RTC (Real Time Clock)**

即BIOS时钟，存储位置：电脑主板上一块独立小芯片，自带纽扣电池供电；

*   核心特点：关机断电后依然走时，拔掉主机电源线，电池仍能维持计时；
    作用

*   服务器 / 主机开机时，BIOS 读取 RTC 时间，把时间灌入系统软件时钟；

*   系统关机 / 重启时，可手动把当前系统时间写回 RTC，防止时间丢失；

*   独立于操作系统，Windows、Linux 都共用同一块 RTC 芯片。

    |  对比项  | 硬件时钟 RTC (CMOS)        | 系统软件时钟（内核时钟）             |
    | :------: | :------------------------- | :----------------------------------- |
    | 存储载体 | 主板芯片 + 纽扣电池        | 操作系统内存                         |
    | 断电表现 | 持续走时，不受关机影响     | 断电直接清零、停止计时               |
    |   精度   | 精度低，每日会漂移几秒     | CPU 定时器，毫秒级高精度             |
    | 读写速度 | 读写慢，不适合程序频繁读取 | 内存读取，几乎无消耗                 |
    | 谁会修改 | 手动 hwclock、BIOS 设置    | date、chrony、ntpd、程序             |
    | 业务依赖 | 仅开机初始化用，业务不读取 | 日志、数据库、定时任务、证书全部依赖 |
    | 时区标准 | 可存本地时间 / UTC         | 系统统一使用 UTC，再换算时区展示     |



### hwclock

```
[root@192 ~]# hwclock
查看硬件时间

[root@192 ~]# hwclock -s
使用RTC覆盖系统时间

[root@192 ~]# hwclock -w
使用系统时间覆盖RTC时间
```



```
[root@192 ~]# timedatectl
               Local time: 四 2026-10-15 15:52:30 CST
           Universal time: 四 2026-10-15 07:52:30 UTC
                 RTC time: 四 2026-10-15 07:52:30
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no



[root@192 ~]# date -s "2011-11-11 11:11:11" ———————————————————————————————修改系统时间为错误的时间
2011年 11月 11日 星期五 11:11:11 CST
[root@192 ~]# hwclock -w ——————————————————————————————————————————————————到此为止，系统时间和RTC时间全错
[root@192 ~]# timedatectl set-ntp 0 ———————————————————————————————————————关闭NTP时钟检查，目的是开启时获取一次同步
[root@192 ~]# timedatectl set-ntp 1 ———————————————————————————————————————开启NTP，立刻检查，发现系统时间又变正常了

至此，手动修改时间为错误时间，之后和NTP同步为正确时间
即通过授时中心获取标准时间进行本地同步



[root@192 ~]# timedatectl list-timezones ——————————————————————————————————查看所有时区
[root@192 ~]# timedatectl set-timezone Africa/Juba ————————————————————————将系统时间变为Africa/Juba的错误时间
[root@192 ~]# timedatectl set-timezone Asia/Shanghai ——————————————————————改回正常时间

至此，手动修改错误时区，之后又修改时区为正确时区
即通过修改时区获取标准时间进行本地同步
```

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20261015161151997.png" alt="image-20261015161151997" style="zoom:80%;" />

可以发现，RTC时间只能通过`hwclock  -w`和`timedatectl set-time "2011-11-11 11:11"`来修改



### date

### timedatectl

```sql
[root@192 ~]# timedatectl set-time "2011-11-11 11:11"

[root@192 ~]# date -s "2011-11-11 11:11:11"

此为手动修改，但是有所差异
```

| 对比项               | `date -s`                                          | `timedatectl set-time`                                       |
| :------------------- | :------------------------------------------------- | :----------------------------------------------------------- |
| **工作层面**         | 直接修改内核中的系统软件时钟                       | 通过 systemd 的时间管理服务（timedated）来修改               |
| **是否受 NTP 影响**  | 不受，即使开着 NTP 也能强行改掉                    | 如果 NTP 同步开启着，会**直接报错拒绝**（必须先 `set-ntp no`） |
| **格式要求**         | 很灵活，`"2011-11-11 11:11:11"` 可以，只写日期也行 | 相对严格，通常需要完整的 `"YYYY-MM-DD HH:MM:SS"`             |
| **会不会同步改 RTC** | 默认**不会**，只改系统时钟，RTC 不动               | 系统时间和RTC时间一起修改                                    |
| **语法来源**         | 传统的 Unix/Linux 老命令                           | systemd 时代的新命令                                         |









## NTP



*   NTP：（Network Time Protocol，网络时间协议）是由RFC 1305定义的时间同步协议，用来在分布式时间服务器和时间同步。

*   **NTP基于UDP报文进行传输，使用的UDP端口号为123**

*   NTP可以对网络内所有具有时钟的设备进行时钟同步，使网络内所有设备的时钟保持一致，从而使设备能够提供基于应用，对于运行NTP的本地系统，既可以接受来自其他时钟源的同步，又可以作为时钟源同步其他的时钟，并且可同步。

*   NTP的其精度在局域网内可达0.1ms，在互联网上绝大多数的地方其精度可以达到1-50ms

*   传统 NTP 服务为老牌标准，缺点：

    ​	网络中断后时间漂移严重

    ​	时间偏差过大时会直接跳跃调整时钟（jump），业务会出现时间回拨，引发 MySQL、JWT、定时任务故障

    ​	云服务器、虚拟机适配差，不适合频繁休眠 / 快照环境。









## chrony



介绍：chrony就是关于ntp这套络时间协议实现出来的具体的软件







```
[root@192 ~]# dnf install chrony -y
[root@192 ~]# systemctl enable --now chronyd
[root@192 ~]# systemctl status chronyd

[root@192 ~]# vim /etc/chrony.conf ————————————————————————————————————————————这是配置文件

————————————————————————————————————————————————————————————————————————————————————————————————————
使用pool.ntp.org项目中的公共服务器，pool 会自动解析多个 IP，自动剔除故障节点，冗余性更强
或者使用server开头的服务器，理论上想添加多少时间服务器都可以
iburst：开机前 4 秒连续快速发送 4 个数据包，加速首次同步，服务器必加
pool pool.ntp.org iburst ——————————————————————————————————————————————————————重要

#记录硬件时钟漂移值，优化RTC走时精度
driftfile /var/lib/chrony/drift

当本机与标准时间偏差 超过 1 秒，且连续检测 3 次 都存在大偏差，才允许直接跳跃调整时间
makestep 1.0 3

启用实时时钟（RTC）的内核同步。
rtcsync
# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16 —————————————————————重要，这是允许享受本机始终服务的设备网段

# Serve time even if not synchronized to a time source.
#local stratum 10 —————————————————————————重要，总计16层0最级始终来源，1层顶级时钟来源，2-15次级时钟来源16未同步无效状态

# Require authentication (nts or key option) for all NTP sources.
#authselectmode require

# 指定包含NTP身份验证密钥的件。
# Specify file containing keys for NTP authentication.
# keyfile /etc/chrony.keys

# 指定日志文件的目录。
# Specify directory for log files.
# logdir /var/log/chrony

# 选择日志文件要记录的信息。
# Select which information is logged.
# log measurements statistics tracking
————————————————————————————————————————————————————————————————————————————————————————————————————


```







### 实验





环境配置方面，先关掉防火墙和禁用SELinux（客户端和服务器都需要）

```
[root@192 ~]# systemctl disable --now firewalld
关闭防火墙

[root@192 ~]# vim /etc/selinux/config
————————————————————————————————————————————————————————————————————————
22 SELINUX=disabled
————————————————————————————————————————————————————————————————————————
切换成宽容模式

[root@192 ~]# reboot
重启电脑以命令生效
```





*   客户端

```
[root@localhost ~]# vim /etc/chrony.conf


————————————————————————————————————————————————————————————————————————————————————————————————————
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
pool 192.168.44.99 iburst

# Use NTP servers from DHCP.
sourcedir /run/chrony-dhcp

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10
————————————————————————————————————————————————————————————————————————————————————————————————

[root@localhost ~]# timedatectl set-ntp 0
[root@localhost ~]# timedatectl set-ntp 1

```









*   服务器

```
[root@192 ~]# dnf install chrony -y
[root@192 ~]# systemctl enable --now chronyd
[root@192 ~]# systemctl status chronyd

[root@192 ~]# vim /etc/chrony.conf

————————————————————————————————————————————————————————————————————————————————————————————————————
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
# pool pool.ntp.org iburst ———————————————————————————————————加上#，直接把自动NTP关了，防止错误的时间传不到客户端

# Use NTP servers from DHCP.
sourcedir /run/chrony-dhcp

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
allow 0.0.0.0/0 ——————————————————————————————————————————————————————————————————————开启

# Serve time even if not synchronized to a time source.
local stratum 10 ————————————————————————————————————————————————————————————————————开启


————————————————————————————————————————————————————————————————————————————————————————————————————

[root@localhost ~]# systemctl restart chronyd
```












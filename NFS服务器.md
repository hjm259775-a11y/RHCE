# NFS服务器



**简介**

什么是NFS：net file system网络文件系统，它允许服务器去使用网络上的硬盘资源，使用起来在本地使用的体验是一样。

NFS的使用场景：使用网络上的共享磁盘时

NFS涉及到的服务：nfs-server、rpc















-   **配置文件**
    -   主配置文件`/etc/exports`
    -   共享控制`/usr/sbin/exportfs`

## 实验



### nfs基础部署

服务器

```
[root@xgz conf.d]# dnf install nfs-utils ——————————————————————————————————————————————————————————下载
[root@xgz conf.d]#  systemctl enable --now nfs-server.service —————————————————————————————————————运行（开机自启）

[root@xgz /]# mkdir weiyan
[root@xgz /]# echo "shudaoqianxian liangcaoruheyunsong" > /weiyan/youhebugan.txt ——————————————————这个文件准备共享出去
[root@xgz /]# vim /etc/exports 
——————————————————————————————————————————————————————————————————————————————————————
/weiyan 192.168.44.9/24(rw,sync,root_squash) ——————————————————————————————————————————————————对于这个目录有读写权限，改了东西立刻处理过去，并且隐藏客户端的root身份
——————————————————————————————————————————————————————————————————————————————————————


[root@xgz /]# chmod -rf 777 /weiyan/youhebugan.txt ————————————————————————————————————————————————给权限，方便别人修改（仅对当前这些文件生效，以后服务器新创建了的话还是权限不够）

[root@xgz weiyan]# exportfs -rv ———————————————————————————————————————————————————————————————————刷新/etc/exports文件

```



客户端

```
[root@localhost ~]# showmount -e 192.168.44.99 —————————————————————————————————————————————————查看192.168.44.99的文件
Export list for 192.168.44.99:
/weiyan 192.168.44.9/24



[root@localhost /]# mkdir /xiaoweiyan
[root@localhost /]# mount 192.168.44.99:/weiyan /xiaoweiyan/ ———————————————————————————————————挂载到/xiaoweiyan里面
[root@localhost /]# ll /xiaoweiyan
总计 4
-rw-r--r-- 1 root root 35 10月18日 19:43 youhebugan.txt

[root@localhost /]# echo "beifakonghaoguoli buruxiaofangdongwu" >> /xiaoweiyan/youhebugan.txt
-bash: /xiaoweiyan/youhebugan.txt: 权限不够
[root@localhost /]# echo "beifakonghaoguoli buruxiaofangdongwu" >> /xiaoweiyan/youhebugan.txt————在此文件中追加，服务器那边的/weiyan/youhebugan.txt也会跟着修改
```













 

### 配置文件



#### /etc/exports



/weiyan 0.0.0.0/0(rw,sync,root_squash)

你甚至可以这么写，别人只要安装了nfs，就能来你这看看



关于括号里的参数：

| 权限参数          | 作用                                                         |
| :---------------- | :----------------------------------------------------------- |
| ro                | 只读                                                         |
| rw                | 读写（最终还要看文件系统rwx权限）                            |
|                   |                                                              |
| root_squash       | 当NFS客户端以root账户访问时，映射为NFS服务器端的匿名账户（nobody）（创建文件的时候别人看不到是谁做的） |
| no_root_squash    | 当NFS客户端以root账户访问时，映射为root账户（不推荐、不安全）（别人的root过来也是root身份，权限过大） |
| all_squash        | 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名账户（nobody），==推荐== |
| no_all_squash     | 除了root，其他账户都不再隐匿                                 |
| anonuid= anongid= | 将文件的用户和工作组映射为指定UID和GID，若不指定则默认为65534（nobody）不管什么账户访问，都会被映射成这个用户和组 |
|                   |                                                              |
| sync              | 同步，同时将数据写入内存与硬盘中，保证数据不会丢失，==推荐== |
| async             | 异步，优先将数据保存到内存，然后再写入硬盘，效率高，但可能会数据丢失 |



```
/weiyan 0.0.0.0/0(rw,sync,root_squash,anonuid=1000,anongid=1000) ————————————————别的账户访问过来都会被映射为xgz账户
```







### 自动挂载 autofs





*   autofs：自动处理挂载问题的程序

​	配置文件/etc/autofs.conf

​		在这个配置文件中可以设置关于autofs这个程序的一些属性，比如超时多久未操作可以自动取消挂载

​	配置文件/etc/auto.master

​		这个配置文件用于设定目定义的自动挂载目录





**自动挂载和传统挂载的区别：**

1.   最核心特点：按需挂载（懒汉模式）

-   **fstab**：只要你开机，不管你今天用不用这个 NFS 目录，它都会立刻连上。
-   **autofs**：开机什么都不做。只有当你的鼠标双击了那个目录，或者你敲了 `cd` 命令进去时，它才会瞬间去连服务器并挂载。**你不碰它，它就不挂载。**

2.   自动卸载（不用了就走）

-   **fstab**：一旦挂载，除非你手动 `umount` 或者关机，否则永远占着那个位置。
-   **autofs**：你进去操作完了，退出来之后。如果你超过 5 分钟（默认时间）没再碰这个目录，`autofs` 会在后台悄悄执行 `umount`，把网络连接断开，释放系统资源。

3.   不死机保命（最实用的特点）

-   **fstab**：如果 NFS 服务器突然宕机了，你用 fstab 挂载的客户端，一重启或者敲 `ls` 访问，整个终端会**死锁卡住**，甚至导致系统开不了机。
-   **autofs**：如果 NFS 服务器宕机了，你的系统依然丝滑流畅。只有当你非要 `cd` 进那个挂载目录时，它才会提示你“连接超时”报错，但绝对不会拖累整个系统卡死。

4.   不暴露挂载状态（隐形挂载）

-   **fstab**：用 `df -h` 查看磁盘，你永远能看到那一长串 NFS 的挂载信息。
-   **autofs**：只要你没进去那个目录，`df -h` 里根本看不到它，非常干净。进去的那一瞬间，它才会出现在列表里。







```
[root@localhost /]# yum install autofs ——————————————————————————————————————————下载

[root@localhost /]# systemctl enable --now  autofs.service ——————————————————————运行

[root@localhost /]# vim /etc/autofs.conf ————————————————————————————————————————进去给人家刷新时间改了
[root@localhost /]# systemctl restart autofs.service

[root@localhost /]# ls
afs   dev   lib    misc  opt   run   sys  var
bin   etc   lib64  mnt   proc  sbin  tmp  xiaoweiyan
boot  home  media  net   root  srv   usr  youhebugan.txt
[root@localhost /]# ls net ——————————————————————————————————————————————————————看上去空无一物

[root@localhost /]# cd net/192.168.44.99 ————————————————————————————————————————但当你cd进去，就能看到加载出东西了

[root@localhost /]# tree net
net
└── 192.168.44.99
    └── weiyan
        └── youhebugan.txt

[root@localhost /]# tree net ——————————————————————————————————————————————时间一到，就被删了
net

```



在 `/etc/auto.master` 文件里，通常默认带着这么一行：`/net -hosts`

-   **`/net`**：是挂载点。
-   **`-hosts`**：这是一个特殊的魔法参数。它告诉 `autofs`：“**只要有人往 `/net` 目录里进，并且敲的是一个 IP 地址，你就去这个 IP 地址上问一句 `showmount -e`，看看它共享了什么，然后全自动帮它挂载过来。**”







### 指定挂载目录





```
[root@localhost /]# vim /etc/auto.master ————————————————————————设定目定义的自动挂载目录

————————————————————————————————————————————————————————————————————————————————————————————————————————————
 13 /net    -hosts
 14 /zhangqiying /etc/dianhua.d —————————————————————————————————自定义的挂载点是/zhangqiying关于挂载详细设定在dianhua.d中
————————————————————————————————————————————————————————————————————————————————————————————————————————————

[root@localhost /]# vim /etc/dianhua.d ——————————————————————————创建自定义配置文件
————————————————————————————————————————————————————————————————————————————————————————————————————————————
ziwei 192.168.44.99:/weiyan —————————————————————————————————————共享目录就是ziwei，后面是要共享的目录
————————————————————————————————————————————————————————————————————————————————————————————————————————————

[root@localhost etc]# systemctl restart autofs.service

[root@localhost /]# cd zhangqiying/
[root@localhost zhangqiying]# cd ziwei
[root@localhost ziwei]# ll ——————————————————————————————————————可以看到，成功了
总计 4
-rwxrwxrwx 1 root root 72 10月18日 20:16 youhebugan.txt
```





### 光盘挂载





修改autofs主配置文件

```
# 设计本地自动挂载目录路径：/media/cdrom

# 定位第7行，输入本地挂载路径的父目录和自配置文件
[root@server ~]# vim /etc/auto.master
/media  /etc/iso.aa
```



编写autofs子配置文件

```
[root@server ~]# vim /etc/iso.aa
cdrom  -fstype=iso9660,ro,nosuid,nodev  :/dev/sr0  # 冒号前有空格

# 注意cdrom为挂载路径/media/cdrom的子目录
# -ftype:为文件系统格式
# iso9660:为光盘设备格式
# ro:以只读模式挂载文件系统
# nodev:不解析文件系统上的块特殊设备
# nosuid:禁止suid操作和设定sgid位(SUID 是 Set User ID, SGID 是 Set Group ID的意思)
```






























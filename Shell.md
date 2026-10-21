# Shell





查看当前系统支持的shell

```
[root@server ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```



查看当前系统默认shell

```
[root@server ~]# echo $SHELL
/bin/bash
```





------------------------------------------------------------------------------------------

编程语言分类：

• 静态编译型
	• 编译型：代码无法直接运行而是先生成一个可执行文件，在windows常见就是.exe结尾的文件
• 动态解释型
	• 解释型：不会生成任何可执行文件，解释器会对你写好的脚本逐行翻译，交由CPU执行。







Shell（外壳）：是一种命令解释器程序，它能识别用户输入的各种命令，并传递给操作系统





*   众所周知。我们把命令敲进文件里面，之后`sh`这个文件可以执行里面的命令，并且如果里面存在`$1`和`${10}`还可以在执行文件时自定义输入，方便很多

*   如果觉得执行脚本需要打出脚本文件的绝对路径太麻烦，可以把它放到`PATH`环境变量里面，这样只需`sh 文件名`就行了（不需要加`.sh`）





查看当前系统的环境变量 `PATH` 到底包含哪些目录

```
[root@xgz ~]# echo $PATH
/root/.local/bin:
/root/bin:
/usr/local/sbin:
/usr/local/bin:
/usr/sbin:
/usr/bin
```









`sh-5.2#`开头是符合`posix`命令格式shell

```
[root@xgz ~]# sh
sh-5.2# bash
[root@xgz ~]# exit
sh-5.2# exit
[root@xgz ~]#
```


































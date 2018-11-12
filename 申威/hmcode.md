# hmcode烧入

> hmcode是申威架构独有的技术，继承自alpha。

## 1、配置hmcode烧入环境

> ​	为了能够使用hmcode烧入相关的一系列软件工具，我们需要一个Linux操作系统以及一个java环境。

​	创建一台虚拟机，安装java软件包，将mt.tgz文件解压到/root目录下，配置环境变量：

``` shell
$ tar xvf mt.tgz /root
$ ls
Maintance Maintance-6a
$ echo "# for 6A " >> /etc/bashrc
$ echo "export PATH=$PATH:/root/Maintance-6a/:/root/Maintance-6a/bin:/root/Maintance-6a/bin/2f:/root/Maintance-6a/swich " >> /etc/bashrc
$ echo "export CLASSPATH=/root/Maintance-6a/classes/server_md.jar:/root/Maintance-6a/classes/2F.jar:/root/Maintance-6a/classes/2F1.jar:/root/Maintance-6a/classes/comm.jar:." >> /etc/bashrc
$ echo $PATH
$ java -v
```

​	检查是否配置成功：

``` shell
$ readme
client version 2.17
```

## 2、扫描服务器维护端口号

​	连接服务器维护端口，端口bmc号（可以认为是一个固定的服务器ip）可以通过打开服务器机盖查看，也可通过脚本扫描。

``` shell
$ ./bmc.sh 0
$ ./bmc.sh 100
$ ./bmc.sh 200
```

> 后面的数字代表的意思是从哪个端口号开始扫描，灵活的使用可以缩短扫描时间。

​	获取到bmc号（假设为x）之后，将这个bmc号配置到hosts：

``` shell
$ echo "192.168.1.x bmcx" >> /etc/hosts
$ echo ping bmcx
```

## 3、烧入hmcode

​	配置好环境之后，烧入就简单多了：

``` shell
$ loadhmcode bmcx hmcode.nh
$ reboot
```

## 4、检查烧入结果


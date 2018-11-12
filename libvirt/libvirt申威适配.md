# libvirt申威平台适配

> 本文主要讲解libvirt在申威平台上的移植方法和需要注意的一些细节。目前已完成基本适配的版本为libvirt3.2+swvm和libvirt4.5.0+qemu2.11.0。

## 1、申威平台上编译libvirt

​	获取libvirt源码，进入源码目录切换到你想要移植的分支，再依据该分支创建新的sw分支：

``` shell
$ git clone https://github.com/libvirt/libvirt.git
$ cd libvirt
$ git checkout -t origin/vxx.xx
$ git checkout -b vxx.xx-sw
```

​	编译源码（由于申威平台编译器的原因，需要添加--werror-disable）：

``` shell
$ ./autogen.sh --system --werror-disable 
```

​	如果只想编译申威平台支持的hypervisor，而不想编译其他driver，可以添加如下参数，减少编译时间：

``` shell
$ ./autogen.sh --system --werror-disable --without-vmware --without-openvz --without-esx --without-vbox
```

> 如果在这一步遇到了xdr相关的报错，并且确定依赖已安装完成，则可能是libc动态连接库有问题，需要将/lib/libc-2.23.so文件替换（目前符合我们需求的libc-2.23.so文件的md5码后几位是*08bb82，后续或有更新，须与申威方确认）。

​	autoconfig完成后，即可开始make：

``` shell
$ make -j 16
$ make install
```

​	通常完成make install之后就代表完成安装，可以运行libvirtd程序了。但是有几个需要提醒的地方：

> 若运行libvirtd出现动态连接版本冲突问题（通常是由于同时安装了两个不同版本的libvirt，如安装libviirt3.2之后，在未能卸载干净的情况下又安装了libvirt4.5，导致动态连接库冲突）则可以尝试先删除某个不需要版本的动态库再运行ldconfig命令。

``` shell
$ systemctl daemon-reload
$ systemctl restart libvirtd
$ systemctl status libvirtd
```

​	查看libvirtd状态，若需要，则安装dnsmasq和防火墙iptables或ebtables（目前iptable创建网卡相关部分依然存在部分问题，在[libvirt与virbr0](G:\博客产出\libvirt\libvir与virbr0.md) 中我们会进行详细讨论）。



***到这里，我们成功地在申威平台上运行了一个未修改过的libvirt，接下来我们为libvirt添加申威平台支持的hypervisor。***



## 2、连接一个申威平台支持的Hypervisor

​	在[libvirt使用](G:\博客产出\libvirt\libvirt使用.md) 中我们已经讲解过，想要使用libvirt首先需要connect到一个hypervisor，但目前申威平台能支持的hyperviosr有限，截至到本文完成的时候，申威平台能对虚拟化提供支持的内容包括：

1. swvm：参考lguest，自主开发的半虚拟化；
2. qemu-system-sw64：qemu移植，目前版本为2.11.0，只能够在申威平台模拟申威架构的虚拟机；
3. qemu-img：qemu-img移植，可支持完整的qcow2格式，与磁盘镜像的创建、快照的创建管理相关；
4. qemu-io：
5. qemu-nbd：
6. qemu-ga：qemu guest agent，虚拟机与宿主机通信通道，详细内容可参考[GuestAgent](G:\博客产出\QEMU\GuestAgent.md);
7. 



## 3、为申威适配libvirt
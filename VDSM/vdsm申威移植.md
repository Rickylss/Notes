---
layout: post
title: "vdsm申威移植"
date: 2018-11-16 11:48:20 +0800
categories: [libvirt, SW]
---



# vdsm申威移植

> 本文讲解了如何将vdsm-4.0移植到申威1621平台上，其中涉及到的诸多软件包及源码包将会在附属文件夹中提供，部分内容涉及到对源码的修改，请确保软件的版本一致。

## 1、获取vdsm源码并编译

​	首先从github上下载vdsm源码，切换分支到ovirt-4.0（注意：不是ovirt-4.0.0），再创建分支ovirt-4.0-sw：

``` shell
$ git clone https://github.com/oVirt/vdsm.git
$ cd vdsm
$ git checkout -t origin/ovirt-4.0
$ git checkout -b ovirt-4.0-sw
```

​	编译vdsm：

``` shell
$ ./autogen.sh --system
$ make
```

​	 由于vdsm项目由python完成，因此编译较快，同时许多程序运行时依赖并没有在Makefile中做检查，运行环境也没有设置，**这些工作都放在spec文件中了**，因此，我们需要手动make check，完成check之后，再手动添加脚本代替spec完成剩下的工作。

## 2、make check

​	为了检查vdsm各个功能是否可用，我们在vdsm源码目录下执行make check：

``` shell
$ make check
```

​	在这一过程中将会遇到许多报错，大多数情况是缺少依赖，这时只需要将依赖的包安装上去就可以了；少数情况下需要修改代码，这一部分我们将详细讲解。

### 2.1、安装libvirt与libvirt-python： 

​	目前最新libvirt未打包（libvirt-3.2.0有申威deb包），因此我们需要手动编译安装，在确定需要安装的libvirt版本之后，再对libvirt-python进行编译，这样就能确保libvirt-python模块与当前使用的libvirt对应（如果更换了libvirt，需要再次编译libvirt-python）：

``` shell
$ cd libvirt
$ ./autogen.sh --system --disable-werror
$ make -j 16
$ make install 
$ cd ../libvirt-python
$ ./setup.py build
$ ./setup.py install
```

​	查看python模块是否安装成功，查看版本信息是否正确：

``` shell
$ python
Python 2.7.11+ (default, Apr 17 2016, 14:00:29)
[GCC 5.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import libvirt
>>> print libvirt.getVersion()
4005000
>>>
```

​	显示400500，版本号为4.5，正确。

### 2.2、添加sw架构信息：

​	我们需要为vdsm添加对sw的cpu架构支持，修改`/vdsm/lib/vdsm/cpuarch.py`和`/vdsm/lib/vdsm/machinetype.py`如下：

``` diff
$ git diff lib/vdsm/cpuarch.py
diff --git a/lib/vdsm/cpuarch.py b/lib/vdsm/cpuarch.py
index 387e7e3..ad92815 100644
--- a/lib/vdsm/cpuarch.py
+++ b/lib/vdsm/cpuarch.py
@@ -29,8 +29,9 @@ from .config import config
 X86_64 = 'x86_64'
 PPC64 = 'ppc64'
 PPC64LE = 'ppc64le'
+SW_64 = 'sw_64'

-SUPPORTED_ARCHITECTURES = (X86_64, PPC64, PPC64LE)
+SUPPORTED_ARCHITECTURES = (X86_64, PPC64, PPC64LE, SW_64)

 PAGE_SIZE_BYTES = os.sysconf('SC_PAGESIZE')

@@ -88,6 +89,9 @@ def is_ppc(arch):
 def is_x86(arch):
     return arch == X86_64

+def is_sw64(arch):
+    return arch == SW_64
+

 def _supported(target_arch):
     if target_arch not in SUPPORTED_ARCHITECTURES:
............
$ git diff lib/vdsm/machinetype.py
diff --git a/lib/vdsm/machinetype.py b/lib/vdsm/machinetype.py
index fa3aef2..fd1af35 100644
--- a/lib/vdsm/machinetype.py
+++ b/lib/vdsm/machinetype.py
@@ -160,6 +160,9 @@ def _caps_arch_element(capfile, arch):
     if cpuarch.is_ppc(arch):
         arch = 'ppc64'

+    if cpuarch.is_sw64(arch):
+        arch = 'sw_64'
+
     arch_element = None

     arch_elements = cpu_map.findall('arch')

```

​	同时由于sw架构的指令特殊性，修改 `/vdsm/lib/vddsm/infra/eventfd/__init__.py`：

``` diff
$ git diff lib/vdsm/infra/eventfd/__init__.py
diff --git a/lib/vdsm/infra/eventfd/__init__.py b/lib/vdsm/infra/eventfd/__init_      _.py
index 91c3391..504a8f7 100644
--- a/lib/vdsm/infra/eventfd/__init__.py
+++ b/lib/vdsm/infra/eventfd/__init__.py
@@ -38,9 +38,18 @@ import os

 libc = ctypes.CDLL("libc.so.6", use_errno=True)

+"""
 EFD_SEMAPHORE = 0o0000001
 EFD_CLOEXEC = 0o2000000
 EFD_NONBLOCK = 0o0004000
+"""
+
+#the  value of above  is from  x86
+#next is from sw_64
+EFD_SEMAPHORE = 1
+EFD_CLOEXEC = 2097152
+EFD_NONBLOCK = 4
+


 class EventFD(object):
```

### 2.3、修改vdsm-tool加载路径

​	由于raise系统的特性，python包的位置与redhat系统有所不同，修改`/vdsm/vdsm-tool/vdsm-tool`：

``` diff
$ git diff vdsm-tool/vdsm-tool
diff --git a/vdsm-tool/vdsm-tool b/vdsm-tool/vdsm-tool
index cd357e8..31815da 100755
--- a/vdsm-tool/vdsm-tool
+++ b/vdsm-tool/vdsm-tool
@@ -28,13 +28,22 @@ import syslog

 # upgrade we need to explicit import vdsm.tool from /usr/lib. Otherwise
 # all tool's verbs are loaded from the old vdsm code during upgrade.
+
+#lib_dir = os.path.join(os.path.dirname(os.__file__).
+#                       replace('lib64', 'lib'), 'site-packages')
+
+#raise base on ubuntu , so  python packages in dist-packages
 lib_dir = os.path.join(os.path.dirname(os.__file__).
-                       replace('lib64', 'lib'), 'site-packages')
+                       replace('lib64', 'lib'), 'dist-packages')
+
 vdsm = imp.load_module('vdsm', *imp.find_module('vdsm', [lib_dir]))
 vdsm.tool = imp.load_module('vdsm.tool',
                             *imp.find_module('tool', vdsm.__path__))
```

> 在进行make check检查的时候，会有一定概率（较低概率）出现 错误，暂时无法发现该错误产生的原因，遇到这个错误的时候，我们的建议是多试几次：）

## 3、make install并手动设置运行环境

​	在检查全部通过之后，我们就可以make install安装vdsm了。但是，即使这样安装完之后，你依旧不能正常使用vdsm，因为你的service文件并不会在make install时安装，没错！！vdsm又把这一步放到spec里做了：）。

​	让我们打开spec文件，一探究竟：

``` shell
$ cat vdsm.spec | grep "install -"
install -dDm 0755 %{buildroot}/var/log/vdsm
install -Dm 0644 vdsm/storage/vdsm-lvm.rules \
install -Dm 0644 vdsm/limits.conf \
install -Dm 0755 init/systemd/systemd-vdsmd %{buildroot}/usr/lib/systemd/systemd-vdsmd
install -Dm 0644 init/systemd/85-vdsmd.preset %{buildroot}/usr/lib/systemd/system-preset/85-vdsmd.preset
install -Dm 0644 init/systemd/vdsmd.service %{buildroot}%{_unitdir}/vdsmd.service
install -Dm 0644 init/systemd/vdsm-network.service %{buildroot}%{_unitdir}/vdsm-network.service
install -Dm 0644 init/systemd/vdsm-network-init.service %{buildroot}%{_unitdir}/vdsm-network-init.service
install -Dm 0644 init/systemd/supervdsmd.service %{buildroot}%{_unitdir}/supervdsmd.service
install -Dm 0644 init/systemd/mom-vdsm.service %{buildroot}%{_unitdir}/mom-vdsm.service
install -Dm 0644 vdsm/vdsm-modules-load.d.conf \
install -Dm 0644 vdsm/vdsm-bonding-modprobe.conf \
install -Dm 0644 init/systemd/vdsm-tmpfiles.d.conf \
install -Dm 0644 init/systemd/unlimited-core.conf \
install -Dm 0644 vdsm_hooks/fcoe/85-vdsm-hook-fcoe.preset %{buildroot}/usr/lib/systemd/system-preset/
install -dDm 1777 %{buildroot}%{_localstatedir}/log/core
install -Dm 0644 vdsm/vdsm-libvirt-access.rules \
install -Dm 0755 vdsm/virt/libvirt-hook.sh \
```

​	可以看到许多配置文件、service文件都是在spec这里安装的，我们把这一段复制出来，创建一个vdsmInstall.sh脚本来运行它：

``` shell
$ touch vdsmInstall.sh
$ echo 'make install' > vdsmInstall.sh
$ cat vdsm.spec | grep "install -" >> vdsmInstall.sh
$ chmod +x vdsmInstall.sh
$ ./vdsmInstall.sh
```

​	这样我们就可以通过vdsmInstall.sh脚本来安装这些本该由spec安装的配置文件了。尝试开启vdsmd服务：

``` shell
$ systemctl start vdsmd
$ systemctl status vdsmd
```

## 4、初始化vdsm，修改配置文件

​	在上一步中尝试开启vdsmd将会报错，这是正常的：）。在第一次运行vdsmd之前，我们先来初始化一下：

``` shell
$ vdsm-tool configure --force
```

​	这时会出现用户和用户组不存在的报错，以及qemu.conf和libvirtd.conf配置冲突的报错。要解决用户与用户组不存在的报错，同样需要spec文件的帮助：

``` shell
$ cat vdsm.spec | grep -C 4 "useradd"
# Force standard locale behavior (English)
export LC_ALL=C

/usr/bin/getent passwd %{vdsm_user} >/dev/null || \
    /usr/sbin/useradd -r -u 36 -g %{vdsm_group} -d /var/lib/vdsm \
        -s /sbin/nologin -c "Node Virtualization Manager" %{vdsm_user}
/usr/sbin/usermod -a -G %{qemu_group},%{snlk_group} %{vdsm_user}
/usr/sbin/usermod -a -G %{cdrom_group} %{qemu_user}
```

​	spec就是在这里添加了vdsm的用户和组（注意：qemu、kvm和sanlock的用户和组，本应该由kvm模块和对应的软件包自行安装，但由于申威平台暂不成熟，并没有生成对应的用户和组，因此还是需要使用useradd命令手动安装）。将cat出来的信息添加到vdsmInstall.sh的脚本中，替换变量。运行脚本，此时用户添加成功。

​	解决文件配置冲突，只需要按提示要求修改配置文件即可。 

## 5、配置申威内核选项

## 6、vdsClients测试



​	
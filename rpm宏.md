# rpm宏

> 由于最近在打包一个rpm的时候发现这个包过于庞大，编译时间非常长，因此想要添加-j参数提升编译速度。

## 查看rpm宏

​	查看rpm宏的命令非常简单：

``` shell
$ rpm --eval %{_sysconfdir}
/etc
```

​	这是查看某个特定的宏；接下来看所有的宏：

``` shell
$ cat /usr/lib/rpm/macros
```

​	这个文件中就包含了所有的rpm宏啦。

## 添加或修改一个宏

​	为了达到我们的目的，接下来我们添加修改并行编译的这个宏：

1、通过-define参数临时扩展

``` shell
$ rpmbuild -ba xxxxx.spec -define '_sysconfig /etc'
```

2、直接修改宏定义文件

``` 

```

https://blog.csdn.net/hawkerou/article/details/53379664
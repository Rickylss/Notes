# windows更换开机动画

* 开机动画图片要求

1. 必须为小于256KB的.jpg格式图片；
2. 必须符合4:3的水平垂直比例。

* 修改注册表法一

    1. 首先在系统盘目录/Windows/system32/oobe下创建“info”文件夹；

    2. 在info文件夹中创建一个“background”文件夹；将开机图片复制到background目录下，同时将图片重命名为“backgroundDefault”；

    3. 以管理员身份打开注册表，在路径：
         ```
         HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Authentication\LogonUI\Background
         ```
          双击键值“OEMBackground” ，修改为“十六进制”“1”；

    4. 接着修改开机问候语，在路径：
         ```
         HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Window NT\CurrentVersion\Winlogon
         ```
         双击“LegalNoticeCaption”即标题信息，“数值数据”修改成自己的开机标题；再双击“LegalNoticeText”，数据数值修改自己成需要设置的开机文本；退出注册表，开始菜单，注销；

* 修改注册表法二

    1. 打开运行（Windows微标键+R键），输入“regedit“，打开注册表编辑器，在路径：
        ```
        HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\LogonUI\Background
        ```
           OEM机器里已经有“OEMBackground”这个项，而非OEM机器就没有这个项，这时候我们要手动新建这个项，右键点击右面空白处，新建一 个”DWORD（32位）值”，名称为“OEMBackground”，然后双击这个项，把“数值数据”里的“0”改为“1”（0代表关闭OEM厂商开关 机画面，1代表启用OEM厂商开关机画面，我们目的就是仿冒OEM厂商来让Windows显示“厂商”自定义的开关机画面^_^）。关闭注册表编辑器。

    2. 打开路径：

        ```
        X\Windows\system32\oobe\info\backgrounds
        ```

        “X”代表你的系统安装盘的盘符。同样，非OEM机器是没有这个目录的，所以非OEM机器用户要新建这样一个目录，然后把制作好并正确命名的图片拷贝进去就OK了，OEM机器用户请直接用新图片覆盖原目录图片就可以了。


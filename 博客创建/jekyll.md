# Jekyll

> Jekyll是一个简单的博客形态的静态站点生产机器。它拥有一个模板目录，其中包含原始文本格式的文档，通过一个转换器和Liquid渲染器转化成一个完整的可发布的静态网站。

## 安装Ruby与Jekyll

> 在安装Jekyll之前先要安装Ruby，虽然官方不建议在windo

- windows下安装Ruby

  在Windows下安装ruby，只需要在官网下载[rubyinstaller](https://rubyinstaller.org/downloads/) 直接安装即可，注意由于之后的jekyll需要使用到make，所以在安装ruby的时候选择同时安装msys和MinGW，安装时选择3再回车。

- Linux下安装Ruby

  ``` shell
  $ yum install ruby
  $ apt-get install tuby
  ```

- Jekyll安装

  ``` shell
  $ gem install jekyll
  ```

  > tips: 使用Pygments或者Rouge来支持代码高亮。




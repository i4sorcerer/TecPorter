# Idea IDE最佳使用指南

## 版本激活

2022年1月11日亲自安装，激活方法真实有效

### 官网下载版本

首先从官网下载，IDEA 2021.3.1 版本的安装包

https://www.jetbrains.com/idea/download/

### 下载激活工具包

链接: https://pan.baidu.com/s/1eqcqNNSdvPxpG-EgQ0iWAg 提取码: 7cds 复制这段内容后打开百度网盘手机App，操作更方便哦

文件下载到本地，加压到任意目录：（文件中的内容不要做任何修改）

/Users/ja-netfilter

### 注册JetBrains账号，以试用方式打开

30天试用期，我是直接以Github账号的方式登录

### 配置VM Options

- 以试用方式登录后，首页下方有个⚙，点击，选择``` Edit Custom VM Options...```

- 之后弹出编辑VM options配置文件弹窗，输入一下配置内容

  ```shell
  -javaagent:/Users/ja-netfilter/ja-netfilter.jar
  ```

> 最终检查Idea的激活情况，发现是永久激活。
>
> 大功告成，BINGO！！！！



## 常用提高效率插件及设置

### 设置

- 调整编辑器字体及schema：搜索[Color Schema Font] 调整size->16，调整schema

### 插件

- SonarLint 代码质量管理
- 代码检查



### 运行

- 客户端如果要启动多个实例，可以在``` Edit Configuration```中选择```Modify Options```,添加```Allow Multiple Instances```，就可以实现多客户端启动。但是这里需要注意，如果本地需要启动端口，需要选择随机的方式，否则会端口占用无法启动多实例。
- 

## 参考文章

1. *[Idea工具激活教程](https://www.exception.site/essay/how-to-free-use-intellij-idea-2019-3)*
2. *[SonarLint代码质量管理工具](https://mp.weixin.qq.com/s/yO1bC2ROLRK9R5fpaq34_Q)*
3. 




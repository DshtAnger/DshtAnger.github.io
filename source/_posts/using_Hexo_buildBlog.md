---
title: 使用Hexo建立博客
date: 2016-10-09 15:12:17
tags: [Hexo,Apache,Linux,Git]
---

如果使用`apt-get install`的方式来安装nodejs、npm会有一些坑，此处使用官方编译好的压缩包来安装Hexo的前期依赖，并在一台`ubuntu 14.x x64`的Ali ECS上进行部署，通过其上的Apache/Nginx直接解析public文件夹下生成的静态页面，通过git向Github Page同步`静态文件分支`，向自己的Github同步`代码分支`达到版本控制的目的

<!-- more -->

# 0x00 安装nodejs

从[官网](https://nodejs.org/en/)下载`Download for Linux (x64)`  
解压缩后，重命名文件夹，放置在合适位置，比如/root/nodejs  
通常用`tar zvxf ...`解压`.tar.gz`结尾的压缩包，`.tar.xz`需要两条命令  
```
xz -d node-v4.6.0-linux-x64.tar.xz
tar xvf node-v4.6.0-linux-x64.tar
mv node-v4.6.0-linux-x64.tar /root/nodejs
```

在`nodejs/bin`文件夹下发现`node`与`npm`两个文件  

设置软链接到`/usr/local/bin`使以上两个命令可以全局使用：
```
sudo ln -s /home/DshtAnger/nodejs/bin/node /usr/local/bin/node
sudo ln -s /home/DshtAnger/nodejs/bin/npm /usr/local/bin/npm
```

# 0x01 安装配置Git

此过程非常easy，主要查看参考文章中的如下两篇  
[Hexo搭建个人博客2](http://www.jianshu.com/p/99665608d295)  或 [生成Github的SSH Key](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)  

##  安装git
```
sudo apt-get install git-core
```

## 生成SSH Key
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```
把邮件地址换成自己的，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码  

如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对  

`id_rsa`是私钥，不能泄露出去。`id_rsa.pub`是公钥，是要往`Github Settings SSH Keys`页面填写的  

这样做的好处在于，有了SSH Key以后通过git push的时候无需像使用https时每次都要输入用户名和密码  

## 配置Github Page
在不购买服务器的情况下，可以直接将网站挂在GitHub Pages，这里仅作为一个ECS的备份使用：
1. 你需要拥有一个[GitHub账号](https://github.com/)
2. 进入[GitHub Pages](https://pages.github.com/)，跟着一步步做，完成后就能在浏览器打开http://DshtAnger.github.io  
  
  
# 0x02 安装Hexo

## npm换源
使用`npm`进行安装，此后一些插件也需要该工具，所以先将它配置好  

`npm` 全称 `Node Package Manager`，是node.js的模块依赖管理工具  

首先，将`npm`的更新源换为淘宝的更新源，否则无法安装成功，你懂的   

* 临时使用(增加参数)：`npm install -g hexo-cli --registry https://registry.npm.taobao.org`
* 持久使用(推荐)：`npm config set registry https://registry.npm.taobao.org`
配置后可通过下面方式来验证是否成功  
`npm config get registry` 或 ` npm info express`

## 开始安装
```
$ npm install -g hexo-cli [--registry https://registry.npm.taobao.org]
```

安装完成后会在nodejs文件夹下出现hexo的链接，再次将该文件链接到/usr/local/bin：
```
sudo ln -s /home/NAME/nodejs/bin/hexo /usr/local/bin/hexo
```
  
# 0x03 Hexo配置

克隆下来Github Page对应的仓库，放在/var/www/：
```
cd /var/www
git clone git@github.com:DshtAnger/DshtAnger.github.io.git
```

很多基本用法可以参考：[Hexo官方文档](https://hexo.io/zh-cn/docs/index.html)  

## 初始化
```
cd DshtAnger.github.io
hexo init
```
此后这个目录就是进行`hexo clean`、`hexo generate`、`hexo server`、`hexo deploy`的根目录  

## 安装依赖
```
npm install
```

## 生成静态文件
```
hexo generate
```

## 本地启动
```
hexo server   
```
浏览器输入`http://IP:4000`就能看到页面了  
 
# 0x04 部署到Github

[参考官网描述](https://hexo.io/zh-cn/docs/deployment.html)

所有操作都在同一个仓库内，`master`分支作为`静态文件分支`;`code`分支作为`代码分支`管理非public文件夹的代码文件  

## 静态文件分支

两种方法:
1. 使用原生git命令,只将public文件夹同步到Github
2. 使用hexo的git插件
  
第一种方法就是常规的git操作,不多说的,主要示范第二种方法  

### 安装git插件
```
$ npm install hexo-deployer-git --save
```

###  修改`_config.yml`中参数
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:DshtAnger/DshtAnger.github.io.git
  branch: master

```

### 一键部署
```
$ hexo deploy
```
完成后会发现根目录下出现一个`.deploy_git`文件夹，它和`.git`是类似的  

这样就将整个public文件夹下生成的静态文件push到`DshtAnger.github.io.git`仓库中的master分支.访问`https://dshtanger.github.io/`就能看到网站内容了  

## 代码分支

其实就是常规的git操作，新建一个分支命名为code，删除掉public文件夹后，同步其余所有文件  

### 前期清理
```
cd DshtAnger.github.io
hexo clean #删除public文件夹和清除.db文件
rm -rf theme/THEME_NAME/.git #克隆人家的主题后遗留的.git文件将干扰后续的代码提交
```

### 提交、创建并切换分支
```
git init
git add *
git commit -m "code files"
git checkout -b code
```
查看当前分支:`git branch`  

### 提交同步
```
git remote add origin git@github.com:DshtAnger/DshtAnger.github.io
git push -u origin code 
```
这样就在仓库里的code分支保存好代码文件了，以后通过此分支管理代码文件  

要用`hexo deploy`部署时，**无需切换回master分支**，因为根目录下的配置文件中已经定义了这个操作时在master下进行

## 后续管理都在code分支进行

无论是生成网页，还是修改文章、代码文件，都在`code`分支进行，但push前记得`hexo clean`一下
```shell
git checkout code
hexo clean
git add *
git commit -m "say something"
git push origin code
```

# 0x05 使用Apache解析静态文件

`hexo generate`后`public`文件夹下生成了网站的完整静态文件，只要让Apache或者Nginx以public文件夹作文根目录进行解析，就能直接通过ECS的公网ip访问到我们的网站，内容和同步到Github Page的一致，但访问速度和承载力自然胜过前者不少  

根据前面，在`/var/www`下有一个`DshtAnger.github.io`文件夹，放置了所有的Hexo代码文件和生成的静态文件夹`public`，现在修改Apache的默认根目录到public即可  

## 修改默认根目录
参考：[Ubuntu 14.04下Apache修改网站根目录及默认网页](http://www.linuxdiyf.com/linux/13068.html) 
```
在 /etc/apache2/sites-available 中修改 000-default.conf
中的DocumentRoot /var/www/ 修改为想要的目录
比如：DocumentRoot /var/www/html/DshtAnger.github.io/public
```

重启Apache服务后，接下来就能直接通过ECS的IP或者域名进行访问了，在未修改其他配置时默认挂载在80端口  

## 禁止显示目录结构
参考：[禁止apache显示目录索引的常见方法](http://www.jb51.net/article/46767.htm)  

`vim /etc/apache2/apache2.conf`，找到下面部分
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```
删除`Indexes`，然后重启服务

## 启用多端口
比如默认的80端口被其他的应用使用，可以将网站挂到其他端口如8080
参考：[多个端口配置虚拟主机](http://www.mengxinjie.cn/node/65)

### 修改ports.conf
`vim /etc/apache2/ports.conf`
```
NameVirtualHost *:80
Listen 80

NameVirtualHost *:8080
Listen 8080

```

### 修改网站启动配置文件
`/etc/apache2/sites-available`  
`cp 000-default.conf blog.conf`   
`vim blog.conf`
```
<VirtualHost *:8080>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/DshtAnger.github.io/public/

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### 启动配置
```
a2ensite blog.conf
service apache2 reload
```
然后就能在http://IP:8080访问到以public文件夹作为根目录的网站，和80端口互不干扰

# 0x06 更换主题

知乎上有个帖子推荐了很多不错的主题与链接，[有哪些好看的 Hexo 主题？](https://www.zhihu.com/question/24422335)  

1. 通过`git clone`对应的项目到`DshtAnger.github.io/themes`
2. 修改网站配置文件`_config.yml`
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: THEMES_NAME
```
3. 修改主题配置文件`THEMES_NAME/_config.yml`

## 本博客主题`yelee`
[Yelee](https://github.com/MOxFIVE/hexo-theme-yelee)基于主题[Hexo-Theme-Yilia](https://github.com/litten/hexo-theme-yilia)修改而来，在此感谢作者MOxFIVE的出色工作，该主题的完整[设置文档](http://moxfive.coding.me/yelee/)在此

### 滚动条问题
基本设置好后会发现代码区域会有很难看的边框滚动条显示出来，解决办法：  
`vim yelee/source/css/_partial/highlight.styl`
```

......
(24行处插入注释那句)
.article-entry
  pre, code
    font-family: font-mono, monospace, font-chs
    font-size: 1em
  pre
    @extend $code-block
    overflow: hidden //去除滑动条
    code
   
......
```
此外修改34行`max-height`参数可以改变代码区域的可显示高度  

### 404页面问题
`hexo new page 404`
`source/404/index.md`中添加页面内容：
```
---
title: 404 Not Found
toc: false
comments: false
permalink: /404
---
页面正文
```
完成后`hexo generate`后就会有`public/404.html`

为了使服务器解析到不存在页面，或者禁止访问的页面时，都跳到404页面，进行如下设置：
`vim /etc/apache2/apache2.conf`
```
//加入如下两行
ErrorDocument 404 /404.html
ErrorDocument 403 /404.html
```

### 修改/archives页面样式
`vim yelee/source/css/_partial/archive.styl`
我是直接替换了原版主题的这个文件，因为个人更喜欢原版的间距风格

### 修改边栏顶部随机颜色
`vim yelee/source/js/main.js`  
144行处找到如下内容：
```
var colorList = ["#6da336", "#ff945c", "#66CC66", "#99CC99", "#CC6666", "#76becc", "#c99979", "#918597", "#4d4d4d"]
```
删除其中的几个粉嫩颜色,`#CC6666`、`#ff945c`、`#c99979`等  

### 资源文件夹设置
为了方便文章内的图片等资源的统一管理，推荐使用hexo的资源文件夹特性，既省了使用外链图床的点击麻烦、也解决了路经引用问题  

参考官方文档说明吧，足够了  

[资源文件夹](https://hexo.io/zh-cn/docs/asset-folders.html)
  
# 0x07 参考文章
* [多种方式安装nodejs](https://my.oschina.net/blogshi/blog/260953)
* [npm换源](http://www.jianshu.com/p/0deb70e6f395)  
* [生成Github的SSH Key](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)  
* [Hexo官方文档](https://hexo.io/zh-cn/docs/index.html)  
* [简书-Hexo专题](http://www.jianshu.com/collection/7fafdc0abb5b)
* [Hexo搭建个人博客1](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)  
* [Hexo搭建个人博客2](http://www.jianshu.com/p/99665608d295)  
* [Hexo搭建个人博客3](http://www.jianshu.com/p/a2023a601ceb)  


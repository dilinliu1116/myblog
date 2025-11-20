---
title: "Hugo Install"
description: "hugo-install"
keywords: "hugo,install"

date: 2025-11-20T15:11:32+08:00
lastmod: 2025-11-20T15:11:32+08:00

math: false
mermaid: false

tags:
  - Hugo
  - Blog
  - Linux
  - git
categories:
  - 技术
  - Linux
  - git

---


# **Hugo Install and Cloudflare**
<!--more-->

---
# 一：安装Hugo
 到 https://github.com/gohugoio/hugo/releases 查看与CPU对应的版本下载
```shell
 root@iZuf6e4mkztyvtneiwbdzrZ:~# wget https://github.com/gohugoio/hugo/releases/download/hugo_extended_0.149.0_linux-amd64.deb
 ```

 进行安装
 ```shell
 root@iZuf6e4mkztyvtneiwbdzrZ:~# dpkg -i hugo_extended_0.149.0_linux-amd64.deb
 (正在读取数据库 ... 系统当前共安装有 183289 个文件和目录。)
正准备解包 hugo_0.80.0_Linux-64bit.deb  ...
正在将 hugo (0.80.0) 解包到 (0.80.0) 上 ...
正在设置 hugo (0.80.0) ...
 ```
验证（查看版本）
```shell
root@iZuf6e4mkztyvtneiwbdzrZ:/usr/local/src# hugo version
hugo v0.149.0-66240338f1b908ca3b163384c8229943e74eb290+extended linux/amd64 BuildDate=2025-08-27T15:37:16Z VendorInfo=gohugoio
```
---

# 二：用Hugo生成自己的博客
---
```shell
root@iZuf6e4mkztyvtneiwbdzrZ:/usr/local/src# hugo new site myblog
```
myblog为自己的博客名，可以随意

---
# 三：下载设置自己喜欢的主题,并在本地目录运行启动命令
---
我用的是reimu， 主题地址是：https://github.com/D-Sketon/hugo-theme-reimu
下载安装：我的myblog路径是/usr/local/src/myblog，进到路径内操作
```shell
git init
git submodule add https://github.com/D-Sketon/hugo-theme-reimu.git themes/reimu
echo 'theme = "reimu"' >> hugo.toml
hugo server -t reimu   --buildDrafts  --bind 0.0.0.0  -p 8081 # 允许所有地址来访问，默认是127.0.0.1，端口改为8081
```
---

# 四：写篇文章测试
---
1.在myblog目录下执行
```shell
root@iZuf6e4mkztyvtneiwbdzrZ:/usr/local/src/myblog# hugo new post/blog.md
```
生成的md文件在myblog/context/post目录下：
```shell
root@iZuf6e4mkztyvtneiwbdzrZ:/usr/local/src/myblog# ls content/post/
blog.md  hugo-install.md
```
2.写作,使用Markdown语法
```shell
root@iZuf6e4mkztyvtneiwbdzrZ:/usr/local/src/myblog# vim  content/post/blog.md 
+++
date = '2025-11-10T02:22:47Z'
draft = false
title = 'Domain-admin for k8s'
tags = ["kubernetes", "domain-admin"]
categories = ["技术", "k8s"]
+++
```
---
# 五：将将个人博客部署到远程服务器
---
1.登录github官网，创建一个新的仓库
2.填写仓库地址
3.将themes/reimu/下的文件夹内容cp到myblog目录下，一一对应，后续修改只需要修改外层的文件就行（主题地址有教程）
4.上传到Github,在myblog目录下
```shell
git init
git add *
git commit -m "第一次提交"
git remote add origin https://gitlab.com/dilinliu1116/myblog.git  #与远程仓库关联
git push -u origin master
```
---
# 六：与cloudflare结合，实现CI/CD自动部署静态页面
1.登录cloudflare，选择Workers and Pages --> 创建应用程序 --> 选择从GitHub进行导入，选择仓库进行连接
2.设置构建
```bash

构建命令: hugo --gc --minify
构建输出: public
根目录: 

生产分支: update-theme
自动部署: 启用
#设置变量
HUGO_VERSION = 0.149.0

名称：myblog
```
---

# 当有新的文档创作好后，执行git push 之后，cloudflare检测到有变更，便会自动拉取更新

---
# 主题地址：https://github.com/D-Sketon/hugo-theme-reimu
# 参考文档：https://blog.csdn.net/qq_42185895/article/details/113780415
# Cloudflare文档：https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site
# Hugo官网：https://hugo.opendocs.io/hosting-and-deployment/hosting-on-cloudflare-pages/
---
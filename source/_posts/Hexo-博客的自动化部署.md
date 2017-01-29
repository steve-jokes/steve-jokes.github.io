---
title: Hexo 博客的自动化部署
tags:
  - hexo
  - 博客
  - nginx
categories:
  - 技术
  - 博客
  - 网站历史
date: 2017-01-30 03:29:05
---


既然拥有了自己的 VPS, 单纯的部署到 GitPages 上这种 easy 的方式当然不能满足我. 将其最终部署到 VPS 上, 通过 [Memphis.Wang](http://memphis.wang/) 这个域名来访问才是王道. 今天就来完善一下我的 Hexo 部署作业. 一番折腾下来并不容易, 简单记录分享一下. (均在 CentOS 7 下完成)

<!--more-->

## 服务器
既然要通过访问我的 VPS 来打开博客, 自然需要一个靠谱的服务器啦. 出于习惯和周围同事的技术栈, 我选用 [Nginx](https://www.nginx.com/) 作为自己的一号方案, 安装部署起来都比较方便. 毕竟后端不是我的专业, 所以我直接通过 yum 包管理来安装一个落后的稳定版本. 通过 `nginx` 命令启动之后, 网页访问自己域名会有 Nginx 的默认页面出现, 安装成功.

## VPS 上的 Git 仓库准备
完成 Git 形式的博客部署, 需要在 VPS 上准备两个仓库:
- 一个 Git Bare 仓库, 没有工作区, 用于接受 Hexo 的部署推送, 触发 Bare 仓库的 hook 脚本, 更新服务器内容.
- 一个标准 Git 仓库, 在 hook 脚本的控制下, 从 Bare 仓库中导出, 带工作区, 存放我们最终用于部署的博客内容.

简单地说, 需要在 VPS 上选择一个目录:
1. `git init --bare MyBlog` (*命名随意*)
2. `git init MyBlogPublish` (*同上*)

## 配置 Nginx 服务器
`vim /etc/nginx/nginx.conf`, 指定 root 参数到 /path/2/MyBlogPublish, 我们的服务器将以这个仓库作为根目录.

## Hexo 配置部署选项
完成服务器 Git 仓库准备之后, 就可以在 **本地机器** 的 Hexo 项目中配置对于这台服务器的 Git 形式部署了, 编辑 Hexo 项目的 \_config.yml 文件, 增加 deploy 选项:

```
- type: git
  repo: ssh://username@your_vps_ip_address/path/2/MyBlog
  branch: master
```

如此一来, 当我通过 `hexo deploy` 执行部署命令时, 我的静态博客文件都会被推送到我指定的 VPS 机器的 Bare 仓库 MyBlog 上, 注意这里要选用 Bare 仓库, 否则推送会失败. 

## 通过 Bare 仓库的 hook 脚本完成博客的部署
博客推送到 Bare 仓库后, 会触发一个 post-receive 事件, 我们可以在这个脚本中注册我们的部署指令:

`vim MyBlog/hooks/post-receive`

增加以下内容:

```
#!/bin/sh
git --work-tree=/path/2/MyBlogPublish --git-dir=/path/2/MyBlog checkout -f
sudo nginx -s reload
```

执行 `chmod +x post-receive` 为这个 hook 增加执行权限.

---

这么一来, 当我在本机执行 `hexo deploy` 后会发生这么几件事: 
1. hexo 将静态博客以 git push 形式退送给我的 MyBlog Bare 仓库
2. 触发 Bare 仓库的 post-receive 事件, 执行 hook 脚本
3. 脚本从 MyBlog 中导出了一个 MyBlogPublish 仓库, 其工作区中包含了静态博客的内容
4. Nginx 发生了重新载入, 将 MyBlogPublish 仓库下的内容更新到了服务器上.

一切看起来这么完美, 但这件美好的事情可能并没有在你身上发生, 因为有如下几个坑需要填平:

## 坑s

#### 修改 Nginx root 目录以后, Nginx 出现了 403 错误
这是因为 MyBlog 目录的所属用户导致, 由于我不是后端, 对安全性需求几乎为 0, 所以粗暴的方法是修改 Nginx 服务器配置 `/etc/nginx/nginx.conf`, 将 user 设为 root.

#### MyBlogPublish 生成成功, Nginx 更新失败
由于 Nginx 的运行要访问几个系统目录, 所以需要通过 sudo 来运行 Nginx, 但是由于整个部署过程是一个自动化流程, hook 使用脚本来完成工作, 没有机会让我们在本地机器上为 VPS 上的部署输入密码, 所以需要去除对 Nginx 命令 sudo 时的密码要求:

登录服务器修改 `/etc/sudoers` 文件, 增加一行 `username ALL=NOPASSWD: /usr/sbin/nginx`

此外这套流程多处使用 ssh 登录 VPS, 为了避免频繁输入密码的麻烦, 建议使用 rsa key 方式登录, 具体办法 Google 一下, 不再详述 

---

至此, 我完成了对 hexo 部署流程的封装, 可以一体化控制, 只需要执行 hexo 的部署命令, 就能将博客更新到我的 GitPages 和 VPS 上. 之后我只需要专注与我的博客内容, 不需要关心这之后的一套部署细节. 甚至可能很长一段时间都不需要登录我的 VPS 了.

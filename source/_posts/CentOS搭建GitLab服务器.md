---
title: CentOS7中搭建Gitlab
tags: [Gitlab, 安装手册]
categories: [Gitlab]
created: 2020-05-25
abbrlink: '202005250941'
cover: https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/cover.png
top_img: https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/top_img.png
description: 每次搭建Gitlab都要百度寻找搭建步骤，修改端口容易忘记端口占用的问题导致修改失败，所以记录一次稍微详细的搭建步骤方便以后使用。
---
# CentOS搭建Gitlab的详细教程

记录一次CentOS搭建gitlab服务器的经历。

## 前期准备

1. 服务器：CentOS7
2. 安装文件：[gitlab-ce-12.6.3-ce.0.el7.x86_64.rpm](https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-12.6.3-ce.0.el7.x86_64.rpm)

## 安装gitlab

介绍一下两种安装方式 yum安装、rmp安装。

### yum安装

这里直接参考[官网](https://about.gitlab.com/install/#centos-7)安装教程

[![gitlab安装教程](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509411.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941-0/2020052509411.png)



打开linux系统终端，首先安装gitlab必须的ssh，以及在系统防火墙中打开HTTP、HTTPS和SSH访问。

```
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

然后是安装发送邮件功能的postfix

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

添加gitlab的包仓库（ee改成ce）

```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

安装gitlab（EXTERNAL_URL指的是你的gitlab访问地址，这里改为http://+你的linux系统ip）

```
sudo EXTERNAL_URL="http://当前系统的ip或域名" yum install -y gitlab-ce
```

### rpm安装

使用官网的安装方式下载很慢，这里可以直接下载rmp安装包手动安装。

首先去[官网安装包仓库](https://packages.gitlab.com/gitlab/gitlab-ce/)下载我们所需的安装包版本

[![官网安装包仓库](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509412.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941-0/2020052509412.png)

下载完成之后将文件拷贝至你的linux服务器，同样需要配置ssh、防火墙、postfix，

```
//安装gitlab必须的ssh，以及在系统防火墙中打开HTTP、HTTPS和SSH访问。
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
//安装发送邮件功能的postfix
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

然后cd进入你的安装包路径进行安装

```
//安装 example.rpm 包并在安装过程中显示正在安装的文件信息及安装进度
rpm -ivh example.rpm
```

出现下图即为安装成功

[![安装成功图](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509413.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509413.png)

这种方式需要我们手动进入配置文件中修改访问地址

```
sudo vim /etc/gitlab/gitlab.rb

//修改文件中external_url 'http://你linux的ip或域名'
```

并且我们还需要修改默认的gitlab clone地址，要不每次都得自己修改

[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509414.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509414.png)

修改文件配置

```
sudo vim /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml
```

将图片上标红处的Host替换成你的域名或ip

[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509415.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509415.png)

## 访问

重置并启动GitLab，执行以下命令

```
gitlab-ctl reconfigure

gitlab-ctl restart
```

提示 "ok: run:"表示启动成功

[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509416.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509416.png)

然后浏览器上输入你的访问地址（第一次访问会让你输入新密码，用户名默认为root）

[![img](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509417.png)](https://cdn.jsdelivr.net/gh/zhangbiao-code/blog_cdn@master/image/essay/202005250941/2020052509417.png)

## 修改访问端口

由于unicorn默认使用的是 `8080` 端口，打开 `/etc/gitlab/gitlab.rb` ，打开 `# unicorn['port'] = 8080` 的注释，将 `8080` 修改为 `9999` ，保存后运行 `sudo gitlab-ctl reconfigure` 即可(该端口不可与上方修改的端口一致)。

## 安装过程中遇到的问题

1. 在浏览器中访问GitLab出现502错误：
   原因：内存不足。
   解决办法：检查系统的虚拟内存是否随机启动了，如果系统无虚拟内存，则增加虚拟内存，再重新启动系统。

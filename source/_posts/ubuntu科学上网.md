---
title: ubuntu科学上网
date: 2019-05-16 13:52:44
categories: linux
tags: network
---

## 前言
安装任意一个linux发行版本后，第一步应该就是把梯子搭起来。毕竟在墙内，git,pip3,snap等等这些命令都需要代理才能流畅运行。

<!--more-->
## 原料准备
+ **一个ss账号，这个账号含有服务器ip,端口，密码以及加密方式这个四个key**
+ **一台能够上网的ubuntu电脑，并且安装了pip3**

## 安装shadowsocks3.0并修改配置
shadowsocks使用socks5 proxy与你租借的服务器建立通道，从而使你的计算机能够和服务器进行通信。
shadowsocks的linux版本很早就停止维护了，去它的github的repository什么也找不到。幸运的是现在还能用pip3命令下载到它3.0版本。
选择3.0版本是因为支持的加密方式更多。

在终端输入一下命令即可安装shadowsocks
`sudo pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip`
安装后执行
`ssserver --version`
显示`Shadowsocks 3.0.0`则安装成功

安装成功后新建一个shadowsock的配置文件，这个配置文件的名字和位置都无所谓，阿猫阿狗都可以。本文给这个配置文件选的是/etc/shadowsocks.conf,这个是最普遍的选择。
`sudo vim /etc/shadowsocks.conf`
之后在这个文件内以json的形式写入你所租借的vps的ip,端口，密码,以及加密方式，还有shadowsocks所连接的你本地的ip以及端口，
一般选择是127.0.0.1:1080.  shadowsocks配置文件格式如下所示。
```json
{
       "server":"XXXXXXXX",                                           
       "server_port":XXXX,
       "password":"XXXXXXX",
       "method":"XXXXXXXX"
       "local_address":"127.0.0.1",
       "local_port":1080,
   }
```
这样关于shadowsocks的准备就完成了。

## 安装privoxy并修改配置
由于shadowsocks从服务器接收到的流量是使用socks协议的，而本地的浏览器以及一些命令是使用http协议的，因此需要使用privoxy将socks流量
转成http流量。

ubuntu安装privoxy很简单，使用apt即可
`sudo apt install privoxy -y`

安装成功后修改位于`/etc/privoxy/config`的配置文件。修改两个参数即可：forward-socks5t和listen-address。
将forward-socks5t的地址修改为shadowsocks监听的本地地址，即前文设置的127.0.0.1:1080，这样privoxy即可接收shadowsocks的socks流量。
将listen-address的地址修改为另一个本地地址的端口，通常使用8118，其他的端口也可。之后所有的命令或者程序都是走这个端口接入外网的。
具体配置如下图：
![](/img/forward-socks5t.png)
![](/img/listen-address.png)

写好配置后保存，关于privoxy的配置完成。

## 修改系统网络配置

在完成privoxy和shadowsocks的安装与配置后，需要将系统的网络连接至privoxy的监听端口。一般有两种设置方法，全局设置和部分设置

### 全局设置
全局设置就是让系统所有的流量都走privoxy的本地监听端口:`127.0.0.1:8118`。这样系统上的绝大多数程序都能使用代理了。但是也有例外，比如
ping这个命令，它不是用http协议，即使走这个端口也ping不通google.com。但是不用担心，其实你的代理访问google是没问题的。
全局设置的方法很简单，到系统设置中找到网络，将network proxy设置为手动，并将http proxy和https proxy设置为prixoxy的监听端口即可。
如下图所示
![](/img/all_proxy.png)

### 部分设置
因为租借的ss账号有流量限制，为了节省流量，我不想所有的程序和网页访问都走这个端口。这样就可以只让某个浏览器使用代理，比如firefox，
当你要使用gitbuh或者google时，就用设置了代理的firefox。当你访问墙内网站时，就可以用其他没有设置代理的浏览器。
设置方法很简单，到firefox的设置里找到网络选项，将proxy设置为manual proxy configuration即可。将SSL Proxy设置为privoxy的本地监听
端口就完成了。至于这里为什么是设置ssl而不是http，我也不太清楚。
配置如下图
![](/img/firefox_proxy.png)

## 启动代理
代理启动有手动和开机启动两种方法。推荐开机启动。我一开始使用的手动，用了没几天就烦了。

### 手动启动
每次开机都输入以下命令，
`sudo service privoxy restart`
`sslocal -c /etc/shadowsock.conf`

### 开机启动
在终端输入下面的命令
`sudo systemctl enable privoxy`
在~/.profile内写入以下代码
`nohup sslocal -c /etc/shadowsocks.conf &>/deb/null &`

代理启动后，就可以快乐访问谷歌了。

## 其他代理设置
在部分代理设置中，如果仅仅给firefox设置代理，则仅有firefox能使用代理。而如果需要终端使用代理的话，则需要单独为其设置。

### 终端设置代理
打开~/.profile，写入以下数据
```
#Proxy
export http_proxy='127.0.0.1:8118'
export https_proxy='127.0.0.1:8118'
```
以上是为你的终端添加关于代理的系统变量。但是经过这样设置后，并不是万事大吉，不是所有命令都能使用代理。比如ping，它不用http协议。
再比如snap，它不使用你设置的系统变量。没办法,都得一一设置，除非你有一条直接连到美国的网线。

### snap使用代理
```
# systemctl edit snapd
加上：
[Service]
Environment="http_proxy=127.0.0.1:8118"
Environment="https_proxy=127.0.0.1:8118"
保存退出
# systemctl daemon-reload
# systemctl restart snapd
```

### pip3使用代理
打开~/.profile，写入以下数据
`alias pip3="pip3 --proxy 127.0.0.1:8118"`

### git使用代理
`git config --global http.proxy http://127.0.0.1:8118`

无需设置https，git clone不走http

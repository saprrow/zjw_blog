---
layout: pose
title: linux聪明命令
date: 2021-01-18 17:12:43
tags: "trick"
categories: "linux"
---

### 修改目录权限为当前用户:
```bash
chown -R 'whoami' $path 
```
-R 表示递归处理目录

### 添加到bash里开机启动这行命令
```bash
nohup v2ray --config ***.config >/dev/null 2>&1 &
```
`v2ray --config ***.config`这条命令表示运行v2ray
`nohup`表示退出终端后这条命令不会被终止,仍然在后台运行.
`>/dev/null`表示将输出导至linux的"文件黑洞",也就是删除输出的log.
`2>&1`表示将错误输出2导至标准输出&1.
最后的&表示将该行命令在当前终端后台运行.


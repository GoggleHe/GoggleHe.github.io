---
layout: fragment
title: Fragment Template
tags: [Linux, SSH]
description: Linux中SSH协议相关命令
keywords: Linux, SSH
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false

---

#### 开启ssh协议，关闭防火墙

sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

#### ssh协议 开启关闭重启状态

sevice ssh start/stop/restart/status
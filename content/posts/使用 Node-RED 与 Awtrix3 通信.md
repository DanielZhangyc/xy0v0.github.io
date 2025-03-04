---
date: 2025-03-05T03:07:57+08:00
tags:
  - 物联网
title: 使用 Node-RED 与 Awtrix3 通信
slug: "1741115277"
share: true
searchHidden: false
disableShare: true
math: false
mermaid: false
canonicalURL: ""
description: 本文介绍了如何使用Node-RED与Awtrix3进行通信。Awtrix3相比Awtrix2无需配置服务端，但需要通过MQTT通信进行App同步。首先，安装Node-RED并通过pm2实现开机自启动。接着，在Node-RED中配置MQTT服务器，使用node-red-contrib-aedes模块添加aedes broker，并配置端口号。然后，将Awtrix和模块连接至MQTT服务器，修改Node-RED中的mqtt out模块设置。重新启动并部署Node-RED后，观察aedes broker下方的连接提示，确保配置成功。最后，在flow上安装所需App即可完成配置。
series: 系列
lastmod: 
lang: cn
cover:
    image: 
author: xy0v0
dir: posts
---
Awtrix3 相比 Awtrix2 不必配置服务端，但是其 App 安装与同步方法也有所改变：需要通过 `MQTT` 通信来进行 App 的同步。

## 安装 Node-RED

```bash
npm install -g pm2
npm install -g --unsafe-perm node-red

pm2 start node-red
pm2 save
pm2 startup
```

通过 `pm2` 实现开机自启动

## 在 Node-RED 中配置 MQTT 服务器

设置 -> 控制板 -> 安装，搜索 `node-red-contrib-aedes` ，在模块中添加 `aedes broker` ，配置端口号/用户名/密码，由于仅在内网使用，就不考虑账号密码设置了
![MQTT服务器配置](https://cn-sy1.rains3.com/pic/pic/2025/03/76f9cac3b22c352cb7304befc01e9809.png)

## 将 Awtrix 和模块连接至 MQTT 服务器

前往 Awtrix Web 后台，进入 MQTT 配置页面，配置 IP 地址和端口即可

![MQTT连接配置](https://cn-sy1.rains3.com/pic/pic/2025/03/ecd37fc4ed2ebdbc0be7ad8f983c2d6f.png)

同时修改 Node-RED 中的 `mqtt out` 模块设置

![NodeRED模块设置](https://cn-sy1.rains3.com/pic/pic/2025/03/2c0a14483b80ffaccfd9de07bc017759.png)

重新启动并部署 Node-RED 后，观察 `aedes broker` 下方连接提示，正常情况下有两个连接即说明配置成功

![image.png](https://cn-sy1.rains3.com/pic/pic/2025/03/89ce7d1d268b14c49ce013dd24c9922f.png)

后续在 [flow](https://flows.blueforcer.de/) 上安装所需 App 即可

![example](https://cn-sy1.rains3.com/pic/pic/2025/03/44cfd4a15371a3fd6eb51d880b03e933.png)

![效果1](https://cn-sy1.rains3.com/pic/pic/2025/03/a819a878651a2c53481bf6ad31d8e5dc.JPG)
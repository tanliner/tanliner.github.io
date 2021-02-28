---
layout:     post
title:      Github 无法访问
subtitle:   
date:       2021-02-28
header-img: img/post-bg-desk.jpg
catalog:    true
tags:
    - Github

---
# VPN
vpn 已经启用，google访问正常，github头天还很正常，突然就无法访问了，ping偶尔超时

需要配置host，之前github的 ip 是`140.82.113.3`，gist的ip是`140.82.112.4`，最新的地址变成了114.4了

[ref-Segmentfault](https://segmentfault.com/a/1190000037498373) 

```
# GitHub Start 
140.82.114.4 github.com
140.82.114.4 gist.github.com
185.199.108.153 assets-cdn.github.com
151.101.64.133 raw.githubusercontent.com
151.101.108.133 gist.githubusercontent.com
151.101.108.133 cloud.githubusercontent.com
151.101.108.133 camo.githubusercontent.com
151.101.108.133 avatars0.githubusercontent.com
151.101.108.133 avatars1.githubusercontent.com
151.101.108.133 avatars2.githubusercontent.com
151.101.108.133 avatars3.githubusercontent.com
151.101.108.133 avatars4.githubusercontent.com
151.101.108.133 avatars5.githubusercontent.com
151.101.108.133 avatars6.githubusercontent.com
151.101.108.133 avatars7.githubusercontent.com
151.101.108.133 avatars8.githubusercontent.com 
# GitHub End
```

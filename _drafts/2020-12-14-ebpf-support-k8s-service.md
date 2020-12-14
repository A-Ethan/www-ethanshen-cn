---
layout:     post
title:      "[kubernetes]利用 eBPF 支撑大规模 K8s Service"
subtitle:   "转自公众号：云原生领域"
date:       2020-12-14 12:00:00
author:     "Ethan"
header-img: "images/default-bg2.jpg"
catalog: true
tags:
    - kubernetes
---


[(译)深入理解 Kubernetes 网络模型 - 自己实现 kube-proxy 的功能](https://mp.weixin.qq.com/s/zWH5gAWpeAGie9hMrGscEg)

[{译}深入理解 Cilium 的 eBPF 收发包路径（datapath）（KubeCon, 2019）](http://arthurchiao.art/blog/understanding-ebpf-datapath-in-cilium-zh/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[Linux 数据包跟踪工具 -- Perf/eBPF](https://mp.weixin.qq.com/s/PTxcT9aqL5lKKZB1cG_WxA)

[Cilium：基于 BPF/XDP 实现 K8s Service 负载均衡](https://mp.weixin.qq.com/s/m7ZVM5bc4N7FnzR_nhqD-g)
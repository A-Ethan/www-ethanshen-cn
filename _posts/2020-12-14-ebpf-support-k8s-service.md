---
layout:     post
title:      "[kubernetes]Cilium CNI通过eBPF实现替代kube-proxy"
subtitle:   "未来可期的CNI插件"
date:       2020-12-14 12:00:00
author:     "Ethan"
header-img: "images/default-bg2.jpg"
catalog: true
tags:
    - kubernetes
---

K8s当前重度依赖iptables来实现Service的抽象。对于每个Service及其backend pods，在K8s里会生成很多iptables规则。**例如5K个 Service时，iptables规则将达到25K条**，导致的后果：

* **较高、并且不可预测的转发延迟（packet latency）**，因为每个包都要遍历这些规则 ，直到匹配到某条规则；
* **更新规则的操作非常慢**：无法单独更新某条 iptables 规则，只能将全部规则读出来 ，更新整个集合，再将新的规则集合下发到宿主机。在动态环境中这一问题尤其明显，因为每 小时可能都有几千次的 backend pods 创建和销毁。
* **可靠性问题**：iptables 依赖 Netfilter 和系统的连接跟踪模块（conntrack），在 大流量场景下会出现一些竞争问题（race conditions）；UDP 场景尤其明显，会导 致丢包、应用的负载升高等问题。

Cilium通过内核的能力实现kube-proxy的功能，这点不同于iptables和netfilter体系结构。

在 netfilter 框架中有5个钩子

![](/images/kubernetes/2020-12-15-11-14-25.png)

Cilium是未来可期的CNI插件，目前商用且掌握的并不多。短期看还是使用IPVS，调研Cilium。

以下文章可以参考：

[利用 eBPF 支撑大规模 K8s Service](https://mp.weixin.qq.com/s/TdSOHk5EnSBer8uG5vqNyg)

[(译)深入理解 Kubernetes 网络模型 - 自己实现 kube-proxy 的功能](https://mp.weixin.qq.com/s/zWH5gAWpeAGie9hMrGscEg)

[{译}深入理解 Cilium 的 eBPF 收发包路径（datapath）（KubeCon, 2019）](http://arthurchiao.art/blog/understanding-ebpf-datapath-in-cilium-zh/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[Linux 数据包跟踪工具 -- Perf/eBPF](https://mp.weixin.qq.com/s/PTxcT9aqL5lKKZB1cG_WxA)

[Cilium：基于 BPF/XDP 实现 K8s Service 负载均衡](https://mp.weixin.qq.com/s/m7ZVM5bc4N7FnzR_nhqD-g)
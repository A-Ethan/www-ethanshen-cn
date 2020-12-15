---
layout:     post
title:      "[devops]Kubernetes如何加速UCloud内部代码部署的CI/CD流程"
subtitle:   "UCloud技术分享"
date:       2020-12-15 12:00:00
author:     "Ethan"
header-img: "images/default-bg2.jpg"
catalog: true
tags:
    - kubernetes
    - devops
---

## 我们对CI/CD的目标

**CI即Continuous Integration（持续集成）**,指代码集成到主干之前必须通过自动化测试，只要有一个测试用例失败，就不能集成。目的是让产品快速迭代的同时还能保持高质量。

CD有两种意思：

**Continuous Delivery，持续交付**，指的是任何的修改都经过验证，可以随时实施部署到生产环境。

**Continuous Deployment，持续部署**，是持续交付的更高阶段，指的是任何修改后的内容都经过验证，自动化的部署到生产环境。

两者的区别，在于是否自动部署到生产环境。对UCloud而言，肯定要以后者，也就是持续部署为目标。

![](/images/devops/2020-12-15-15-52-47.png)

## Gitlab分支管理

我们采用的Gitlab分支管理模型在接入KUN平台前后并未发生变化，故简述如下：

![](/images/devops/2020-12-15-15-53-43.png)

* master：主分支，代码经过验证。从 master 创建 tag 进行 Release。
* dev：研发主分支，用于合入特性分支和补丁分支，在此分支。
* 临时分支：
  * 特性分支：用于开发某个特性；
  * 补丁分支：用于修复线上 bug。

在实例中，我们需要开发其中名为optimize-allocate 的一个feature：

![](/images/devops/2020-12-15-16-47-35.png)

则开发流程为：

1. 首先，在 Gitlab 上创建了编号为80的Issue，跟进这个optimize-allocate的feature；
2. 从dev分支创建一个新分支，名为feature/80-optimize-allocate，在该分支上进行开发；
3. 在feature/80-optimize-allocate上开发完成，进行commit，此时会触发静态测试、单元测试、Review等Pipeline Job；
4. 测试通过后，创建一个从feature/80-optimize-allocate到dev的merge request，由负责人进行审核。审核通过并且merge成功后，触发静态测试、单元测试、镜像构建、镜像部署、集成测试等Pipeline Job；
5. 测试通过后，创建一个从dev到master的mergerequest，由负责人进行审核。审核通过并且merge成功后，负责人创建tag v1.1.1，然后触发静态测试、单元测试、镜像构建、镜像部署、集成测试等Pipeline Job；

注：版本号tag是有命令规范的，v{x}.{y}.{z}代表着v{主版本}.{次版本}.{小修订版本}

## Gitlab CI/CD Pipeline

持续集成的流水线，其中有Pipeline、Stage和Job三个层级需要设计。

#### Pipeline

Gitlab会检测一个项目的根目录下的`.Gitlab-CI.yml`文件，用户可在该文件中定义CI/CD Pipeline，一个Pipeline代表了CI/CD的整个过程。代码发生变化时（比如 push、tag、merge等），将触发一个Pipeline 的运行。

#### Stage

一个Pipeline包含多个Stage，比如“静态检查”、“单元测试”、“构建镜像”等等，这些Stage一个接一个顺序执行。

![](/images/devops/2020-12-15-16-56-41.png)

#### Job

每一个 Stage 可以包含多个 Job，同一个 Stage 的所有 Job 同时执行。每个Job需指定若干个重要属性如image, stage, tags, service等。

针对要开发的feature，其Pipeline设计如下，包括静态检查、单元测试和最后的两个手动 Code Review：

![](/images/devops/2020-12-15-16-59-14.png)

当其需要发布到公共测试环境（类似预发布环境），则设计另一个Pipeline，包括：执行完整的静态检查, 单元测试, 预发布镜像 build, 预发布部署, 预发布集成测试。

![](/images/devops/2020-12-15-16-59-48.png)

而合并 master 之后触发 prod 环境的另一个 Pipeline，包括静态检查、生产环境镜像的 build、生产环节的部署、最后集成测试：

![](/images/devops/2020-12-15-17-00-14.png)

## Gitlab Runner

![](/images/devops/2020-12-15-17-01-42.png)

我们将Gitlab Runner和Gitlab-CI配合使用，负责具体 Job 的运行。Gitlab Runner Kubernetes Executor 提供了在 Kubernetes 中运行 Job 的能力。运行原理如下：

Gitlab Runner 需要事先部署并注册到 Gitlab 上；

当代码发生更新时，Gitlab 根据用户的配置，通知 runner 来运行 Job；

Gitlab Runner 使用某个镜像来创建一个 Pod，然后在其中运行某些命令；

Gitlab Runner 将整个运行过程以及运行结果告诉 Gitlab。

## Kaniko集成和改造：在容器中构建Docker镜像

为了使用 CI/CD 将代码变成最终运行在 Kubernetes 中的服务，必不可少的一步就是容器镜像的构建。由于CI Job本身就是以容器的形式运行的，所以需要在容器中构建出 Docker 镜像。

**Kaniko**（https://github.com/GoogleContainerTools/kaniko）是Google开源的一个工具，可以实现在docker容器里面制作 docker镜像，并且不需要任何特权。

![](/images/devops/2020-12-15-17-03-23.png)

原生的Kaniko镜像需要整合busybox工具包，使其成为一个CI Job容器镜像来运行。

## KUN+Gitlab：基于Kubernetes的CI/CD流程

![](/images/devops/2020-12-15-17-11-09.png)

KUN中CI/CD的整个流程如上图所示。从图中我们可以看到，CI部分是一个单元测试，预发布部署，集成测试，Debug，提交代码的循环过程。在CI过程中预发布的部署是通过CD系统来实现的，CI其中一个步骤是生成部署文件，然后按照项目、资源集、版本等参数提交到CD后端系统。CD系统提供了页面入口，当部署文件推送到CD系统后，用户可以在页面查找对应的部署文件。如果需要部署，在页面点击“部署”按钮即可实现部署。

#### 部署系统

目前部署系统主要包含两部分功能：

1. 资源集管理：

   * 资源集是用户项目下一个或多个资源的集合，资源指Kubernetes的object的描述文件，一般为YAML文件；
   * 资源集分为多个版本，不同的版本对应资源的不同的描述文件；
   * 用户可以选择某个版本，然后指定集群执行部署。

![](/images/devops/2020-12-15-17-16-16.png)

![](/images/devops/2020-12-15-17-16-25.png)

2. 部署任务

用户每次部署都会生成一个 job，就是部署任务

执行部署后，用户可在任务详情页直接查看部署任务日志。

![](/images/devops/2020-12-15-17-16-50.png)

![](/images/devops/2020-12-15-17-17-00.png)

当完成上述所有工作，Gitlab能很好嵌入KUN平台，运行在Kubernetes上的各模块就可像齿轮一样有序运转，从而获得CI/CD上的良好效果。



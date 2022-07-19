# CI/CD概述

https://www.redhat.com/zh/topics/devops/what-is-ci-cd

1）一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。

2）核心概念：持续集成、持续交付、持续部署

3）由开发和运维以`敏捷`方式协同支持

> CI

指持续集成，属于开发人员的自动化流程。成功的 `CI` 意味着代码会定期构建、测试并合并到代码库中

> CD

1）持续交付

- 指开发人员对应用的修改会自动进行错误测试并上传到代码库，然后由运维部署到环境中
- 目的是尽可能减少部署新代码时所需的工作量

2）持续部署

- 另一种 `CD`，指自动将开发人员的修改从代码库发布到环境
- 为解决因手动部署导致的效率低下问题

> CI / CD 步骤

构建 => 测试 => 部署

# github actions

https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html

1）持续集成由：抓取代码、运行测试、登陆远程服务器、发布到第三方服务等。github 把这些操作叫 `actions`

2）很多不同项目的操作类似，完全可以共享，github 想出个点子，允许开发者自己写脚本放到代码库里供其他人用

3）可以引用他人的 `action`，整个持续集成过程变成了一个 `actions` 组合

## 术语

1）workflow：持续集成一次运行的过程

2）job：一个 workflow 由一个或多个 job 构成

3）step：每个 job 由多个 step 构成，即：一步一步完成 job

4）action：每个 step 依次执行一个或多个 action（命令）

# jaeger

用于追踪分布式服务之间事务的开源软件，监控复杂的微服务环境

# prometheus

作为监控后起之秀，广泛用于 `k8s` 集群的监控系统中

# 云效

阿里云的一套 ci / cd

https://developer.aliyun.com/article/776788

https://help.aliyun.com/document_detail/153762.html

# rust


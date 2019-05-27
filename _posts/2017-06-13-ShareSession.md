---
title: 分布式下共享session
date: 2017-06-13
tags: 服务器
---
# 解决分布式环境session共享

1. session replication : 适合小集群。
2. 集中存储 : 适合session数比较多，服务器数量比较大的情况
3. session维护在客户端 。 access_token/token 缺点安全性

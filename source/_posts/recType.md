---
title: 推荐系统实验策略
date: 2017-07-08 16:50:29
tags:
---
##实验策略
 根据不同业务分层，根据用户id进行hash取摸，然后分成100片，每个用户只在某一个分片里，来对不同用户进行不同的实验策略划分。

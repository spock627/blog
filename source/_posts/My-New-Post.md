---
title: 在线服务优化总结
date: 2017-07-05 11:02:29
tags:
---
优化目标：

1. 减少对redis依赖。

2.降级问题

3.出错诊断困难

4.压测缺乏。

5.插件化优化。

 

 1. 减少redis依赖

   （大部分召回和过滤依赖的redis，可以本机cache，考虑持久化的索引文件，用户相关的暂不适合）

      定期离线计算=>定期更新redis索引=>召回时实时读取redis索引

     改为：

      定期离线计算=>定期更新redis索引=>定期读取redis索引到内存(dump到本地文件)=>召回实时读取本机内存(本地文件backup）

   

 

 

    用户相关：

        用户观看历史 (保持redis)

    召回相关：

          量少，可以单机存：redis=>单机cache， 或者索引文件。 结构: map<string, vector<FeedIdWithScore>>

              热门视频，冷启动视频

              根据视频标签、owner_id召回

              同城热门

          不能单机索引，只能redis的：矩阵分解为每个用户的结果

         质量分：也是定期更新，更新间隔短一些。

   过滤：

      人工黑名单： 本机cache，(持久化)，定期同步

      视频属性过滤：每天推荐出去的视频9万个，各种属性（发布时间，ownerid，标签，计数器）可以定期存储，全量或增量从redis更新，并持久化

      按照owner 过滤的，维护一个owner黑名单， 也可以转成feed_id黑名单去过滤。

 

       

 

2. 降级：

    系统降级：

     统计短期的请求数，超时情况，判断系统降级等级， 根据等级不同，关闭不同的逻辑：插件，api，耗时逻辑（读redis的召回逻辑，读redis取用户历史的逻辑，耗时的api，耗时打分排序）。

    请求降级：维护一定大小（比如100个）的cache，正常有结果的时候随机替换cache，请求无结果时从cahce里挑一个返回。

    请求降级是偶发性的，系统降级是大规模的。

 

 

3. 调试：

request的context中搞一个debug info的结构体 或者string， 接口传递调试参数 debug_level， 处理过程中，根据debug_level判断下面信息是否要写入debug_info中：

  用户的信息（观看，标签。。）

         例如：if (debug_level >= 1) {  记录 user_history_size}

                   if (debug_level >= 2) {  记录 recent_10_user_history(feed_id + time) }

                   if (debug_level >= 3) {  记录 recent_10-100_user_history(feed_id + time) }

 

 

  召回与过滤：每一路召回的key，个数，每一个过滤器过滤的个数

          例如： if (debug_level>=2) { 记录 rec_type + size }；

                     if (debug_level >=4) {记录 各feed_id }

                     if (debug_level>=1) {记录 filter_name + filted_size }

                     if (debug_level >=3) {记录 filter_name + filted_feed_id_list }

     调用api：api名称 耗时 重试 等基本信息

             例如 if (debug_level>=2) { 记录 api name time_cost.}

                     if (debug_level >=1 && time_cost > 100) { 记录 api name + time_cost}

  打分： 每一个item的打分过程

  多样性干预： 比如: 把item 从第5调到第2的原因

 

  每一个阶段的耗时,每一个插件/api的耗时。

 

最后打印debug_info，或者api返回。

 

 

4.测试：

上线前的测试，要在测试环境，依赖接口也尽可能改成非线上接口，避免影响线上。

 

 5.插件化优化

   部分逻辑写在了不合适的地方。

 

 

 

 

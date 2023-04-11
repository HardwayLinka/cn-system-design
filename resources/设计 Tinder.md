[原文](http://highscalability.com/blog/2022/1/17/designing-tinder.html)

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51825355006_1f7f8b400e_n.jpg)
# 问题陈述

设计一个基于位置的社交搜索应用程序，类似于 Tinder，如果经常用作约会服务。它允许用户使用滑动动作来喜欢 (右滑) 或不喜欢 (左滑) 其他用户，如果双方都喜欢对方 (“匹配”)，则允许用户聊天。

## 收集需求

### 范围内

应用程序应该能够支持以下需求。

* 用户应该能够通过添加个人简介和上传照片来创建他们的 Tinder 个人资料。

* 用户应该能够查看附近地理区域的其他用户的推荐。

* 用户应该能够喜欢 (向右滑动) 或不喜欢 (向左滑动) 其他推荐用户。

* 用户应该在与其他用户匹配时获得通知。

* 用户应该能够移动到不同的位置，仍然可以得到附近用户的推荐。

### 超出范围

* 发送和接收其他用户的消息。

# 高级设计

## 架构

![](https://live.staticflickr.com/65535/51825289156_54f5a08b26_z.jpg)

在网关后面将会有一群微服务为用户请求提供服务。当创建用户配置文件时，配置文件创建器服务将被调用。该服务将用户信息存储在数据库中，并将用户添加到相应的 geo-sharded 索引中，以便该用户出现在附近用户的推荐中。当推荐服务收到为其他用户生成推荐的请求时，这个索引就会被查询。一旦用户开始在这些推荐中滑动，滑动服务就会接收这些滑动并将它们放置在数据流中 (例如。AWS Kinesis/ SQS)。有一群 worker 从这些流中读取数据以生成匹配。worker 通过查询 LikesCache 来确定它是否匹配，在这种情况下，匹配通知会使用 WebSockets 等技术发送给两个用户。

# 组件设计

## 创建用户资料

![](https://live.staticflickr.com/65535/51825294236_1804c53b9b_b.jpg)

上面的序列图展示了用户在 Tinder 上创建个人资料时执行的操作序列。在同步过程中，用户媒体 (例如照片) 被上传到文件服务器上，用户信息 (包括用户的位置) 被持久化到一个 key-value 存储中，比如 Amazon DynamoDB。此外，该用户被添加到一个队列中，用于将该用户添加到地理分区索引中。

异步进程从队列中读取用户信息，并将此信息传递给 GeoShardingIndexer。索引器使用类似谷歌的 S2 库这样的地理库来将用户的位置映射到地理分片上，并将用户添加到与该分片相关联的索引中。这有助于用户出现在附近其他用户的推荐中。例如，在下面的图片中，我们展示了一个来自北美的用户如何被映射到相应的索引中，以便该用户在附近用户的推荐中显示出来。

![image.png](https://raw.githubusercontent.com/HardwayLinka/image/master/20230411203811.png)
图：来自北美的用户将被映射到相应的分片（图片来源：[Tinder Engineering Blog](https://medium.com/tinder-engineering/geosharded-recommendations-part-1-sharding-approach-d5d54e0ec77a))

# UserProfileInfo - 样本数据模型

我们在下面展示了一个用于存储用户配置文件信息的 json blob。我们可以使用 key-value 存储，如 Amazon DynamoDB 或 Riak 来维护此数据。

```
{
    “userId”(PK) : “AWDGT567RTH”,
	“name” : “Julie”,
	“age” : 25,
	“gender” : “F”,
	“location”: {
		“latitude” : 
		“longitude” : 
        },
        “media”: {
	   “images”: [
		“https://mybucket.s3.amazonaws.com/myfolder/img1.jpg”,
		“https://mybucket.s3.amazonaws.com/myfolder/img2.jpg”,
		“https://mybucket.s3.amazonaws.com/myfolder/img3.jpg”
           ]
        },

    “recommendationPreferences”: {
	“ageRange”: {
	“min”: 21,
	“max”: 31
        },
       “radius”: 50
    }
}
```

# 获取用户推荐

![](https://live.staticflickr.com/65535/51825308581_bf0482a3f2_b.jpg)

在上一节中，我们看到了用户是如何被添加到 geo-sharded 索引中的。再来看看用户在其他用户的推荐中是如何显示的。当用户请求到达推荐引擎时，它会将请求转发给 GeoShardedIndexer。索引器根据用户位置和半径使用像谷歌的 S2 这样的地理库来确定要查询的地理分片。之后，索引器查询所有映射到谷歌 S2 返回的分片上的地理分片索引 (更多细节请参见下一节)，以获取这些索引中所有用户的列表，并将该列表返回给推荐引擎。引擎根据用户偏好对列表进行过滤，并将最终的推荐列表返回给用户。

## Geo-Sharded 索引

一个维护这个索引的天真的方法是有一个 Elasticsearch 集群，该集群有一个索引和默认数量的分片。然而，这种方法无法满足 Tinder 这样的应用所需的扩展预期。我们应该利用 Tinder 的推荐是基于位置的这一事实。例如，当我们为印度用户提供服务时，我们不需要包括美国用户。这一事实可以通过保持最佳的索引大小来获得更好的性能。我们可以通过基于地理位置对活跃用户记录进行分片来优化索引大小，从而使活跃用户数量在各个分片之间保持平衡。我们可以通过下面提到的跨分片的活跃用户计数的标准差来表示一个具有 N 个分片的 geo-sharding 配置的平衡。

```
Balance(Shard1, Shard2,…, ShardN) = standard-deviation(Active User Count of Shard1, Shard2,…, ShardN)
```

标准差最小的地理分片配置将达到最佳平衡。我们可以使用像 [谷歌的S2库](https://docs.google.com/presentation/d/1Hl4KapfAENAOf4gv-pSngKwvS_jwNVHRPZTTDzXXn6Q/view?pli=1#slide=id.i5) 这样的地理库，它是基于使用四叉树将球体分层分解为“单元”。我们在下面展示了为我们的用例生成的地理分片地图的可视化。我们可以从下图中推断，对于活跃用户数量较低的地区，地理分片在物理上更近、更大。例如，在下图中，碎片在海洋等水体上更大，因为它们只有来自某些岛屿的用户，然而，陆地上的碎片更小。在下图中，我们可以看到北美有三个分片，然而，由于活跃用户密度较低，整个英格兰和格陵兰以及大部分大西洋共享一个分片。

![](https://live.staticflickr.com/65535/51825312781_62ac1d2278_c.jpg)

S2 库提供了两个主要功能:i) 给定一个位置点 (lat, long)，返回包含它的 S2 单元格 ii) 给定一个圆 (lat, long, radius)，返回覆盖圆的 S2 单元格。每个 S2 单元格都可以由一个 geo-shard 来表示，这个 geo-shard 将被映射到我们系统中的一个索引。当创建一个配置文件时，用户就会被添加到对应的 S2 单元格的搜索索引中。为了获取用户的推荐，我们根据下图所示的圆半径查询附近 S2 单元格的索引。

![](https://live.staticflickr.com/65535/51826037680_645ba3d363_z.jpg)

图: 从附近的 shards 中为用户获取推荐 (图片来源:[Tinder工程博客](https://medium.com/tinder-engineering/geosharded-recommendations-part-3-consistency-2d2cb2f0594b))

# 滑动和匹配

![](https://live.staticflickr.com/65535/51824400947_69b4985d5c_h.jpg)

图: 用户滑动和匹配的操作顺序

在上图中，我们展示了用户向左/向右滑动时执行的操作序列。滑动摄取器处理滑动并将左滑动放入流中，该流将这些滑动持久化到低成本的数据存储 (例如 Amazon S3)。这些左滑动可以用于一些用例的数据分析。

另一方面，右滑动被放在一个单独的流中，并最终由匹配器工作线程读取。matcher worker 线程从流中读取点赞消息，并检查对应条目是否存在于 LikesCache 中。例如，在上图中，Alice 喜欢 Bob, match worker 检查缓存中是否存在 Bob 喜欢 Alice 的条目。如果 Alice 和 Bob 都喜欢对方，那么它被称为匹配，并使用服务器推送机制 (如 Websockets) 向两个用户发送匹配通知。如果 Bob 还不喜欢 Alice，则会在 LikesCache 中生成一条 Alice 喜欢 Bob 的记录。

# 匹配数据模型

我们可以使用键值存储 (例如 Amazon DynamoDB) 来持久化匹配信息 (用户喜欢对方)。用于这个数据存储的哈希键可以是互相喜欢的用户的唯一标识符的复合键。数据存储中的值将包含与匹配相关的元数据信息。

![](https://live.staticflickr.com/65535/51825431623_e00189bdf3_z.jpg)

# 用户切换位置

![](https://live.staticflickr.com/65535/51825324321_91226f9c13_b.jpg)

当用户切换位置时，我们希望确保我们从新位置向用户提供推荐，反之亦然。用户位置更新到新位置，以便更新后的位置用于为用户获取推荐。此外，我们还用用户的信息更新映射到用户新位置的索引，这样用户就会出现在位置的推荐中。这个过程是异步执行的。

弹性搜索集群 (下文解释) 包含地理分片索引从队列中读取消息以更新用户的索引。elastic search 协调节点将用户的信息从映射到用户旧位置的索引移动到映射到用户新位置的索引。这将确保该用户在新位置的其他用户的推荐中出现。

# 弹性搜索集群

![](https://live.staticflickr.com/65535/51825436353_c643f49405_z.jpg)

集群将由多个主节点组成，每个主节点有两个 [自动伸缩组(ASG)](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)，一个只包含协调节点 (这是所有请求发送的地方)，另一个包含所有数据节点。每个数据节点将包含一定数量的随机分布的分片的索引 (主节点和副本的组合)。对于每个用户查询，协调节点的责任是查询目标分片的数据节点，以处理用户查询。我们利用用户数据的地理位置对其进行分片，并创建这些分片的副本，从而提高了 elastic search 集群的可靠性和鲁棒性。

# 优化

Tinder 这样的应用程序最重要的一个方面是它为用户提供的 (潜在匹配对象) 推荐。在上面的一个小节中，我们看到了如何通过查询一个用户附近的地理分片对应的索引来为用户生成推荐。我们可以通过应用机器学习对推荐进行排序来优化系统。机器学习模型将优化用户右击推荐潜在匹配项的潜力。我们在下面列出了一些可能影响用户决定向左或向右滑动的特征。

* **用户人口统计数据**: 年龄、性别、种族、位置、职业等
* **用户 Tinder 历史数据**: 向左滑动、向右滑动、历史地理位置、每日使用时间
* **从用户简介中提取的信息**: 喜欢，不喜欢，偏好
* **从用户图片中提取的信息**: 五官、发色、体型

我们可以通过使用这些特征来找出用户向右滑动推荐的概率来构建回归问题。然后，我们可以利用逻辑回归等算法来计算这些概率，这些概率将用于对推荐进行排名。除此之外，我们还可以优化机制，通过预取用户被推荐用户右滑的信息，向用户发送匹配通知。预取这些信息，我们可以立即通知用户匹配情况 (如果发生以及何时发生)，从而防止网络调用。例如，如果 Alice 显示在 Bob 的推荐中，并且 Alice 已经向右滑动了 Bob，那么 Bob 就会得到一个即时的匹配通知 (没有任何网络跳)，以防 Bob 也向右滑动 Alice。

# 参考文献
*   https://medium.com/tinder-engineering/geosharded-recommendations-part-1-sharding-approach-d5d54e0ec77a
*   https://medium.com/tinder-engineering/geosharded-recommendations-part-2-architecture-3396a8a7efb
*   https://medium.com/tinder-engineering/geosharded-recommendations-part-3-consistency-2d2cb2f0594b
*   https://medium.com/tinder-engineering/taming-elasticache-with-auto-discovery-at-scale-dc5e7c4c9ad0
*   https://medium.com/tinder-engineering/how-we-improved-our-performance-using-elasticsearch-plugins-part-1-b0850a7e5224
*   https://medium.com/tinder-engineering/how-we-improved-our-performance-using-elasticsearch-plugins-part-2-b051da2ee85b
*   https://www.youtube.com/watch?v=Lq4aNihcS8A
*   https://www.youtube.com/watch?v=8zh4iUNFjKc
*   https://www.youtube.com/watch?v=o3WXPXDuCSU

[原文](http://highscalability.com/blog/2022/1/11/designing-instagram.html)

更多文章请到 [GitHub - HardwayLinka/cn-system-design: Translate some system design articles into Chinese](https://github.com/HardwayLinka/cn-system-design)，欢迎来点个 star 哦~

# 问题陈述

设计一个类似 Instagram 的照片分享平台，用户可以上传自己的照片，并分享给粉丝。随后，用户将能够查看包含他们所关注的所有其他用户的帖子的个性化 feed。

# 收集需求

## 范围内

应用程序应该能够支持以下需求。

* 用户应该能够上传照片并查看自己上传的照片。

* 用户应该能够关注其他用户。

* 用户可以查看包含他们所关注用户的帖子的 feed。

* 用户应该能够点赞和评论这些帖子。

## 超出范围

* 发送和接收来自其他用户的消息。

* 生成基于机器学习的个性化推荐，以发现与自己兴趣相关的新人物、照片、视频和故事。

# 高层次设计

## 架构

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813606581_b6b3b9d53a_z.jpg)

当服务器从客户端接收到一个操作请求 (post, like 等) 时，它执行两个并行操作: i) 在数据存储中持久化操作 ii) 在 pub-sub 模型的流数据存储中发布操作。之后，各种服务 (例如用户订阅服务，媒体柜台服务) 从流数据存储中读取操作并执行它们的特定任务。==流数据存储使系统具有可扩展性，以支持未来的其他用例== (例如媒体搜索索引、位置搜索索引等)。

**有趣的事实**: 在这次 [演讲](https://www.youtube.com/watch?v=1sPgogJlKWM) 中，Instagram 的工程总监 Rodrigo Schmidt 谈论了他们在扩展 Instagram 的数据基础设施时所面临的不同挑战。

## 系统组件

系统将由几个微服务组成，每个微服务执行一个单独的任务。我们将使用图数据库 (如 Neo4j) 来存储信息。我们选择图数据模型的原因是，我们的数据将包含数据实体之间的复杂关系，如用户、帖子和评论，作为图的节点。在此之后，我们将使用图的边来存储关注、点赞、评论等关系。此外，我们可以使用像 Cassandra 这样的柱状数据库来存储用户订阅、活动和计数器等信息。

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51814333775_fc4aa6f641_z.jpg)

# 组件设计

## 在 Instagram 上发帖

![](https://live.staticflickr.com/65535/51824416827_25bdf72ec6_h.jpg)

图 2: 在 Instagram 上发帖的同步和异步流程

当用户在 Instagram 上发布照片时，主要执行两个进程。首先，同步过程负责在文件存储上上传图像内容，在图数据存储中持久化媒体元数据，向用户返回确认消息并触发该过程更新用户活动。第二个过程是异步发生的，通过在柱状数据存储 (Cassandra) 中持久化用户活动，并触发进程预先计算非名人用户 (拥有几千名粉丝) 的粉丝动态。我们不会对名人用户 (拥有 100 万 + 粉丝) 的订阅进行预计算，因为将订阅分发给所有粉丝的过程将是极其计算和 I/O 密集的。

## API 设计

我们在下面提供了在 Instagram 上发布图片的 API 设计。我们将使用 multipart/form-data 内容类型在一个请求中发送文件和数据。[MultiPart/Form-Data](http://highscalability.com/blog/2022/1/11/www.w3.org/TR/html401/interact/forms.html#h-17.13.4.2) 包含一系列的部分。每个部分预计包含一个 content-disposition 头 [RFC 2183]，其中 disposition 类型为“form-data”。

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813619306_dec26bebed.jpg)

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813962854_de777c6a2e.jpg)

## 预计算 feed

![image.png](https://raw.githubusercontent.com/HardwayLinka/image/master/20230419111313.png)

当非名人用户在 Instagram 上发布帖子时，这个过程就会执行。当一条消息被添加到用户订阅服务队列时，它会被触发。消息添加到队列后，用户 Feed 服务会调用关注者服务，获取该用户的关注者列表。之后，这篇文章就会被添加到列式数据存储中所有关注者的 feed 中。

## 获取用户 Feed

![image.png](https://raw.githubusercontent.com/HardwayLinka/image/master/20230419112005.png)

当用户请求 feed 时，那么将涉及两个并行线程来获取用户 feed，以优化延迟。第一个线程将从用户关注的非名人用户那里获取 feed。这些提要由上面的“预计算提要”一节中描述的扇出机制填充。第二个线程负责获取该用户所关注的名人用户的 feed。之后，User Feed Service 会合并来自名人和非名人用户的 Feed，并将合并后的 Feed 返回给请求 Feed 的用户。

## API 设计

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813650341_970383fb68_z.jpg)

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51814347145_96928483c8.jpg)

# 数据模型

## 图数据模型

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813972019_ddfbe16c97_z.jpg)

我们可以使用图数据库，如 Neo4j，它将用户信息、帖子、评论等数据实体存储为图中的节点。节点之间的边用来存储数据实体之间的关系，比如关注者、帖子、评论、点赞和回复。所有的节点都被添加到一个名为 nodeIndex 的索引中，以实现更快的查找。我们选择了这种基于 NoSQL 的解决方案而不是关系型数据库，因为它提供了超越两层的层次结构的可扩展性，并且由于 NoSQL 数据存储的无模式行为而具有可扩展性。

## 图数据库支持的示例查询

### 获取 Jeff Bezos 的所有关注者

```
Node jeffBezos = nodeIndex.get(“userId”, “user004”);
List jeffBezosFollowers = new ArrayList();

for (Relationship relationship: jeffBezos.getRelationships(INGOING, FOLLOWS)) {
    jeffBezosFollowers.add(relationship.getStartNode());
}
```

### 获取比尔·盖茨的所有帖子

```
Node billGates = nodeIndex.get(“userId”, “user001”);
List billGatesPosts = new ArrayList();

for (Relationship relationship: billGates.getRelationships(OUTGOING, POSTS)) {
    billGatesPosts.add(relationship.getEndNode());
}
```

### 获取所有杰夫·贝佐斯评论过的比尔·盖茨的帖子

```
List commentsOnBillGatesPosts = new ArrayList<>();

for(Node billGatesPost : billGatesPosts) {
     for (Relationship relationship: billGates.getRelationships(INGOING, COMMENTED_ON)) {
    commentsOnBillGatesPosts.add(relationship.getStartNode());
     }
}

List jeffBezosComments = new ArrayList();

for (Relationship relationship: jeffBezos.getRelationships(OUTGOING, COMMENTS)) {
    jeffBezosComments.add(relationship.getEndNode());
}

List jeffBezosCommentsOnBillGatesPosts = commentsOnBillGatesPosts.intersect(jeffBezosComments);
```

# 柱状数据模型

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51813632951_0f0f673287_w.jpg)

我们将使用 Cassandra 等列式数据存储来存储用户提要和活动等数据实体。每行将包含用户的 feed/活动信息。我们还可以有一个基于 TTL 的功能来驱逐旧的帖子。数据模型看起来类似于:

```
User_id -> List
```

**有趣的事实**: 在这次 [演讲](https://www.youtube.com/watch?v=_BfMH4GQWnk) 中，Instagram 核心基础架构团队的软件工程师 Dikang Gu 提到了他们如何使用 Cassandra 来服务关键用例、高可扩展性需求以及一些痛点。

# 流数据模型

我们可以使用云技术，如 [Amazon Kinesis](https://aws.amazon.com/kinesis/) 或 [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) 来收集、处理和分析实时流数据，以获得及时的见解和对新信息的快速反应。一个新的点赞、评论等)。我们在下面列出了一些主要流数据实体和动作的反规范化形式。

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51814358575_69266135f8_z.jpg)

上面的数据实体**A**和**B**显示了包含用户及其帖子的非规范化信息的容器。随后，数据实体**C**和**D**表示用户可能采取的不同操作。实体**C**表示用户点赞一篇文章的事件，实体**D**表示用户关注另一个用户时的动作。这些动作由相关的微服务从流中读取并进行相应的处理。例如，**likeevent** 可以被 **媒体计数器服务** 读取，并用于更新数据存储中的媒体计数。

# 优化

我们将使用一个具有基于 LRU 的驱逐策略的缓存来缓存活跃用户的用户源。这不仅将减少向用户显示用户源的整体延迟，而且还将防止用户源的重新计算。

![](https://raw.githubusercontent.com/HardwayLinka/image/master/51812677532_5206aeb46f_z.jpg)

优化的另一个范围在于，在用户订阅中提供最好的内容。我们可以通过对用户关注的人的新 feed(用户上次登录后生成的 feed) 进行排名来实现这一点。我们可以应用机器学习技术，通过为单个 feed 分配分数来对用户 feed 进行排名，这些分数将表明点击、喜欢、评论等的概率。我们可以通过一个特征向量来表示每个提要，该特征向量包含关于用户、提要以及用户与提要中的人的交互信息 (例如，用户是否点击/喜欢/评论了故事中的人之前的提要)。很明显，feed 排名中最重要的特征将与社交网络相关。下面列出了一些理解用户网络的关键。

* 密切关注的用户是谁?例如，一个用户是埃隆·马斯克的密切关注者，而另一个用户可能是戈登·拉姆齐的密切关注者。

* 该用户总是喜欢谁的照片?

* 用户最感兴趣的是谁的链接?

我们可以使用 [深度神经网络](https://en.wikipedia.org/wiki/Deep_learning)，它将采取我们训练模型所需的几个特征 (> 100K dense features)。这些特征将通过 n-fold 层传递，并将用于预测不同事件 (点赞、评论、分享等) 的概率。

**有趣的事实**: 在这次 [演讲](https://www.youtube.com/watch?v=Xpx5RYNTQvg) 中，Lars Backstrom, Facebook 工程副总裁谈论了为用户创建个性化新闻源所做的机器学习。他谈到了他们在初始阶段使用的经典机器学习方法，通过使用决策树和逻辑回归来个性化新闻源。然后他谈到了他们在使用神经网络时观察到的改进。

# 参考文献

* https://www.youtube.com/watch?v=_BfMH4GQWnk
* https://www.youtube.com/watch?v=hnpzNAPiC0E
* https://instagram-engineering.com/what-powers-instagram-hundreds-of-instances-dozens-of-technologies-adf2e22da2ad
* https://instagram-engineering.com/types-for-python-http-apis-an-instagram-story-d3c3a207fdb7
* http://highscalability.com/blog/2012/4/9/the-instagram-architecture-facebook-bought-for-a-cool-billio.html
* https://docs.oracle.com/cloud/latest/marketingcs_gs/OMCAC/op-api-rest-1.0-assets-image-content-post.html
* https://instagram-engineering.com/under-the-hood-instagram-in-2015-8e8aff5ab7c2
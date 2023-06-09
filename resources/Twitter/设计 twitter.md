[原文](https://www.codekarle.com/system-design/Twitter-system-design.html)

更多文章请到 [GitHub - HardwayLinka/cn-system-design: Translate some system design articles into Chinese](https://github.com/HardwayLinka/cn-system-design)，欢迎来点个 star 哦~

还记得 2011 年那个纯粹偶然地在博客上直播奥萨马突袭行动的家伙吗?不管是政治竞选、社会运动、自然灾害，什么都有!任何重大事件都会首先在推特上报道。

因此，这是一个典型的面试问题——“你会如何设计一个像 Twitter 这样的系统?”那么，让我们来设计 Twitter 吧!

让我们先看看需求。

# 功能需求

- 推文——应该允许你发布文本、图像、视频、链接等
- 转发 - 应该允许你分享别人的推文
- 关注——这将是一种直接的关系。如果我们关注巴拉克·奥巴马，他就不必再关注回我们
- 搜索

# 非功能性需求

- 阅读量大 - twitter 的读写比率非常高，所以我们的系统应该能够支持这种模式
- 快速渲染
- 快速推送
- 延迟是可以接受的 - 从之前的两个 NFRs 中，我们可以理解系统应该是高可用的，并且有非常低的延迟。所以当我们说延迟是可以接受的时候，我们的意思是在几秒钟后收到别人的推文通知是可以的，但是内容的渲染应该是几乎瞬时的
- **可扩展** ——平均每天推特上每秒有 5k+ 条推文发送。在高峰时间，它很容易翻倍。这些只是推文，正如我们已经讨论过的，推特的读写比率非常高，即这些推文会有更高的阅读数量。这是每秒巨大的请求量。

那么我们如何设计一个系统，在不影响性能的情况下满足我们所有的功能需求呢?在讨论整体架构之前，我们先把我们的用户分成不同的类别。每一个类别的处理方式都会略有不同。

1.  **著名用户:** 著名用户通常是名人、运动员、政治家或拥有大量粉丝的商业领袖
2.  **活跃用户**: 这些是在过去几小时或几天内访问过系统的用户。在我们的讨论中，我们将过去三天内访问过 twitter 的用户视为活跃用户。
3. **实时用户**: 这些是目前正在使用系统的活跃用户的子集，类似于 Facebook 或 WhatsApp 上的在线用户。
4.  **被动用户:** 这些是拥有活跃账户但最近三天没有访问系统的用户。
5.  **不活跃用户**: 这些可以说是“已删除”的账户。我们并没有真正删除任何账号，它更像是一种软删除，但就用户而言，这个账号已经不存在了。

现在，为了简单起见，我们把整体架构分为三个流程。我们将分别考察 系统的登入流程、推文流程以及搜索和分析方面。

**注意:** 还记得 twitter 是一个非常重阅读的系统吗?嗯，在设计一个偏重阅读的系统时，我们需要确保我们正在进行尽可能多的预计算和缓存，以保持尽可能低的延迟。

# 系统架构

![image.png](https://raw.githubusercontent.com/HardwayLinka/image/master/20230419155118.png)

# 登入流程

我们有一个 **用户服务 (user service)**，它将在我们的系统中存储所有与用户相关的信息，并通过提供通过 id 或电子邮件获取用户信息的 GET api，添加或编辑用户信息的 POST api，以及获取多个用户信息的 bulk GET api，为登录、注册和任何其他需要用户相关信息的内部服务提供端点。这个用户服务位于 **用户数据库 (user DB)** 之上，这是 **MySQL** 数据库。我们在这里使用 MySQL 是因为我们的用户数量有限，并且数据是非常相关的。此外，用户数据库将主要支持与用户详细信息相关的写入和读取，这将由 **Redis** 缓存提供，它是用户数据库的映像。当用户服务收到带有用户 id 的 GET 请求时，它将首先在 Redis 缓存中查找，如果用户在 Redis 中存在，它将返回响应。否则，它会从用户数据库中获取信息，存储在 Redis 中，然后响应客户端。

# 用户关注流程

与“关注”相关的请求将由 **Graph Service** 提供服务，它创建了一个用户在系统内如何连接的网络。Graph service 将公开 api 来添加关注链接，让用户被某个用户 id 关注或关注某个用户 id 的用户关注。这个图服务位于 **用户图数据库 (user graph db)** 之上，这也是一个 **MySQL 数据库**。同样，follow-links 不会频繁更改，所以将这些信息缓存到 **Redis** 中是有意义的。现在在关注流中，我们可以缓存两条信息——谁是特定用户的 **粉丝**，以及谁是该用户在关注谁。与 user service 类似，当 graph service 接收到 get 请求时，它会首先在 Redis 中查找。如果 Redis 有信息，它会响应给用户。否则，它从图形数据库中获取信息，将其存储在 Redis 中，并响应用户。

现在，根据用户与 Twitter 的互动，我们可以得出一些结论，比如他们的兴趣等等。所以当这样的事件发生时，一个 **分析服务 (Analytics Service)** 会把这些事件放在一个 **Kafka** 中。

现在，还记得我们的实时用户吗? 假设 U1 是一个关注 U2 的直播用户，U2 发了一些推文。既然 U1 是直播的，那么 U1 立即得到通知就说得通了。这是通过 **用户实时 Websocket 服务 (User Live Websocket service)** 来实现的。该服务与所有实时用户保持开放连接，每当发生需要通知实时用户的事件时，它都会通过该服务发生。现在根据用户与这个服务的交互，我们还可以跟踪用户在线多长时间，当交互停止时，我们可以得出用户不再在线的结论。当用户下线时，通过 websocket 服务，一个事件将被触发到 Kafka, Kafka 将进一步与用户服务交互，并在 Redis 中保存用户的最后活动时间，其他系统可以使用这些信息来相应地修改他们的行为。

# 推文流程

现在一条推文可以包含文本、图片、视频或链接。我们有一种叫做 **资产服务 (Asset service)** 的东西，它负责上传和显示推文中所有的多媒体内容。我们已经在 [Netflix设计文章](https://www.codekarle.com/system-design/netflix-system-design.html) 中讨论了资产服务的细节，感兴趣的可以去看看。

现在，我们知道推文有 140 个字符的限制，其中可以包括文本和链接。由于这个限制，我们不能在推文中发布巨大的 url。这就是 **网址缩短服务 (URL shortener service)** 的作用。这个服务的工作原理我们就不详细介绍了，我们已经在 [微URL文章](https://www.codekarle.com/system-design/TinyUrl-system-design.html) 中讨论过了，大家一定要去看看。现在我们也处理好了链接，剩下的就是存储推文的文本并在需要的时候获取。这就是 **推文采集服务 (tweet ingestion service)** 的作用。当用户试图发布 tweet 并点击提交按钮时，它会调用 tweet 采集服务，该服务将 tweet 存储在永久数据存储中。我们在这里使用 **Cassandra**，因为我们每天都会有大量的推文输入，我们在这里需要的查询模式是 Cassandra 最适合的。要了解更多关于我们为什么使用数据库解决方案，我们可以查看我们关于 [选择最佳存储解决方案](https://www.codekarle.com/system-design/Database-system-design.html) 的文章。

![](https://raw.githubusercontent.com/HardwayLinka/image/master/twitter-system-design-post-tweet.svg)

现在，**推文摄取服务(tweet ingestion service)**，顾名思义，只负责发布推文，不暴露任何 GET api 来获取推文。一条推文一经发布，推文摄取服务就会向 **Kafka** 触发一个事件，说明一条推文 id 是由某某用户 id 发布的。现在，在我们的 Cassandra 之上，有一个 **推文服务 (Tweet service)**，它将公开 api，通过推文 id 或用户 id 获取推文。

现在，让我们快速浏览一下用户方面的内容。在读取流中，用户可以有一个用户时间轴 (即来自该用户的推文) 或家庭时间轴 (即来自用户所关注的人的推文)。现在一个用户可能有一个他们所关注的用户的巨大列表，如果我们在显示时间线之前在运行时进行所有查询，它将会减慢渲染速度。所以我们转而缓存用户的时间轴。我们将预先计算活跃用户的时间轴，并将其缓存在一个 **Redis** 中，这样活跃用户就可以立即看到他们的时间轴。这可以通过一个叫做 **推文处理器 (Tweet processor)** 的东西来实现。

如前所述，tweet 发布后，Kafka 会触发一个事件。Kafka 将同样的信息传递给推文处理器，并为所有需要收到这条最近推文通知的用户创建时间轴，并缓存它。为了找出需要通知此更改的关注者，tweet 服务与图服务进行交互。假设用户 U1，随后是用户 U2、U3 和 U4 发布了一条推文 T1，那么推文处理器将用推文 T1 更新 U2、U3 和 U4 的时间线并更新缓存。

现在，我们只缓存了活跃用户的时间线。当一个被动用户，比如说 P1，登录系统会发生什么?这时候 **时间线服务 (Timeline Service)** 就派上用场了。请求会到达 **时间线服务**，时间线服务会与用户服务进行交互，识别 P1 是主动用户还是被动用户。现在由于 P1 是被动用户，它的时间线不会在 Redis 中缓存。现在 timeline 服务会和 graph 服务对话，找到 P1 关注的用户列表，然后查询 tweet 服务，获取所有这些用户的推文，缓存在 Redis 中，并返回给客户端。

现在我们已经看到了主动用户和被动用户的行为。我们将如何为我们的实时用户优化流量?正如我们之前讨论过的，当一条推文成功发布后，一个事件将被发送到 Kafka。然后 Kafka 将与 tweet 处理器进行对话，该处理器为活跃用户创建时间线并将其保存在 Redis 中。但在这里，如果 tweet 处理器识别出其中一个需要更新的用户是实时用户，那么它将触发一个事件到 Kafka，该事件将与我们之前简要讨论的实时 websocket 服务进行交互。这个 websocket 服务现在将发送一个通知到应用程序并更新时间轴。

所以现在我们的系统可以成功地发布不同类型内容的推文，并且内置了一些优化，以某种不同的方式处理主动、被动和实时用户。但它仍然是一个相当低效的系统。为什么?因为我们完全忘记了我们的著名用户!如果唐纳德·特朗普有 7500 万粉丝，那么特朗普每发一条推特，我们的系统就需要做 7500 万次更新。而这只是一个用户的一条推文。所以这个流程对我们著名的用户来说是行不通的。

Redis cache 只会在预先计算的时间线内缓存非知名用户的推文。Timeline service 知道 Redis 只存储普通用户的推文。它与 graph service 交互，获取我们当前用户 (比如 U1) 关注的著名用户列表，并从 tweet service 获取他们的推文。然后它会在 Redis 中更新这些推文，并添加一个时间戳来指示时间线最后一次更新的时间。当下一个请求来自 U1 时，它会检查 Redis 中针对 U1 的时间戳是否来自几分钟前。如果是，它将再次查询推文服务。但如果时间戳是相当近期的，Redis 将直接响应应用程序。

现在我们已经处理了主动用户、被动用户、live 用户和著名用户。至于不活跃的用户，他们已经是停用账号了，所以我们不用担心。

现在，如果一个著名用户关注了另一个著名用户，比如唐纳德·特朗普和埃隆·马斯克，会发生什么?如果唐纳德·特朗普发了推文，即使其他非知名用户没有得到通知，埃隆·马斯克也应该立即得到通知。这又是由推文处理器处理的。推文处理器，当它从 Kafa 收到一个关于著名用户的新推文的事件，比如说唐纳德·特朗普，就会更新关注特朗普的著名用户的缓存。

现在，这看起来是一个相当高效的系统，但是存在一些瓶颈。比如 Cassandra——它将承受巨大的负载，Redis——它需要高效地扩展，因为它完全存储在 RAM 中，以及 Kafka——它将再次接收大量的事件。因此，我们需要确保这些组件是水平可扩展的，对于 Redis 来说，不要存储只是不必要地消耗内存的旧数据。

现在来谈谈**搜索**和**分析!**

还记得我们在上一节中讨论的 **tweet 摄取服务**吗?当一条推文被添加到系统时，它会向 Kafka 触发一个事件。一个收听 Kafka 的搜索消费者将所有这些传入的推文存储到一个**Elasticsearch**数据库中。现在，当用户在**搜索 UI**中搜索一个字符串时，它会与**搜索服务**对话。搜索服务会和 elastic Search 对话，获取结果，然后返回给用户。

现在假设发生了一个事件，人们在 Twitter 上发布或搜索它，那么可以肯定的是，会有更多的人搜索它。现在我们不应该在 elasticsearch 上一次又一次地查询相同的东西。一旦搜索服务从 elasticsearch 得到一些结果，它将以 2-3 分钟的生存时间将它们保存在 Redis 中。现在，当用户搜索一些东西时，搜索服务将首先在 Redis 中查找。如果在 Redis 中找到数据，它将返回给用户，否则，搜索服务将查询 elasticsearch，获取数据，将其存储在 Redis 中，并返回给用户。这大大减少了 elasticsearch 的负载。

让我们再次回到 Kafka。会有一个**spark streaming consumer**连接到 Kafka，它会跟踪热门关键词，并将它们传达给**Trends service**。这可以进一步连接到**趋势 UI**来可视化这些数据。我们不需要永久的数据存储来存储这些信息，因为趋势是暂时的，但我们可以使用 Redis 作为短期存储的缓存。

现在你一定注意到了，我们在设计中大量使用了 Redis。现在，即使 Redis 是一个内存中的解决方案，仍然可以选择将数据保存到磁盘。所以万一出现宕机，如果有些机器宕机了，你还是会把数据持久化到磁盘上，让它的容错性更强一点。

现在，除了趋势，还有一些其他的分析可以执行，比如来自印度的人在谈论什么。为此，我们将在一个**Hadoop 集群**中转储所有传入的推文，这可以支持像转发次数最多的帖子等查询。我们还可以在 Hadoop 集群上运行一个**每周 cron 任务**，它将提取我们的被动用户的信息，并向他们发送每周简报，其中包括他们可能感兴趣的一些最新的推文。这可以通过运行一些简单的 ML 算法来实现，这些算法可以根据用户之前的搜索和阅读来判断推文的相关性。时事通讯可以通过通知服务发送，该服务可以与用户服务对话，以获取用户的电子邮件 id。

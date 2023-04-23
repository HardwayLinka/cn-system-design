[原文](https://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491988&idx=1&sn=d1f28063c64ce619251d3af175a9e8ed&chksm=ebdb9106dcac181074e664617a2f336a307636768f24f5f39d66fed25c33eaf3a3d75c209772&scene=178&cur_album_id=1500522652925526016#rd)

# 一、背景

现在日常官网商城的运营中有一定概率出现以下两个问题：

**1）优惠信息未对齐**

官网商城促销优惠的类型越来越多，能影响最终用户实付价的优惠就有抢购、满减、优惠券、代金券等。实际业务操作中存在不同促销优惠由不同运营配置的情况，如果运营间内部没有对齐的情况下，就会出现正常情况下不会同时设置的优惠被用户叠加享受，出现最终实付价低于成本价的可能。

**2）优惠价格配错**

在日常或大促优惠配置中，存在一定的概率会配错优惠价格（极端场景下，一口价少了个 0，这就相当于在原来预期的优惠价基础上打了一折），这种情况一旦发生可能会引发用户疯狂下单，造成非常大的损失，这也是我们平时说的 “薅羊毛”

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsgwHsniaQicRnjktGGyvybPQEDBLXPJ3USOwpia5dOI2rib7DmECJNDr38qw/640?wx_fmt=jpeg)

（引用自英国每日邮报，摄影 - AIan Price）  

针对前述两种情况，我们希望能够对于出现低于运营所设「底价阈值」的下单购买行为能进行一定预警，必要时能购阻断用户下单行为，及时止损，如果能提前规避这些行为就更好了。

## 二、营销价格能力矩阵

想要解决背景中所遇到的问题，我们先简单了解下营销价格能力矩阵的规划建设。

通过《[vivo 商城计价中心 - 从容应对复杂场景价格计算](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491732&idx=1&sn=2dad67f0a646701123c2d68b4ff5abfb&chksm=ebdb9006dcac1910f08ec047f375b9a0863d6ac824275d0a17bc1afd0fdac2e93710eb9be829&scene=21#wechat_redirect)》、《[vivo 全球商城时光机 - 大型促销活动保障利器](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491752&idx=1&sn=ac5367bbe7614eaec2556692be0e5cda&chksm=ebdb903adcac192c46bde69fc74dbde56cecf8b0eebe39d52ffe256aedb5ce942332c6faa2ba&scene=21#wechat_redirect)》，我们了解到官网商城的营销价格服务已经统一收口到促销的计价中心，在计价中心建设的过程中也发现一些问题：

* 计价中心的业务定位是商城购物链路中的下单商品**实时价格的计算**，而有些接入计价的业务（如官网商品列表）其实对于商品实时价格要求并没有那么高或者准实时的优惠价格在业务上也是能接受的。

* 运营同学在维护相关优惠或配置相关优惠券时，无法方便感知在未来某一时刻某商品所享受的优惠信息或者某一时刻商品的**最低价格能到多少**，也就会出现了不同运营配置了多重优惠导致实际售卖价格低于预期。

* 商城售卖的商品在某一时间段内的实际优惠后的**价格没有历史记录**，对于运营回顾历史数据无法提供实质帮助。

* 若后续平台对于大促期间商品价格**承诺 xx 天内保价**也无从做起，没有数据作参照比对。

针对目前已有的场景及未来可预见的场景，打破眼下仅有实时优惠价的局限，通过对**未来优惠**、**准实时优惠**、**历史优惠**的业务功能的不断补充建设，逐步完善官网商城商品优惠的多维度建设，形成一个围绕商品 SKU **优惠价格的业务能力矩阵**，进一步提升促销系统的业务价值。

可以通过如下的业务架构图来描述我们的营销价格能力矩阵规划：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5euQKicgJA9sDtaMR4aTP28wgfxCicsoKZibjAEibflOeK6nFVOyAia0guycmafibMkbCm6OKLiaXDq3ksQ/640?wx_fmt=png)

# 三、 价格监控

## 3.1 目的

结合「**商城营销价格能力矩阵**」规划的能力，希望能达成：

* 提升运营配置优惠活动的准确性 （事前）

* 提供多维度策略供运营决策       （事中）

* 提供相关营销价格数据供挖掘    （事后）

## 3.2 方案

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5euQKicgJA9sDtaMR4aTP28AKgIibLhic3uddh4O53Ure4JwYESj5nTmseGhXFRlEWMmtgmvqZNGuMQ/640?wx_fmt=png)

### **3.2.1 事前**

**a. 提前规避**

* 优惠互斥设置

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsggoISicIdP1HzicaufNmd2w5IOicycAYibJNzQBVySSP1HEILXnKw7PRWYg/640?wx_fmt=png)

对于默认可共融叠加的优惠提供是否与其他优惠互斥配置；该配置适用于运营在配置营销优惠时确认当前优惠不与其他类型优惠同享。

* 设置 SKU 底价阈值

支持按照价格绝对值或折扣比例两种方案来设置，如原价 1000 元的 SKU，按价格绝对值可以设置 750 元的底价，或按折扣比例设置 75 折作为底价。（这个操作非常关键，是事前和事中方案中一些监控手段的大前提）。

**b. 提前提醒**

* 设置活动优惠价时，出现低于底价阈值的进行及时提醒。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsggvFh16ZqgqjWwia0gBhSjtUUjibqafxiaZLXOnrRFg6ol3aHzYG24mvxQ/640?wx_fmt=png)

**c. 提前告警**

* 定时巡检商品，预警未来时间点的优惠价低于阈值的。

针对所有设置底价阈值商品的巡检工作流程如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5euQKicgJA9sDtaMR4aTP28Bpsjcg2Sp5uZib83RKOx8z3HxkyYCDf6dZFYOUKjuYpGmYcnV808DDw/640?wx_fmt=png)

如果发现了低于底价阈值的情况，则会通过内部通讯工具立即通知相关人员进行及时处理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsgFYsPddJibE0M2uNmBAFn6ye4QYiaVzSD6WyNbX7sUY0z4RJFg3T0hUBA/640?wx_fmt=png)

### **3.2.2 事中**

**a. 优惠生效及时预警**

优惠活动生效时第一时间预警低于阈值的信息。

优惠生效及时预警的处理流程如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5euQKicgJA9sDtaMR4aTP28ErAS5o029jOTia3cibciaM4n6yRsno7srapXTTfibXIWIRIRSctRRK4Uyw/640?wx_fmt=png)

如果发现有需要告警的通知，则向运营相关的同学发出如下通知：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsgMOMnX5SiaVLPgO3V2o7GjXWbAM1HPcD1YD9E5ia5f8rTc7HNQ8giaUVyw/640?wx_fmt=png)

**b. 实时监控下单行为**

监控每个 SKU 实时下单优惠价，根据策略或告警或阻断下单行为。  

实时监控下单处理流程如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5euQKicgJA9sDtaMR4aTP28Rhic14SAqkqKnHo3IjADQ4wavlhgezZ2aUscH3lYWBCscBvuvBnmHiaw/640?wx_fmt=png)

实时下单经过计价中心处理时，如果发现低于底价阈值，则会发出如下告警信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt4OTWID6K5DMjEwTWCYScsgFnl10Oha6ibohZRxV7gMlYGfbF15xL2uxBvXQiaHQIvL6bYPicNXPAqVQ/640?wx_fmt=png)

另外价格监控还提供了一系列阻断下单策略，当符合预设条件时，会直接阻断正常下单流程，以减少不必要的损失。另外由于阻断下单这一行为性质很严重，所以针对是否开启阻断下单这一行为专门设置了全局性的阻断下单开关，由运营灵活掌控。

### **3.2.3 事后**

**a. 历史营销价分析**

* 查询历史优惠价格走势

* 沉淀历史优惠价供运营分析决策

**b. 价保 xx 天**

* 承诺低价保证

**c. 下单最低价提醒**

* 商详页到手价低价提醒

* 结算页低价提醒

# 四、最后

通过前述方案中的事前及事中两个维度的执行，运营基本能在发生问题的第一时间接到系统的通知，极端场景下满足预设的条件可以直接阻断用户下单，避免损失扩大。

在使用的过程中我们要避免” 狼来了 “这样对告警通知麻木的情况，因此为解决这个问题，我们可以对于告警信息进行一个闭环处理，对于每一个告警信息需要做处理，哪怕是事后处理，要区分告警原因，是因为系统误报还是确实优惠设置有问题等等，逐渐习惯于对每一个告警信息都能保持关注和及时响应，把所有可能存在的问题都在事前阶段就暴露出来。

END

猜你喜欢

* [vivo 商城 - 从容应对复杂场景价格计算](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491732&idx=1&sn=2dad67f0a646701123c2d68b4ff5abfb&chksm=ebdb9006dcac1910f08ec047f375b9a0863d6ac824275d0a17bc1afd0fdac2e93710eb9be829&scene=21#wechat_redirect)

* [vivo 全球商城：优惠券系统架构设计与实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491631&idx=1&sn=b53c69df13a80c6bf37531e0c3788188&chksm=ebdb90bddcac19ab3dae9eb635089644d6032dfc4394c3f9bcfedf837060eaeb8c5a4d70c532&scene=21#wechat_redirect)

* [vivo 商城促销系统架构设计与实践 - 概览篇](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491367&idx=1&sn=2d652011d0405b24775fa3f0b75a15ac&chksm=ebd86fb5dcafe6a3884271600da673d5feb3ae9ef8e19565f8b7b30758fc04b769a20cfb20e7&scene=21#wechat_redirect)
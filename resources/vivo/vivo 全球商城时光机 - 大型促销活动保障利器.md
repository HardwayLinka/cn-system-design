[原文](https://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491752&idx=1&sn=ac5367bbe7614eaec2556692be0e5cda&chksm=ebdb903adcac192c46bde69fc74dbde56cecf8b0eebe39d52ffe256aedb5ce942332c6faa2ba&scene=178&cur_album_id=1500522652925526016#rd)
作者：vivo 官网商城开发团队 - Wei Fuping

# 一、背景

官网商城在双 11、双 12 等大促期间运营同学会精心设计许多给到用户福利的促销活动，当促销活动花样越来越多后就会涉及到很多的运营配置工作 (如指定活动有效期，指定活动启停状态，指定活动参与商品等等）。

如果因为某些原因导致其中部分配置未按预期配置，等到大促那一刻才发现配置没有正确配置，这样大概率会流失不少订单，同样也可能会出现错配优惠导致一些本不该享受的优惠也被用户享受到，可能会给商城带来比较大的损失，因此为了尽量减小前面这些情况的发生的概率，我们就想能不能提供一种能力，让运营同学在重要的电商大促正式开始前，提前去校验所有期待的优惠是否配置正确。

# 二、构思

想让运营同学能去校验所配的大促优惠是否正常，同时又希望不会增加多余的额外工作，如何做到呢？

考虑到电商业务的特殊性，所配置的各种大促优惠最终主要都会体现在优惠后的价格上，因此我们考虑从这个角度去实现。

在电商的核心链路上，主要有商详页、购物车、确认订单、提交订单这几个核心场景，那么只需在这几个场景中实现提前看到优惠后的价格即可判断大促优惠是否配置正确。

**那现在的关键问题是如何做到「提前」看到呢？** 在前序的促销系列文章我们介绍了 [计价中心的建设](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491732&idx=1&sn=2dad67f0a646701123c2d68b4ff5abfb&chksm=ebdb9006dcac1910f08ec047f375b9a0863d6ac824275d0a17bc1afd0fdac2e93710eb9be829&scene=21#wechat_redirect)，计价中心统一收口了所有的优惠价的计算能力，因此我们只要让计价中心能提供「提前」的能力即可。

计价中心计算优惠价正常只会实时计算当前时间商品能够享受的各种优惠，并将最终优惠价告诉上游业务方，所以我们能让计价中心能够计算「**未来某个时间点**」的优惠价即可，而计价中心在计算优惠价时，依赖的一个关键信息是「**当前时间**」，因此我们只要将所谓的「当前时间」进行「篡改」变成未来的某个时间点，实现我们所谓的穿越的目的。

还有一个极为重要的点需要关注，也是这个穿越能力的大前提，就是不能影响线上正常交易，即不能让正常的普通用户也「提前」看到未来的优惠价。

因此如何做到既让运营体验又不影响正常用户呢？我们考虑采用白名单机制，**只针对已登录且用户 id 在白名单中的用户**才能进行所谓的穿越体验。

在确定大体思路后，还有一些问题需要确认：

**1. 对于穿越的完整体验是否只需要商城购物流程？**

如果需体验大促期间整个官网商城的所有氛围，可能涉及改动的点较为多，比如大促宣传活动页面、专属聚合类商品页面，简化版的只关注整个购物下单流程。

**2. 整个穿越过程是否需要真的要真实创建订单？**

由于穿越时光后，用户的下单时间和确认订单的时间是一致的，因此确认订单页的所有优惠及最终的价格是真正的所见即所得，无需真实下单即可获知所有优惠活动信息

所以在提交订单的时候建议直接阻断并提醒用户 “您当前处于时空穿越，请回到现实中再下单哦”，并不作真正的创建订单，也就不会作后续许多写资源的连锁操作，同时这种情况下也会减少很多不必要的改动点。

**3. 对于穿越过程中领取的用户特殊券是否需要作特殊标识？**

* a) 穿越过程中领取的券，如果作了特殊标识，那么退出时光机后，到了优惠券真实可用期后，应建议不作使用，防止占用普通用户资源，同时这种情况下也不建议增加优惠券已发券数量。

* b) 穿越过程中领取的券，不作特殊标识，那么退出时光机后使用该券与其他正常领取的券并无差别，这种情况算是占用了普通用户资源，那么相应的也建议增加优惠券已发券数量上。

a 方案需要优惠券系统作相关的适配改动，但线上真实资源无任何污染或占用；b 方案无需作任何改动，但会侵占极少量真实资源，**如果运营方觉得问题不大建议采用 b 方案，从项目角度成本最小。**

# 三、实现

## 3.1 核心流程图

根据前述的构思方案，得出如下商城穿越核心购物流程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5CVP389KY4tCJYBFYwnVtE20JqcBRac81p7ibr3RnMcztOxtZOPL6ArA9DJHqkEbhrEtghIz02xbw/640?wx_fmt=png)

## 3.2 改造重点

从上述流程图中可以看出改造的重点：

* 白名单信息的维护

* 获取「当前时间」

### **3.2.1 白名单信息维护**

为方便后续穿越用户时间信息共享，我们将此信息（openId: travelTime）存储在配置中心中，并提供相应的管理台方便设置穿越用户及穿越时间点。

### **3.2.2 获取「当前时间」**

整个上下游关联系统可能都会需要获取「当前时间」，而获取「当前时间」需要能获取到配置的白名单信息以及当前用户信息。显然为了各个业务系统能尽可能减少代码变动，==获取「当前时间」适合做到一个公共模块中==，各个业务系统依赖这个公共模块自动具备能获取所期待的「当前时间」。

因此集成了时光机模块后的整个业务系统链路关系如下所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5CVP389KY4tCJYBFYwnVtEibcvkwxPx4RXo16D75ZPH8juB4DqtsNN7ibouKibD8QywHbibqGaXKUECw/640?wx_fmt=png)

### **3.2.3 时光机模块**

从前述内容，我们可以得出时光机模块（vivo-xxx-time-travel）中需要包含的主要能力：

* a ) 穿越用户白名单信息

* b ) 获取「当前时间」

* c ) 读取、设置上下文 openId

其中 a、b 的实现都比较简单，只需正常接入公司的配置中心，并根据指定 openId 获取「当前时间」即可，比较麻烦一点的是获取「当前时间」时的这个用户 openId 信息。

之前的各个业务系统间的接口调用可能是不需要用户 openId 信息的，但现在穿越用户是指定白名单用户的，所以必须要将入口链路检测到的用户 openId 信息一路向下传递到下游的各个业务系统中。

**方案一：** 各个业务系统间接口调用耦合 openId 信息，需要各个业务系统全部都改造一遍，显然这个方案比较初级原始也对各业务方非常不友好，非常不建议采用。

**方案二：** 由于我们后端各个业务系统间都使用 dubbo 进行接口调用，因此我们可以利用 dubbo 基于 spi 插件机制的定制业务过滤器将 openId 当作附加接口调用时的附加信息进行透传。（如果是其他接口调用方式的，也建议采用类似原理的处理方式）

下面我们就看下时光机模块中一些核心的代码实现：（当前业务系统作为消费方时执行的过滤器）

```java
/**
 * 当前业务系统作为消费方时执行的过滤器
 */
@Activate(group = Constants.CONSUMER)
public class BizConsumerFilter implements Filter {
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        if (invocation instanceof RpcInvocation) {
            String openId = invocation.getAttachment("tc_xxx_travel_openId");
            if (openId == null && TimeTravelUtil.getContextOpenId() != null) {
                // 作为消费方在发起调用前，如果缺失openId,则设置上下文的openId
                ((RpcInvocation) invocation).setAttachment(openIdAttachmentKey, TimeTravelUtil.getContextOpenId());
            }
        }
        return invoker.invoke(invocation);
    }
}
```

当前业务系统作为服务提供方执行的过滤器；

```java
/**
 * 当前业务系统作为服务提供方时执行的过滤器
 */
@Activate(group = Constants.PROVIDER)
public class BizProviderFilter implements Filter {
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        if (invocation instanceof RpcInvocation) {
            String openId = invocation.getAttachment("tc_xxx_travel_openId");
            if (openId != null) {// 作为下游服务提供方，获取上游系统设置的上下文openId
                TimeTravelUtil.setContextOpenId(openId);
            }
        }
        try {
            return invoker.invoke(invocation);
        } finally {
            TimeTravelUtil.removeContextOpenId();
        }
    }
}
```

穿越时间获取工具类；

```java
/**
 * 穿越时间获取工具类
 */
public final class TimeTravelUtil {

    private static final ThreadLocal<TimeTravelInfo> currentUserTimeTravelInfoThreadLocal = new ThreadLocal<>();
    private static final ThreadLocal<String> contextOpenId = new ThreadLocal<>();

    public static void setContextOpenId(String openId) {
        contextOpenId.set(openId);
        setUserTravelInfoIfExists(openId);
    }

    public static String getContextOpenId() {
        return contextOpenId.get();
    }

    public static void removeContextOpenId() {
        contextOpenId.remove();
        removeUserTimeTravelInfo();
    }

    /**
     * 设置当前上下文用户穿越信息，如果存在的话
     * @param openId
     */
    public static void setUserTravelInfoIfExists(String openId) {
        // TimeTravellersConfig 会接入配置中心，承载所有白名单穿越用户信息配置，并将每一个穿越用户信息转换为TimeTravelInfo
        TimeTravelInfo userTimeTravelInfo = TimeTravellersConfig.getUserTimeTravelInfo(openId);
        if (userTimeTravelInfo.isInTravel()) {
            currentUserTimeTravelInfoThreadLocal.set(userTimeTravelInfo);
        }
    }

    /**
     * 移除当前上下文用户穿越信息
     */
    public static void removeUserTimeTravelInfo() {
        currentUserTimeTravelInfoThreadLocal.remove();
    }

    /**
     * 当前链路上下文是否处于穿越中
     * @return
     */
    public static boolean isInTimeTravel() {
        return currentUserTimeTravelInfoThreadLocal.get() != null;
    }

    /**
     * 获取「当前」时间，单位：毫秒。
     * 若当前是穿越中，则返回设置的穿越时间,否则返回实际系统时间
     * @return
     */
    public static long getNow() {
        TimeTravelInfo travelInfo = currentUserTimeTravelInfoThreadLocal.get();
        return travelInfo != null ? travelInfo.getTravelTime() : System.currentTimeMillis();
    }

}
```

用户穿越信息

```java
/**
 * 用户穿越信息
 */
public class TimeTravelInfo {
    /**
     * 当前是否处于穿越中
     */
    private boolean isInTravel = false;

    /**
     * 当前穿越时间点，仅在isInTravel=true时有效
     */
    private Long travelTime;

    public boolean isInTravel() {
        return isInTravel;
    }

    public void setInTravel(boolean inTravel) {
        isInTravel = inTravel;
    }

    public Long getTravelTime() {
        return travelTime;
    }

    public void setTravelTime(Long travelTime) {
        this.travelTime = travelTime;
    }
}
```

在业务系统依赖这个 vivo-xxx-time-travel 模块后，凡是需要获取当前时间的地方将原来的 System.currentTimeMillis()

改为 TimeTravelUtil.getNow() 即可。

## **3.4 问题**

在时光机能力建设过程中碰到一个比较重要的问题，就是上下文传递 openId 信息时，会出现跨线程传递丢失问题。

如果底层是 Java 线程池直接实现异步调用，那通过对线程池相关拦截可以实现上下文复制拷贝传递，我们内部的全链路系统已经通过相关代理技术对线程上下文信息已作了相关处理。

如果使用 Hystrix 实现异步调用，可以看下笔者另一篇专门介绍的文章《[Hystrix 中如何解决 ThreadLocal 信息丢失](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247488123&idx=1&sn=5aa14d82628e43a34decbfee5ad3e539&chksm=ebd862e9dcafebff94e17faf3c258cbfb2f042ee22003a0e05e7635ffc8f9b8c0567e44fccba&scene=21#wechat_redirect)》 。

# 四、最后

本文介绍的时光机相关能力主要应用在官网商城，但并不局限于电商场景，时光机模块在设计的时候就没有与某个具体业务耦合，因此对于其他一些业务场景也可以适用或者有一些借鉴意义。

另外本文中电商场景中关注的是优惠价格是否正常，基本涉及到的是读操作，如果有些场景需要穿越后进行完整的业务功能操作，如进行实际下单，那么就会涉及到一些写操作，此时可以借助影子库的相关能力去完成完整的穿越操作之旅。

END

猜你喜欢

* [vivo 全球商城计价中心 - 应对复杂场景价格计算](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491732&idx=1&sn=2dad67f0a646701123c2d68b4ff5abfb&chksm=ebdb9006dcac1910f08ec047f375b9a0863d6ac824275d0a17bc1afd0fdac2e93710eb9be829&scene=21#wechat_redirect)

* [vivo 全球商城：优惠券系统架构设计与实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491631&idx=1&sn=b53c69df13a80c6bf37531e0c3788188&chksm=ebdb90bddcac19ab3dae9eb635089644d6032dfc4394c3f9bcfedf837060eaeb8c5a4d70c532&scene=21#wechat_redirect)

* [vivo 全球商城促销系统架构设计与实践 - 概览篇](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491367&idx=1&sn=2d652011d0405b24775fa3f0b75a15ac&chksm=ebd86fb5dcafe6a3884271600da673d5feb3ae9ef8e19565f8b7b30758fc04b769a20cfb20e7&scene=21#wechat_redirect)
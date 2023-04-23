[原文](https://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491732&idx=1&sn=2dad67f0a646701123c2d68b4ff5abfb&chksm=ebdb9006dcac1910f08ec047f375b9a0863d6ac824275d0a17bc1afd0fdac2e93710eb9be829&scene=178&cur_album_id=1500522652925526016#rd)
作者：vivo 互联网服务器团队 - Wei Fuping

# 一、背景

随着 vivo 商城的业务架构不断升级，整个商城较为复杂多变的营销玩法被拆分到独立的促销系统中。

拆分后的促销系统初期只是负责了营销活动玩法的维护，促销中最为重要的计价业务仍然遗留在商城主站业务中，且由于历史建设问题，商城核心交易链路中商详页、购物车、下单这三块关于计价逻辑是分开独立维护的，没有统一，显然随着促销优惠的增加或者玩法的变动，商城侧业务重复开发量会显著加大。

促销系统的独立，计价相关业务能力从业务边界上也应由促销系统提供，因此促销侧需要从头开始设计促销计价相关能力。

# 二、原有计价业务

## 2.1 计价业务场景

商城原有涉及到计价业务的主要是商详页、购物车、确认下单、提交订单这几个业务场景。

如果将每一个影响最终售卖价的优惠叫做计价因子的话，那前述几种场景下对于售卖价有影响的计价因子归为三大类：

* 优惠活动（单品优惠、订单优惠）

* 优惠券（优惠券、代金券）

* 虚拟抵扣（积分、换新鼓励金）

对于每种计价场景与计价因子有如下关系：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJg81gFRMGZwHoCsHHrHj4QpTRGLvtgOrU22ukQicicDfqtWkRSVZPXSWJA/640?wx_fmt=png)

## 2.2 原有计价模型

对于具体执行的计价业务中各计价因子间是有一定的先后优先级关系的，综合如下图所示，也在一定程度说明了原有计价业务模型：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgrlC5d8NnG3IOTick8eRuUBszQglLWsSibzMovCWKCBsGGfvRIEBBnTRg/640?wx_fmt=png)

# 三、促销计价模型

## 3.1 分层模型

促销系统从零搭建基础计价能力，对于系统的稳定性及扩展性必须有一定的保障，而这也就对于促销系统的计价模型提出了一定的要求，通用的基础计价模型最好是能有过一定的实践经历验证过的，因此我们采用了传统电商久经考验的计价模型：分层计价。

所谓的分层计价即传统电商中优惠涉及的三个层面：商品级、店铺级、平台级，正常情况下**不同级别的优惠默认是可以叠加的，同一级别的优惠默认情况下是互斥的**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgapS2KzibzqpOVU98SvTlS4EJwV9ib8ONUmzPA0GF9d9jJ1xO6u70dmpg/640?wx_fmt=png)

这里需要说明的是，每一层级的优惠计算的时候，对于有些优惠的门槛条件是否满足需要依赖原价，**默认情况下依赖于上一个层级的优惠计算后的价格**，即商品级优惠计算依赖商品原价，店铺级优惠依赖于商品级优惠计算后的价格，平台级优惠依赖于店铺级优惠计算后的价格。

**叠加规则特别说明：**

正常优惠叠加是指两个优惠可以同时享受，对于不同层级的优惠默认就是叠加的，对于同一层级的优惠默认是不叠加的，比如正常情况下，优惠券下的各种类型券是只能用一张的。

但某些场景下，业务上会指定同一层级的优惠可以叠加使用的，同时指定叠加使用的场景下还会分为普通叠加和并行叠加，举个例子：订单优惠和优惠券这两个类型的叠加就属于普通叠加（优惠券门槛是否满足的判断取决于订单优惠后的价格），优惠券和代金券的叠加属于并行叠加（优惠券和代金券的门槛是否满足的判断都取决于这两者的前序优惠后的价格）。

对于同一层级的优惠按不同维度分为：**必选 / 勾选、可叠加 (并行叠加 / 普通叠加)/ 不可叠加 。**

## 3.2 新的计价模型

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgWtbI0AcFQ2xpye8WenzIr7bibF3oxfzL5KriaaCHiczdTIZrlUI0uvicug/640?wx_fmt=png)

## 3.3 核心计价流程

### **3.3.1 主流程**

通过前述计价模型可以得知，在计算优惠价时的先后顺序是：商品级 (CalcItem)、店铺级 (CalcShop)、平台级 (CalcGroup)，另外根据一些特殊业务场景，增加了可能的中断业务逻辑 (CalcInterrupt)，因此可得到下图所示的最粗粒度的计价流程

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgFf1c6WEPYOShrGoqyAnzbwQUV1IUSP7pjg1vW5vNCiad0icUulEj2dPQ/640?wx_fmt=png)

那这三个级别的计算优惠价内部又是如何实现的呢？经过业务抽象，这三个级别的计算可以变成一个通用的计算优惠逻辑，仅有优惠级别的区分。

### **3.3.2 通用流程**

经过业务抽象发现三个级别的优惠计算的通用逻辑：

* 获取当前层级的优惠查询器
  
    （Get Current Level PromotionGetter）

* 过滤优惠查询器
  
    （Filter PromotionGetter）

* 查询优惠（Get Promotion）

* 过滤优惠（Filter Promotion）

* 通过计价引擎计算优惠（Calc Engine）

* 过滤计价结果（Filter CalcResult）

因此我们得出如下的通用的计价流程：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJg81QbNGg5ziccAdhXLz2sDOPRFFUwSf2h28MAuGhlL0wTlaaibM3K15cw/640?wx_fmt=png)

通用计价流程中的又有几个相对灵活的与业务相关过滤逻辑，从后面的细节流程中可以了解更多的实现。

### **3.3.3 细节流程**

之所以在通用计价流程中会有几个过滤节点，是因为在业务上会有一些特殊的过滤逻辑，比如商详页来源的时候，只能使用商品级优惠查询器，某个优惠只能特殊渠道去享受等等。

所以需要抽象出一个通用的可扩展的过滤机制来实现业务需求，因此会按照不同维度去定制一些链式过滤器，执行流程如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgmBtibPsYiadXRPI9yZ9P6IrdBO9ic28OpaVQIBnbj95oiaYGQHVHjpqTicg/640?wx_fmt=png)

当然图中所示的不同维度额过滤器只是目前业务中的一部分，比如还有按照终端、付款方式、外部业务方等等，这些在具体实现的时候可以非常灵活的支持。

**那上述过滤器是如何制定？以及与业务如何关联的？**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJgIJbfna4SfUV5HOdHfwU1nPd238eK6PMC80QCUWq7W9GL00w23mcLWQ/640?wx_fmt=png)

上图中列出部分业务定制过滤序器，自定义过滤器后会自动注册到统一的优惠业务过滤器工厂中，在前述的计价流程中，需要用到相关过滤器时，只需带上相关上下文参数可以自动从过滤器工厂中获取匹配的过滤器。

### **3.3.4 完整全流程**

把前面这一系列流程中进行一个组合拼装，就可以得到计价的完整全流程图，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJg2VfchdmQGUaSxU7gickgBweA0O4MEkM3tcpZ99xbGjTDLx4ibkJQPouQ/640?wx_fmt=png)

从这个完整流程图中，可以看到一个通用稳定的核心计价流程以及一个支持业务多变的定制过滤器，既保证了核心的稳定，又保留灵活的扩展。

# 四、系统核心设计

在通用的计价执行流程中一个节点是「Calc Engine」，也就是计价引擎，这是整个计价逻辑中最核心底层的能力，由它来判定每个优惠是否能被用户享有。

## 4.1 统一优惠模型

由于计价中心在建设的时候，已经存在了促销系统中的各个优惠活动、独立的优惠券及代金券、遗留在商城主站的未迁移的优惠，因此想用兼容这么多的优惠类型，必然需要建立一个统一的优惠模型，而在建设过程中需将现有的优惠模型进行适配转换至统一模型。

统一优惠模型中的一些关键信息有：优惠标识、优惠类型、优惠模板 id、开始结束时间、优惠参数及一些扩展参数等。

## 4.2 优惠模板

1）在进行促销计价时，每个具体的优惠都会对应一个唯一的优惠模板，每个优惠模板本质上是一个 JSON 字符串，只是这些 JSON 字符串是由遵循了一定特殊逻辑规则的元信息数据转化而成，而这些元信息在被计价引擎解释执行时，都是返回布尔类型标识是否通过。

2）基本的元信息数据有这几种：

**AndMeta(与）**

对应逻辑关系中的 “与” 关系，表示该类型的元信息所包含的子元信息解释执行都返回真才为真；

**OrMeta(或）**  

对应逻辑关系中的 “或 “关系，表示该类型的元信息所包含的子元信息任一解释执行返回真就为真；

**NotMeta(非）**  

对应逻辑关系中的 “非” 关系，表示该类型中元信息所包含的子元信息解释为假当前元信息为真；

**ConditionalMeta(条件）**  

如果条件参数不存在或者从上下文获取参数指定的布尔值不为 true，则当前元信息返回真，否则根据元信息中包含的子元信息解释执行的结果作为当前元信息执行结果；

**ComplexMeta(组合元信息）**  

该元信息作为所有模板的通用载体，该元信息中包含两个重要信息 conditon、action，两者的关系是只有 condition 条件都满足后后，才会去执行后续的 action，而 condition 和 action 都可能为前述中的各种元信息的复杂组合。

3）模板元信息关系：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt5lpu5LFYmJBzVYJDCszDJghflictPgDFBGAd8G71kGB6kiclviakedSfh115F8mD3He5oEqIduRYavA/640?wx_fmt=png)

4）优惠模板示例：

```
{
  "type": "COMPLEX",
  "condition": {
    "type": "AND",
    "metas": [
      {
        "type": "CONDITIONAL",
        "metas": [
          {
            "type": "CONDITION",
            "metaCode": "terminalCheckCondition"
          }
        ],
        "param": "needTerminalCheck"
      },
      {
        "type": "CONDITION",
        "metaCode": "amountOverCondition"
      }
    ]
  },
  "action": {
    "type": "AND",
    "metas": [
      {
        "type": "ACTION",
        "metaCode": "cutPriceAction"
      },
      {
        "type": "ACTION",
        "metaCode": "freezeCouponAction"
      }
    ]
  }
}
```

（滑动查看）

### 4.3 计价引擎

计价引擎本质上就是对应优惠模板的解释执行，并配合相关上下文，进行优惠计算，关键代码如下：

```
private boolean executeMeta(Meta meta, EngineContext context) {
    if (meta instanceof AndMeta) {
        return executeAndMeta((AndMeta)meta, context);
    } else if (meta instanceof OrMeta) {
        return executeOrMeta((OrMeta) meta, context);
    } else if (meta instanceof NotMeta) {
        return executeNotMeta((NotMeta)meta, context);
    } else if (meta instanceof ComplexMeta) {
        return executeComplexMeta((ComplexMeta)meta, context);
    } else if (meta instanceof ConditionalMeta) {
        return executeConditionalMeta((ConditionalMeta)meta, context);
    } else {
        return executeIMeta(meta, context);
    }
}

......

private boolean executeComplexMeta(ComplexMeta complexMeta, EngineContext context) {
    Meta condition = complexMeta.getCondition();
    Meta action = complexMeta.getAction();
    return executeMeta(condition, context) && executeMeta(action, context);
}

private boolean executeConditionalMeta(ConditionalMeta conditionalMeta, EngineContext context) {
    PromotionContext promotionContext = context.getPromotionContext();
    if (promotionContext == null || promotionContext.getParameters() == null) {
        return true;
    }

    String conditionParam = conditionalMeta.getParameter();
    String sNeedProcess = promotionContext.getParameters().get(conditionParam);
    if (sNeedProcess == null) {
        return true;
    }

    boolean needProcess = Boolean.parseBoolean(sNeedProcess);
    if (needProcess) {
        return executeMeta(conditionalMeta.getMetas().get(0), context);
    } else {
        return true;
    }
}

private boolean executeIMeta(Meta meta, EngineContext context) {
    IMeta iMeta = MetaFactory.get(meta.getMetaDef().getMetaCode());
    if (iMeta == null) {
        throw new CalcException("meta not found, metaCode=" + meta.getMetaDef().getMetaCode());
    }

    return iMeta.execute(context);
}
```

（滑动查看）  

# 五、小结

通过前面几章内容的描述，我们基本把 vivo 商城促销系统建设计价中心的关键思路阐述完了。建设完计价中心后，整个促销系统的核心基础才立住，但这也只是个开始，整个商城围绕着促销计价中心仍然还有其他待建设的内容，比如整个商城的营销价格能力矩阵，价格监控，商城时光机等等，而这些内容我们后续有机会也会陆续输出相关文章，与大家一起交流学习。

END

猜你喜欢

* [vivo 商城促销系统架构设计与实践 - 概览篇](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491367&idx=1&sn=2d652011d0405b24775fa3f0b75a15ac&chksm=ebd86fb5dcafe6a3884271600da673d5feb3ae9ef8e19565f8b7b30758fc04b769a20cfb20e7&scene=21#wechat_redirect)

* [](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247488706&idx=2&sn=9a2d3358cbf3acf0e06a7bd5626f2fd6&chksm=ebd86450dcafed468207470b8baca16f94f2e08f16c19132df562c7a357feffb009fdf7327cc&scene=21#wechat_redirect)[vivo 商城架构升级 - SSR 实战篇](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247488706&idx=2&sn=9a2d3358cbf3acf0e06a7bd5626f2fd6&chksm=ebd86450dcafed468207470b8baca16f94f2e08f16c19132df562c7a357feffb009fdf7327cc&scene=21#wechat_redirect)

* [十亿级流量下，我与 Redis 时延小突刺的战斗史](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491344&idx=1&sn=5d573319d989b908454e26711c10e817&chksm=ebd86f82dcafe694446825a09eb4c419369ee14285c7a0a5a72b80236d0f12d6dce07f51d348&scene=21#wechat_redirect)
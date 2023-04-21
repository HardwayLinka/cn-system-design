[原文]([https://mp.weixin.qq.com/s/vD9yvYNaxTQBLABik6aqNg](https://mp.weixin.qq.com/s/vD9yvYNaxTQBLABik6aqNg))

# 【背景】

一年前 vivo 商城还是以 Java 为技术核心，前后台一起，Java 既要负责服务、数据库，也要负责页面的渲染。在早期这种开发模式也能够很好的运行。然而随着业务迭代的加快，前端技术的发展，这种开发模式的弊端越来越明显。主要突出的有以下两个方面：

1. **前端技术栈架构繁杂且陈旧，导致迭代速度很难提升**
   
   到 2018 年 12 月，整个商城前端系统随着不同需求叠加积累的原因，造成了不同页面使用不同的技术，比较典型的有 jQuery，Vue，FreeMarker，artTemplate，这些不同的技术栈从开发来看，相同的内容，在不同的页面可能使用不同的技术栈，导致需要开发很多遍，对开发、测试来说工作量都是成倍的增长。

2. **无法适用多端开发的需求**
   
   自 2017 年微信上线小程序功能后，各种小程序如雨后春笋般出现，vivo 商城一开始也推出了自己的微信小程序，然而由于业务发展，需要适配的端越来越多，原先使用原生开发的小程序方式，无法做到一套代码编到多个平台。

为了提升开发效率，满足高速发展的业务需求，在过去的一年里，我们通过对商城内外部系统的全面分析，按照分层的逻辑整理出前端架构的升级指导说明。

# 【分层架构】

在《前端架构 - 从入门到微前端》一书中提到，前端架构自上而下可以设计为四个层次，分别为系统级、应用级、模块级、代码级，我们通过这四个层次来分析 vivo 商城前端架构升级过程中的种种思考和实践，最终形成了一套以 Vue + Node.js 为核心的全端架构方案。

**技术演进过程：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuYey9h8NmRoRtjiajwicZGoah4T257qSYAia5tMWrpd7iaVyb1Sqj2rSEcw/640?wx_fmt=png)

**分层架构实施图：**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uugTHT9L6XomJzicUIHEShOgJNdwdLf4VibF9VG170NBJRKlRMjFuVoWhA/640?wx_fmt=jpeg)

## 一、系统级

即应用在整个系统内的关系，比如如何和后台通讯，如何与其他应用集成。针对这一级别，我们进行了前后端分离、多端统一、BFF、SSR 等方面的探索和实践。

### **1、前后端分离**

架构升级，第一步面临的问题便是前后端分离，vivo 商城仍然处于业务高速发展时期，不能因为技术重构而停下业务版本的迭代, 因此业务版本迭代必须要和前后端分离同时进行，那怎么才能做到双线并行，平滑升级？

这里举个小例子：当我们分离完成订单模块后，就会通过 Nginx 将关于订单模块的所有请求转发到新的静态资源服务上，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt5ZdEoUwS6VLaS04rqFVEzqt4WjkIqdqMicaQREFZzcCCyNarD38QLL9zU0TVTPFKllw2RgQWH5IQQ/640?wx_fmt=jpeg)

通过前后端分离，我们彻底解放前端，让前端开发效率提升了至少 2 个档次。

更多技术细节比如：新老页面如何同步信息，如何容灾容错等等，请关注我们的系列第二篇《vivo 商城前端架构升级 (二)：前后端分离篇》

### **2、多端统一**

从 PC 浏览器，到移动端浏览器、到 App 内嵌，再到各个小程序，再到服务端、客户端。繁荣的生态，犹如百家争鸣，百花齐放。然而，繁荣的背后，对前端工程师的挑战，则与日俱增。

我们承接的端越来越多，新的端不断的出现，如小程序、快应用等。前端工程师陷入了如下套娃陷阱：

1. 现有代码、新代码要适配新的端开发场景

2. 已经适配新的端开发场景的代码成为了现有代码

3. 现有代码、新代码要适配新的端开发场景

4. 循环...

我们希望解决这种问题，希望做到一套技术架构方案【代码】，可以覆盖现在的端和未来的端。

通俗点说，我们希望做到如下图所示的能力：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uutN5cfkiaYu7G5ZzPTVPQliahiakcwRzdML0jic0ptkMyz13HyOahI8icf9w/640?wx_fmt=jpeg)

在这种前端开发背景下，多端统一出现了。它是前端的一个未来趋势，它也是解决上面套娃陷阱的一剂良药。

更多细节内容，请关注我们的系列第三篇《vivo 商城前端架构升级 (三)：多端实践篇》

### **3、BFF**

**业务现状**

随着端的增多，新的接口数量呈现爆发式增长，老的接口为了适配各端，也会增加了各种不同的字段，导致前后端适配多端的工作量越来越大。但是抽象来看，大部分端的变动都是 UI 层级的变动，很少有底层服务的改变，所以带来了一个问题接口究竟是面向 UI，还是面向通用服务？

为了解决这个问题，Sam Newman 发表了一篇文章，讲述了这种体验者专用 API 的方式，并将其称为 BFF（ [Backends for Frontends](https://samnewman.io/patterns/architectural/bff/) ）模式。

在 BFF 理念中，最重要的一点是：服务自治，谁使用谁开发。服务自治减少了沟通成本，带来了灵活和高效。

**关键技术**

商城前端积极适应前端技术的发展，为了提供一流的用户体验，积极推动 BFF 层在商城业务中的实现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuBgvg4EEb59IZwa2q49ib66ibmeDsFdVybafz8JTXsDxAA5KSov73bnDw/640?wx_fmt=png)

**直连 Dubbo：**

Apache Dubbo 是一款高性能、轻量级的开源 Java RPC 框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

我们使用了社区提供的 Dubbo2.js 进行 Dubbo 服务的调用，并且做了一些封装来优化发开体验。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuhGuLbdIv7wGVNzIPknySpHp32POzIjT4x2I4iazNKhzWvgfdtkCIQicw/640?wx_fmt=jpeg)

**集成 GraphQL 网关：**

GraphQL 是一个开源的 API 数据查询和操作语言及实现为了实现上述操作的相应运行环境。相较于 REST 以及其他 web service 架构提供了一种高效、强大和灵活的开发 web APIs 的方式。

（1）请求你所要的数据 不多不少

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuUlIChhB0ibfy5JSSwCanIpLK13BmFuj71ud1M4neymKzrdM85HPO8ew/640?wx_fmt=gif)

（2）获取多个资源 只用一个请求

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuZk6DEIAGW9dBLH2hsB0xJrKrFVbmzwqjVAHxYtTvJqd1WONHLXReOg/640?wx_fmt=png)

（3）强大的 API 调试工具

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uukbsvDsYXJCKtibp12aaH4OKrfJ1q0e7bPNB3ba6PwLHSSsJ7ZskKq7Q/640?wx_fmt=png)

### **4、SSR**

自从前后端分离后，前端采用了 SPA 技术，走的都是 CSR（客户端渲染）的模式。使用 CSR 的优势在于节省后端资源、局部刷新等，但随着应用的日益复杂, 首屏渲染时间不断变长, 并且存在严重的 SEO 问题。所以为了解决 SPA 应用遇到的这些问题, 我们必须考虑 SSR。

SSR 即服务端渲染，是指由服务器端完成页面的 HTML 结构拼接，并且直接将拼接好的 HTML 发送到浏览器，然后为其绑定状态与事件，成为完全可交互页面的处理技术。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uu3Dm92wX5uVWXA4vetvlPmibyVke7Uqic431k8uYZbIH4ZCrCk8icm2Jxg/640?wx_fmt=jpeg)

主要优势在于：

1. 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。

2. 更快的内容到达时间 (time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。

为了避免重复造轮子，我们使用了社区非常优秀的 SSR 框架 nuxt，通过一系列的优化，比如：页面缓存、组件缓存、API 缓存、最小化渲染等方式，最终让我们页面在 500ms 内就能全部展示，这是用户体验上的极大提升。

## 二、应用级

即应用外部的整体架构，如多个应用之间如何共享组件、如何通信、如何开发通用脚手架等。在应用级别的架构上面，我们主要沉淀了适用于商城的 UI 库，为其他商城衍生项目提供基础组件支持。

### **1、组件库**

移动端的设计千变万化，市场上非常流行的移动端的组件库比如 antd-mobile ， vant ，他们对于开发通用型的 App，非常高效且美观，然而大部分自主研发的 App 都有自己的一套设计风格和理论。如果使用流行的组件库，就会出现频繁需要修改源码，以适应 UI 风格的变化。这样的工作量日积月累，就会变得越来越大。所以我们还是建议如果是做自己特色的 App，还是要自建 UI 库。如果感觉自建 UI 库难度较大，可以先 fork 一份流行的组件库，学习其中的各种实现，慢慢形成自己的 UI 库。

商城也实现了自己的 UI 库，目前已经在商城、秒杀、vivo 内购、v 客联盟等应用中广泛使用，极大的提高了开发效率。如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuzQFFxMsVofktOWflSRK6xlzWw7IXWpDW7kN4DlKmydoZFTFKbNBccQ/640?wx_fmt=png)

## 三、模块级

应用内部的模块架构，如代码的模块化、数据和状态的管理等，在项目中比较典型的是我们设计了针对 Vue 的极致模块化方案，顶层 page 设计，数据自治等方面的工作。

### **1、极致模块化**

我们的方案摈弃了官方推荐的按文件类型组织模块，而采用按照功能组织模块，这种 " 可插拔式 " 组件设计，让代码按照功能聚合。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uu1SMM2x1B6OydmSLba8eibtjAbUtmKtHH3T2oU75zLakPnSIjBxwjO5Q/640?wx_fmt=png)

一个文件包里面包含该功能的所有实现，包括图片、样式、脚本、数据流、组件等等，这样另一个项目想要使用，直接迁移即可，极大地减少了迁移的成本，并且还有一个优势是，如果这个功能以后下架，直接删除即可，不必各个文件夹下找文件，极大地提升了代码的简洁性和可维护性。

```
➜  confirm git:(dev_abtest_gray) tree 
.
├── api.js     // 接口资源
├── components // 内部组件
│   ├── component1
│   │   ├── images
│   │   ├── index.scss
│   │   └── index.vue
│   ├── component2
├── images   // 图片资源
├── index.scss // 样式资源
├── index.vue  // 逻辑实现
├── module.js  // 数据流
└── trackData.js // 埋点
```

### **2、顶层 page 设计**

所有的页面都会有很多通用的功能，比如加载前的骨架图、出错后的处理逻辑、数据采集逻辑、上拉刷新、下滑分页加载，全局 iOS 适配等等，这些功能在逻辑上是需要抽象的，避免各个页面多次实现，导致浪费开发资源和降低程序的可维护性。

针对此我们实现了一个顶级 page 组件，所有的页面都继承于这个 page 。做到通用性的功能全局管理。

顶级 page 的部分 API 使用如下：

```
<template>
  <v-page
    // 页面是否完成
    :pageInit="true"
    // 页面是否出错
    :pageError="false"
    // 页面监控开启，包括pv,uv,渲染时间
    :pageMonitor="true"
    // 上拉刷新
    @pullDownRefresh="pullDownRefresh"
    // 下拉监听
    @reachBottom="reachBottom"
    // 滚动监听
    @pageScroll="pageScroll"
  >
    <template slot="header">
      <!-- 自定义头部，如果没有则使用默认顶部导航菜单 -->
      <Header />
    </template>
    <template slot="skeleton">
      <!-- 自己实现的骨架图，如果没有则使用默认骨架图-->
      <Skeleton />
    </template>
    <template slot="footer">
      <!-- 自己实现的底部，吸底显示，如果没有则不显示-->
      <Footer />
    </template>
  </v-page>
</template>
```

（滑动可查看）  

### **3、数据自治**

本着谁使用，谁负责的原则，对于页面中的数据流也是一样的，我们开发了针对 page 的全局 mixin，负责自动注册和卸载页面数据，并将各个页面之间的数据进行隔离。

```
import { mapState, mapGetters, mapActions, mapMutations } from 'vuex'

export default (name, module) => ({
  computed: {
    ...mapState(name, Object.keys(module.state)),
    ...mapGetters(name, Object.keys(module.getters))
  },
  created () {
    // todo 要加判断是否已经注册，动态注册模块
    if (!(name in this.$store._modules.root._children)) {
      this.$store.registerModule(name, module)
    }
  },
  methods: {
    ...mapActions(name, Object.keys(module.actions)),
    ...mapMutations(name, Object.keys(module.mutations))
  }
})
```

（滑动可查看）

## 四、代码级

当我们开始编写代码的时候，就要考虑代码的规范和质量。规范的目的是为了提升维护性，而质量则是开发的门面，一个好的项目离不开这这两个内容的约束。

### **1、规范**

一个好的规范应该做到简单、好记、易于执行。为了实现这个目标，我们制定了一系列的规范，最主要的是开发规范、提交规范。

每个项目组的规范可能都不一样，需要根据自己的项目特色，可以参考优秀的项目实践，整理出自己的项目规范，小组内部讨论优化，达成一致意见，最后发布执行。每一位新进项目组的成员，首先要做的就是学习这些规范，用规范引导开发。

**（1）开发规范**

包含但不局限于以下内容：命名规范、HTML 规范、css 规范、js 规范。

**（2）提交规范：**

为了规范提交代码，从而方便开发者追踪项目的开发信息和功能特性，我们封装了 @vivo/commit，对我们的提交进行强制校验。比如：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuKIibFmYXXC7HwX92vtt6t6qDywOGBcIiaRXQV9OMGYCoXb3gbLkVHTUw/640?wx_fmt=png)

每一条 commit 由四个部分组成，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuarwiagfM22nKK99d6PzS6RfFFuWKTl5wXjF5bcs6DLrAuwyvksiciaZrQ/640?wx_fmt=jpeg)

* **修改类型**

style: 样式修改  
fix: bug 修复  
feat: 功能开发  
refactor: 代码重构  
test: 测试类修改  
doc: 文档更新  
conf: 配置修改  
merge: 代码合并  

* **影响模块**
  
  每一条 commit，应明确指出其影响范围是哪个模块，如果是通用模块，注释上（全局）字样，方便 code reviewer 对方案进行评估  

* **跟踪单号**
  
  每一条 commit，必须要有单号，每个公司都有自己的缺陷跟踪系统，单号的目的是为了让每一条提交有据可循，方便后续对问题的回溯。  

* **问题描述**
  
  问题描述应该简洁明了，让其他人一看就知道这条 commit 修改了什么，禁用一些通用描述，比如：' 修改了一个 bug'，' 添加了一个功能 '  

### **2、质量**

关于质量我们从两个方面进行提升，代码检视 和 代码覆盖率。

**（1）代码检视：**

为了提高代码检视的效率，调研了市场上面众多的代码检视工具，好用都需要收费，并且功能比较复杂，比如：upsource。于是开发了一个基础 vscode 的 code review 插件，支持 GitLab，实时消息通知。

添加评论

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuomylnvfH3TaUOMN017jAlyaNG9r2Lukzciazr2UeQoxvI6sAcszFxpA/640?wx_fmt=png)

**（2）代码覆盖率：**

商城的业务迭代速度非常快，使得开发单元测试开发的成本非常大，然而我们有时又想看看测试场景的覆盖情况，为了实现这些目标，我们研发了集成测试代码覆盖率平台。通过这个平台可以清晰的看到每一行代码被测试执行的情况。保证了开发的质量，并能给测试提供精确的指导建议。

* 服务端架构

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuMqDBWRj7uLcHxyzISxN5J72ibGxVg8z0DAAP1EV9oJ5jicKItuiaJUICg/640?wx_fmt=jpeg)

* 前台架构

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uurdvlic42glgCmJUU41L2vRPUZTA72jtKp5ubKQdVzZZtOzzgoa5TDcA/640?wx_fmt=jpeg)

* 自动集成 GitLab

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuI08txeJTlEXwUEyWWhkXzgVeQyZ7JSbvc3vsXLtAbM8g1oan7ORolA/640?wx_fmt=png)

* 结果展示  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uunbsItBzqGaRR6xmqV5Pk4Yic9kKbtr8uUxx1J1njTtz7YodAn7WepDA/640?wx_fmt=png)

* 结果展示

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6I1hGYwkQN1dvx92Ad39uuOMA1nGJQb2BoOE6ZvkQkMsXKo3GUdAoU4ry9DPDHJH2iadKXTNSkLAQ/640?wx_fmt=png)

# 【小结】

本篇文章介绍了 vivo 商城架构升级的背景，并从系统级、应用级、模块级、代码级 四个层次，总结了 vivo 商城前端架构升级过程中的种种实践和探索，希望能给有类似需求的团队带来帮助。

我们在前端技术方面的探索并未结束，作为前端架构升级的第一篇，后面会围绕架构升级带来一系列的文章，为大家更详细的讲解其中的难点和经验，敬请期待。

END

猜你喜欢

* [Puppeteer 入门与实战](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247487500&idx=2&sn=e51245a758f3de1baf436fdfefd851f8&chksm=ebd8609edcafe9882bab6d682e648154c0ba4c89ddc576bbb52c1962a80ad924cb38654a5e46&scene=21#wechat_redirect)

* [聊聊微前端的原理和实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247487315&idx=2&sn=32bcf0d8a7215da88857c68e09da967f&chksm=ebd87fc1dcaff6d78cfda49168520c73a79ed4cede16cf19ef78893cc73ca9b503143c9e39ee&scene=21#wechat_redirect)

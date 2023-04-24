

作者：vivo 官网商城开发团队 - Sun Yanwu

# 一、背景

随着经济全球化的深入，许多中国品牌纷纷开始在海外市场开疆扩土。实现全球化意味着你的产品或者应用需要能够在全球各地的语言环境使用，我们在进行海外业务的推进时，需要面对的最大挑战就是多语言问题。实现好多语言系统的本地化，更方便快捷的修改多语言文案能让你的产品在各个国家地区里有更强的产品竞争力和更好的用户体验以及更低的维护成本。以此为目标，在 vivo 外销项目的发展过程中我们经过多次迭代，最终结合公司中间件的能力，实现了一套完整的多语言解决方案。

# 二、多语言文案系统的优势

## 2.1 传统的多语言方案基于 Spring

i18n 方案，虽然能够解决基本的多语言问题，但是每次多语言的更新维护都需要发版，项目维护成本非常高。vivo 多语言文案系统的诞生就很好的解决了这个问题，系统结合公司实际情况，基于公司已有的平台—配置中心，最小化接入成本，优化了项目流程，标准化了文案需求的提出，翻译，测试发布等流程，集中式系统化的管理多语言文案极大的提高了发布效率和文案质量。支持传统 web 项目和前后端分离项目，因此可以作为基础能力支撑许多系统的本地化方案。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iak7UBK4VprUlwUnNCTJoABz5zCSh8W2rbRAmBsTUWuETVo3XSxtic66Ig/640?wx_fmt=png)

（ 图 2-1 多语言文案系统和传统多语言方案对比）

# 三、多语言文案系统的整体设计方案

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakeiauDFbY4y6UjhNdjjEpUgwbyMGcLlPph4vIx5k5j91xX1nictzUwVEg/640?wx_fmt=png)

（图 3-1：系统整体设计方案）

多语言文案系统的解决方案整体可以分为 3 个部分：

**1）MCMS 管理系统**  

提供多语言项目的创建、语种的创建，各个语种多语言文案的新增，修改，删除和同步等功能，为多语言文案提供了系统化的管理平台。

**2）配置中心**

结合公司实际情况，基于公司中间件—配置中心（vivo cfg），使得接入成本大大降低。配置中心可以看作是 MCMS 多语言文案管理系统和业务系统的交互桥梁，在 MCMS 系统按照一定的命名规则对多语言文案进行配置后，会将文案同步至配置中心，供后续业务系统根据自身的项目和语种信息进行多语言文案的获取。

**3）业务系统**

通过引入多语言文案系统（MCMS）提供的 jar 包，与配置中心进行交互。接入的业务系统，在需要使用到多语言的地方，只需用和 MCMS 系统相同的命名规则进行多语言 key 的占位，在项目运行时根据当前环境的语言码即以从配置中心拉取到对应项目 - 语种的多语言文案并且替换项目中的多语言 key，从而实现语言的本地化。

# 四、多语言的具体处理流程

业务系统接入多语言系统后，进行多语言适配的整体架构如下图所示，流程如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iak5tRAzAiciahr1qAqksaX3Dl4btVvnoc3EeWl8CFOiaHia5hZ8W3xibqd1iag/640?wx_fmt=png)

（图 4-1：系统整体架构）

1）用户请求业务系统，携带的信息包括：

* 界面可供选择的国家 / 语言按钮；

* 路径中携带的国家信息；

* cookie 中携带的国家信息。

2）服务器启动后进行信息初始化：

* 配置当前的默认国家 / 语种 / 时区；

* 配置 mcms 系统所需的默认语言。

3）根据用户请求时携带的信息根据以下流程图逻辑判断出最终反馈给用户的国家和语言信息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iak6N4roNDc9vfzXpBnwHiaGW5jkMONvx13TtzL5XP2HO8auHzjNvK0r6A/640?wx_fmt=png)

（图 4-2：多语言处理流程）

4）根据 requestCountry,requestLocale 设置系统 locale，系统 locale 会影响时区的设置以及多语言文案的后缀。

5）根据系统 locale 从配置中心中获取对应语种文案，这样，在基本的多语言处理中，我们就加入了国家和 locale、时区的对应关系。

通过以上流程一个外销系统在不同的地区，不同的语言环境在系统运行时就可以从配置中心拉取到对应地区的多语言配置，用实际的多语言文案来替代代码中的多语言 key，从而实现在不同语言环境下展示不同的多语言文案。

# 五、多语言文案系统的使用

## 5.1 系统简介

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakt5Hp9ibTzSdEg4PEicRKicKibAmic6hb0CrHiangLN8pEp0xD5TDr9P8CHuA/640?wx_fmt=png)

（图 5-1 MCMS 基本功能简介）

* 项目管理：进行项目的创建，在 MCMS 后台新建一个项目需要在配置中心有一个对应的项目。

* 语言管理：在对应的项目下，进行语言版本的创建。

* 内容管理：在对应的项目下，根据具体的语言版本配置多语言信息。

* 用户管理：进行用户权限的管理，不同用户查看不同的项目。

### **5.1.1 使用流程**

**项目的创建**

1）登录 MCMS 系统，新建项目

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakMgPf9kPibhkK3kCNuDibIO7FKMW9RXhAQoIIqJXdXM4RlcNXsGdUO9Rg/640?wx_fmt=png)

（图 5-2 新建多语言项目）

2）AppKey 和 appSecret 的申请（创建语言版本时使用）

APPKey 和 APPSecret 在配置中心的 “开放接口” 中进行申请。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iak84vzZ2qmUuQajia3AP5D3lC2sib1C6RgRIrh8vZESYzicuRsCInyPbwkQ/640?wx_fmt=png)

（图 5-3：配置中心 - 开放接口）

在配置中心的 “开放接口” 中生成 appKey 和 appSecret。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakGmRibEp62gd1Mo4zMniaaok4MHiaANJtViaDBBibjiaYCIRFGqDYbKm6hW7A/640?wx_fmt=png)

（图 5-4 AppKey 和 appSecret 的申请）

## 5.2 语言版本的创建

在多语言文案系统（mcms）后台为项目新增语言版本，可以单个新增或者批量新增。

**1）单个新增**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakxNz3oDMClFZEwpuBkicantWjaBE3usVibMwpEyTGByC0X0fiaEsF7yJ3Q/640?wx_fmt=png)

（图 5-6 语言版本创建方式）

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakibREHOqYID3hqj7tsHpyJWia70fcgq3HwGpIYyz47J8cQZaFyVph3eWQ/640?wx_fmt=png)

（图 5-7 新建语言版本）

**2）批量新增**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakZmATgtvNeDaMiaNmRwhHcfaVJJTqXrn5nCfIDs3PeU3Zm2mLUibeeNfA/640?wx_fmt=png)

（图 5-8 批量新增语言版本）

# 5.3 内容管理

在创建完项目并在项目中新增了若干个语言版本后，在内容管理中进行具体的语言信息的配置。

可以单条新增 / 编辑 / 审核，也可以批量导入;

1）新增配置项

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakUI8Knp7dMnJmhT9AZLTPnrLZFgqIbVxkWhlm6giafsY6MtXQLiclPA7A/640?wx_fmt=png)

（图 5-9 新增配置项内容）

2）配置项内容的审核

审核通过 / 审核驳回 --> 配置项生效 / 无效

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakibVPLV3x66GalZfA1TaKlHX0Ob6EaSs8qZ679FBtcgpuicqGGroVA4Nw/640?wx_fmt=png)

（图 5-10 新增内容审核）

3）内容管理支持查询、新增、导入 、审核、修改、导出功能， 配置项内容的状态转换如图 5-11 所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakgOjrhswwaZaVoUAwV5OrDOl2NicaAgdE77nNWptEZV5BCgEiblKb5HIw/640?wx_fmt=png)

（图 5-11 配置项内容状态转换图）

4）批量导入多语言文案

获取导入模板

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakWBGkdckDvqghicUWcYEQLMIJJl02IRAUTzjGQbOUtAVwg3dNrBP1Xsg/640?wx_fmt=png)

（ 图 5-12 配置项的批量导入和导出）

填写配置键、配置场景、中文文案、外文翻译

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakic2LMFxWrSFr2MwTn0hrhye6R85RNzOx9QfiaNqDbDDiavD6EYB6fO2CQ/640?wx_fmt=png)

（ 图 5-13 导入文件模板）

导入配置文件生效

## 5.4  用户管理（权限管理）

为项目相关人员申请账号并绑定项目：

* 管理员 -- 最高权限，管理所有项目；

* 普通用户 -- 维护相关项目的多语言内容的权限；

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iaksEF6E2SScdVtPeYpNZ6aN4vrLjEXNLKM0GN8cAxn50W8fibTa0mHwyg/640?wx_fmt=png)

（图 5-14 新建用户和项目权限管理）

# 六、多语言文案系统的接入

## 6.1 引入 jar 包

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakoNKRWXQ86XRARL0yhBlyjdoia6wicSxHtDAAecNgtlGhcqceuDDeT8Mg/640?wx_fmt=png)

（ 图 6-1：引入 jar 包）

## 6.2 配置中心和多语言系统进行关联配置

多语言系统与配置中心是强关联的，可以通过在配置中心配置不同的语言版本，来达到区分不同的语种的目的。

在项目启动时获取到语言的配置信息。

locale.config

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakerFwIBMSDXh3pwpzMCuJgUg8OIzErbZGd8sbfy1J0NcTJpBwGTztTg/640?wx_fmt=png)

env.list.config

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakUibTpnFTSk6Nib4YIx4dp8DQsV1nwBdvmvuw8d8ib8zeG8JBiczERKBzcg/640?wx_fmt=png)

（图 6-2：语种的配置）

```
@Configuration
public class WebConfiguration {

@Bean
public VivoPropertyExtendConfigurer vivoPropertyExtendConfigurer(){
VivoPropertyExtendConfigurer vivoPropertyExtendConfigurer = new VivoPropertyExtendConfigurer();
vivoPropertyExtendConfigurer.setLocations(
new ClassPathResource("config/vivo.properties"));
vivoPropertyExtendConfigurer.setEnvListConfigKey("env.list.config");
return vivoPropertyExtendConfigurer;
}
@Bean
public FilterRegistrationBean mcmsFilterRegistration() {
FilterRegistrationBean registration = new FilterRegistrationBean();
registration.setFilter(new VivoMcmsPropsFilter());
registration.addUrlPatterns("/mcms/props");
registration.setName("vivo-mcms-filter");
registration.setOrder(1);
return registration;
}
}
```

## 6.3 配置中心创建多语言版本

在配置中心创建和多语言系统相对应的语言版本，在多语言系统中配置的多语言文案会同步到对应的配置中心语言版本中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iakBa6QzYxkGa8PJuJUVksaYPFrBx5JR61Ac3iaELZAjV5W374ZwopTjNg/640?wx_fmt=png)

（图 6-3：当前项目的语言版本）

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iak7Y9uHQ2aVhHMicSSqOsloHjgXan7ADcPIDUIgwcrupqJ9yvFPbx5V9g/640?wx_fmt=png)

（图 6-4：从多语言系统同步到的多语言文案）

## 6.4 多语言的切换

在多语言文案系统中创建的多语言都会同步到配置中心，而配置中心与项目是强关联的，在项目启动时，会从配置中拉取若干配置项。在项目创建了语言版本后，项目启动时会根据当前的语言环境拉取配置中心对应的多语言文案，根据其 key-value 的数据结构，对项目中的 key 进行替换，从而实现在不同的语言环境展示不同的语言。

# 七、多语言的切换

多语言文案系统的另一个优点是实现了前台可视化，所见即所得。通过浏览器插件的方式，在前台进行页面调试时，可以实时的对页面当前标签的文案进行多语言的修改，在前端文案展示修改的同时，通过调用 MCMS 系统提供的接口将修改内容实时的更新到数据库和同步到配置中心，从而实现前端页面修改和后台服务端数据的一致性。多语言文案的可视化大大提高了系统的使用体验，提升了接入方的开发效率。

![图片](https://mmbiz.qpic.cn/mmbiz_png/4g5IMGibSxt6RiaCwS23fDz06Or28hp9iaksvXfQGQnxc6JKIILZHqjUqz6WYVyC4haOaWNjP1wScFt9pl2DiaFqpw/640?wx_fmt=png)

（ 图 7-1   多语言前端可视化）

# 八、结语

本文主要介绍了 vivo 外销多语言文案系统的产生背景，方案设计，业务系统的接入，以及可视化等。凭借接入简单，系统化管理，可视化，即时生效等优点已经有越来越多的外销项目接入了多语言文案系统。接下来为了适应快速发展的外销业务，系统也会不断的迭代优化，成为更方便，更快捷，更直观的多语言解决平台。

END

猜你喜欢

* [](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247492304&idx=1&sn=4f2a666cd60ce0297db5c20a763cb4b9&chksm=ebdb9242dcac1b54e234786ee3b2d60a873f304ee5c562410e95fbc4ef21f969c0113c7189e0&scene=21#wechat_redirect)[vivo 全球商城：商品系统架构设计与实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247492304&idx=1&sn=4f2a666cd60ce0297db5c20a763cb4b9&chksm=ebdb9242dcac1b54e234786ee3b2d60a873f304ee5c562410e95fbc4ef21f969c0113c7189e0&scene=21#wechat_redirect)

* [vivo 全球商城 - 营销价格监控方案的探索](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491988&idx=1&sn=d1f28063c64ce619251d3af175a9e8ed&chksm=ebdb9106dcac181074e664617a2f336a307636768f24f5f39d66fed25c33eaf3a3d75c209772&scene=21#wechat_redirect)

* [vivo 全球商城：订单中心架构设计与实践](http://mp.weixin.qq.com/s?__biz=MzI4NjY4MTU5Nw==&mid=2247491631&idx=1&sn=b53c69df13a80c6bf37531e0c3788188&chksm=ebdb90bddcac19ab3dae9eb635089644d6032dfc4394c3f9bcfedf837060eaeb8c5a4d70c532&scene=21#wechat_redirect)

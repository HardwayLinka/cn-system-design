# cn-system-design
Because there are very few articles about system design in Chinese on the Internet, this project was born. If you want to learn system design written in English, you can see the following excellent open source projects (these projects are also reference sources for the current project)：
* https://github.com/shashank88/system_design
* https://github.com/karanpratapsingh/system-design
* https://github.com/alexpate/awesome-design-systems
* https://github.com/checkcheckzz/system-design-interview
* ...

Next, I will start using Chinese to give a brief introduction to the current project.

因为需要着手设计大型的系统，所以我想要依靠在系统设计有许多积累的英文世界，并且将我阅读过的文章翻译成中文。

本项目会以实际案例出发，自顶向下地逐渐学习更细节的知识点。

## 案例
* Uber
  * [设计 uber](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/%E8%AE%BE%E8%AE%A1%20Uber.md)
* Tinder
  * [设计 Tinder](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/%E8%AE%BE%E8%AE%A1%20Tinder.md)
* Instagram
  * [设计 Instagram](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/Instagram/%E8%AE%BE%E8%AE%A1%20Ins.md)
* Twitter
  * [设计 Twitter](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/Twitter/%E8%AE%BE%E8%AE%A1%20twitter.md)
* DropBox
  * //
* Airbnb
  * //
* Quora
  * //
* Azure
  * //
* LinkedIn
  * //
* Netflix
  * //
* AWS
  * //
* Snowflake
  * //
* vivo 互联网技术
  * [vivo 商城前端架构升级-总览篇](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%95%86%E5%9F%8E%E5%89%8D%E7%AB%AF%E6%9E%B6%E6%9E%84%E5%8D%87%E7%BA%A7%20-%20%E6%80%BB%E8%A7%88%E7%AF%87.md)
  * [vivo 商城前端架构升级—前后端分离篇](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%95%86%E5%9F%8E%E5%89%8D%E7%AB%AF%E6%9E%B6%E6%9E%84%E5%8D%87%E7%BA%A7%E2%80%94%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E7%AF%87.md)
  * [vivo 商城前端架构升级—多端统一探索、实践与展望篇](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%95%86%E5%9F%8E%E5%89%8D%E7%AB%AF%E6%9E%B6%E6%9E%84%E5%8D%87%E7%BA%A7%E2%80%94%E5%A4%9A%E7%AB%AF%E7%BB%9F%E4%B8%80%E6%8E%A2%E7%B4%A2%E3%80%81%E5%AE%9E%E8%B7%B5%E4%B8%8E%E5%B1%95%E6%9C%9B%E7%AF%87.md)
  * [vivo 全球商城：架构演进之路](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E6%9E%B6%E6%9E%84%E6%BC%94%E8%BF%9B%E4%B9%8B%E8%B7%AF.md)
  * [vivo 全球商城：从 0 到 1 代销业务的融合之路](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E4%BB%8E%200%20%E5%88%B0%201%20%E4%BB%A3%E9%94%80%E4%B8%9A%E5%8A%A1%E7%9A%84%E8%9E%8D%E5%90%88%E4%B9%8B%E8%B7%AF.md)
  * [vivo 商城架构升级 - SSR 实战篇](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%95%86%E5%9F%8E%E6%9E%B6%E6%9E%84%E5%8D%87%E7%BA%A7%20-%20SSR%20%E5%AE%9E%E6%88%98%E7%AF%87.md)
  * [vivo 全球商城：订单中心架构设计与实践](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E8%AE%A2%E5%8D%95%E4%B8%AD%E5%BF%83%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5.md)
  * [vivo 商城计价中心 - 从容应对复杂场景价格计算](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%95%86%E5%9F%8E%E8%AE%A1%E4%BB%B7%E4%B8%AD%E5%BF%83%20-%20%E4%BB%8E%E5%AE%B9%E5%BA%94%E5%AF%B9%E5%A4%8D%E6%9D%82%E5%9C%BA%E6%99%AF%E4%BB%B7%E6%A0%BC%E8%AE%A1%E7%AE%97.md)
  * [vivo 全球商城时光机 - 大型促销活动保障利器](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%E6%97%B6%E5%85%89%E6%9C%BA%20-%20%E5%A4%A7%E5%9E%8B%E4%BF%83%E9%94%80%E6%B4%BB%E5%8A%A8%E4%BF%9D%E9%9A%9C%E5%88%A9%E5%99%A8.md)
  * [vivo 全球商城 - 营销价格监控方案的探索](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%20-%20%E8%90%A5%E9%94%80%E4%BB%B7%E6%A0%BC%E7%9B%91%E6%8E%A7%E6%96%B9%E6%A1%88%E7%9A%84%E6%8E%A2%E7%B4%A2.md)
  * [vivo 全球商城：商品系统架构设计与实践](https://github.com/HardwayLinka/cn-system-design/edit/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E5%95%86%E5%93%81%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5.md)
  * [vivo 全球商城全球化演进之路—多语言解决方案](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%E5%85%A8%E7%90%83%E5%8C%96%E6%BC%94%E8%BF%9B%E4%B9%8B%E8%B7%AF%E2%80%94%E5%A4%9A%E8%AF%AD%E8%A8%80%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88.md)
  * [vivo 全球商城：电商平台通用取货码设计](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E7%94%B5%E5%95%86%E5%B9%B3%E5%8F%B0%E9%80%9A%E7%94%A8%E5%8F%96%E8%B4%A7%E7%A0%81%E8%AE%BE%E8%AE%A1.md)
  * [vivo 全球商城：库存系统架构设计与实践](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E5%BA%93%E5%AD%98%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E8%B7%B5.md)
  * [vivo 全球商城：电商交易平台设计](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/vivo%20%E5%85%A8%E7%90%83%E5%95%86%E5%9F%8E%EF%BC%9A%E7%94%B5%E5%95%86%E4%BA%A4%E6%98%93%E5%B9%B3%E5%8F%B0%E8%AE%BE%E8%AE%A1.md)
  * [大数据平台架构设计探究](https://github.com/HardwayLinka/cn-system-design/blob/main/resources/vivo/%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E6%8E%A2%E7%A9%B6.md)
* 阿里
  * //
* 腾讯
  * //
* 美团
  * //
* 百度
  * //
## 开源工具
//
## 算法
[resumejob/system-design-algorithms](https://github.com/resumejob/system-design-algorithms)
## 领域知识
//
## 面试
[soulmachine/system-design](https://github.com/soulmachine/system-design)

title: 微服务中的关键数据采集和stream化（待完善）

date: 2018-12-10 9:30:21
tags: 微服务 数据采集 监控 stream

### 微服务中的关键数据采集和stream化

----

### 目的：

- 建立系统级的系统调用数据采集基础设施，便于进行关键业务数据的采集
- 支持实时业务分析需求，进行数据的stream化

### 参考：

 - Spring Cloud Sleuth进阶实战 https://blog.csdn.net/forezp/article/details/76795269
 - SpringBoot非官方教程 | 第二十六篇： sprinboot整合elk，搭建实时日志平台 https://blog.csdn.net/forezp/article/details/71189836

### 依赖的组件

- sleuth
- zipkin
- elasticresearch
- mysql等
- kibana
- 
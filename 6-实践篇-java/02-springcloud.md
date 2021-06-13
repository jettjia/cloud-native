# springcloud 发布到 k8s

## 代码概述

```
项目架构：前后端分离

后端技术栈：SpringBoot + SpringCloude + SpringDataJpa (Spring全家桶)
```

微服务项目结构

```
tensquare_parent
  tensquare_admin_service
  tensquare_common
  tensquare_eureka_server
  tensquare_gathering
  tensquare_zuul
  pom.xml
  
```

* tensquare_parent：父工程，存放基础配置
* tensquare_common：通用工程，存放工具类
* tensquare_eureka_server：SpringCloud的 Eureka注册中心
* tensquare_zuul：SpringCloud的网关服务
* tensquare_admin_service：基础权限认证中心，负责用户认证（jwt）
* tensquare_gathering ：一个简单的业务模块，活动微服务相关逻辑





## idea 创建项目





## 代码发布


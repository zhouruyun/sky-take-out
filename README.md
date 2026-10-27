# 苍穹外卖 · Sky Take-out

一个面向餐饮商户的外卖点餐系统，包含 **管理端（商家后台）** 与 **用户端（微信小程序）** 两条业务线，覆盖「菜品上架 → 用户下单 → 商家接单 → 派送完成 → 数据统计」的完整闭环。

> 本项目基于黑马程序员《苍穹外卖》课程需求实现，用于学习 Spring Boot 企业级开发与求职展示。

![Java](https://img.shields.io/badge/Java-17-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.1.2-6DB33F?logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)
![License](https://img.shields.io/badge/License-Apache%202.0-blue)

## 技术栈

| 分类 | 选型 |
|---|---|
| 后端框架 | Spring Boot 3.1.2、MyBatis、PageHelper |
| 数据库 | MySQL 8.0、Druid 连接池 |
| 缓存 | Redis、Spring Cache |
| 认证 | JWT（管理端 / 用户端双拦截器） |
| 实时通信 | WebSocket（来单提醒、客户催单） |
| 定时任务 | Spring Task |
| 报表 | Apache POI（Excel 导出） |
| 接口文档 | Knife4j（OpenAPI 3） |
| 构建 | Maven 多模块 |

## 功能模块

**管理端**

- 员工管理：登录、新增、分页查询、启用/禁用
- 分类管理：菜品分类、套餐分类的增删改查
- 菜品 / 套餐管理：图片上传、起售停售、口味与规格
- 订单管理：接单、拒单、派送、完成、订单搜索
- 店铺管理：营业状态切换
- 数据统计：营业额、订单量、新增用户、销量 Top10，支持 Excel 导出

**用户端**

- 微信登录、地址簿管理
- 菜品浏览、购物车、下单、支付回调
- 历史订单、再来一单

## 项目结构

```
sky-take-out
├── sky-common      # 公共模块：常量、异常、工具类、返回结果
├── sky-pojo        # 实体模块：Entity / DTO / VO
├── sky-server      # 业务模块：Controller / Service / Mapper / 配置
│   └── src/main/java/com/sky
│       ├── controller   # admin 与 user 两套接口
│       ├── service      # 业务实现
│       ├── mapper       # MyBatis 接口
│       ├── config       # Redis、WebSocket、Knife4j、拦截器注册
│       ├── interceptor  # JWT 鉴权拦截器
│       ├── aspect       # 公共字段自动填充
│       └── task         # 定时任务（超时订单处理）
└── demo/mysql.sql  # 建库建表 + 初始数据
```

## 实现要点

1. **JWT 双端鉴权**：管理端与用户端各自一套拦截器与 token 校验规则，配合 ThreadLocal 保存当前登录人 ID，业务层随取随用。
2. **公共字段自动填充**：自定义 `@AutoFill` 注解 + AOP，统一填充 `create_time`、`update_time`、`create_user`、`update_user`，避免每个 Service 手写。
3. **缓存设计**：菜品、套餐列表使用 Spring Cache + Redis 缓存；新增/修改/起售停售时按分类维度精准清理缓存；对查询为空的结果做短 TTL 缓存，防止缓存穿透。
4. **实时提醒**：管理端通过 WebSocket 建立长连接，新订单进入后主动推送并触发语音提醒；用户催单同样走 WebSocket 实时触达。
5. **订单流转**：订单状态机覆盖待付款、待接单、已接单、派送中、已完成、已取消；Spring Task 定时扫描超时未支付订单，自动取消并回滚库存。
6. **报表导出**：Apache POI 按日聚合营业额、订单量、新增用户，导出 Excel 运营报表。

## 本地运行

**环境要求**：JDK 17、Maven 3.8+、MySQL 8.0、Redis 7

```bash
# 1. 初始化数据库
mysql -uroot -p < demo/mysql.sql

# 2. 修改配置
#    sky-server/src/main/resources/application-dev.yml
#    填写 MySQL、Redis 连接信息，以及微信小程序 / 支付相关配置

# 3. 启动
mvn clean install -DskipTests
mvn -pl sky-server spring-boot:run
```

启动后访问接口文档：`http://localhost:8080/doc.html`

## 后续计划

- [ ] 用 Docker Compose 一键拉起 MySQL + Redis，降低运行成本
- [ ] 补一份 JMeter 压测记录，量化缓存改造前后的接口 RT
- [ ] 尝试接入 LangChain4j，用自然语言完成「查菜单 / 下单」的对话式点餐

## 说明

课程项目，代码在课程资料基础上整理实现，仅用于学习交流。

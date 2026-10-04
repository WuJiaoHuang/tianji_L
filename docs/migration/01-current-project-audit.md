# Phase 1 - Heavenly-Secrets-Academy Current Project Audit

> 审计时间：2026-10-04  
> 审计范围：当前 `dev` 分支原“天机学堂”后端仓库  
> Phase 限制：本阶段仅做基线审计，不做业务改造、不启动 Docker、不升级框架、不创建前端。

## 1. 项目概览

当前仓库是一个 Java 11 + Spring Boot 2.7.2 + Spring Cloud 2021.0.3 + Spring Cloud Alibaba 2021.0.1.0 的多模块 Maven 微服务项目。

根模块：

| 项 | 值 |
| --- | --- |
| groupId | `com.tianji` |
| artifactId | `tjxt` |
| version | `1.0.0` |
| packaging | `pom` |
| 当前分支 | `dev` |
| 远程仓库 | `https://github.com/Kiraaaaaaa/Heavenly-Secrets-Academy.git` |

README 明确说明：`dev` 分支是作者基于 `init` 分支学习开发的分支，完整源代码在未上传的 `master` 或配套资料中。因此本仓库应视为“可参考但未完全闭环”的基线项目。

## 2. Maven Modules

父 POM 声明的一级模块如下：

| 原模块 | 类型 | 说明 |
| --- | --- | --- |
| `tj-common` | jar | 公共工具、异常、MVC advice、Swagger、MQ、Redisson、MyBatis 自动配置 |
| `tj-auth` | pom | 认证授权聚合模块 |
| `tj-api` | jar | OpenFeign client、跨服务 DTO、本地 Caffeine 缓存配置 |
| `tj-gateway` | jar | Spring Cloud Gateway 网关 |
| `tj-user` | jar | 用户、学员、教师、员工 |
| `tj-message` | pom | 消息聚合模块 |
| `tj-media` | jar | 媒资、文件、视频点播 |
| `tj-course` | jar | 课程、分类、目录、草稿、发布 |
| `tj-search` | jar | Elasticsearch 搜索、课程索引、推荐 |
| `tj-learning` | jar | 学习记录、课表、签到、积分、排行榜、问答 |
| `tj-pay` | pom | 支付聚合模块 |
| `tj-trade` | jar | 购物车、订单、退款申请 |
| `tj-exam` | jar | 题库、题目管理 |
| `tj-data` | jar | 数据中心/看板 |
| `tj-remark` | jar | 点赞记录服务 |
| `tj-xxljobDemo` | jar | XXL-JOB demo |
| `tj-promotion` | jar | 优惠券、兑换码、促销 |
| `tj-aopdemo` | jar | 空 demo 模块 |

嵌套模块：

| 聚合模块 | 子模块 |
| --- | --- |
| `tj-auth` | `tj-auth-common`, `tj-auth-service`, `tj-auth-resource-sdk`, `tj-auth-gateway-sdk` |
| `tj-message` | `tj-message-domain`, `tj-message-api`, `tj-message-service` |
| `tj-pay` | `tj-pay-domain`, `tj-pay-api`, `tj-pay-service` |

## 3. 版本基线

| 技术 | 当前版本/来源 | 风险判断 |
| --- | --- | --- |
| Java | POM 编译目标 `11`；本机验证环境 Java `17.0.12` | 后续迁移 Java 17 风险中等；Java 21 需单独 Phase 验证 |
| Maven | 本机 `Apache Maven 3.9.10` | 可用 |
| Spring Boot | `2.7.2` | 升级到 Boot 3 涉及 `javax` -> `jakarta`，高风险 |
| Spring Cloud | `2021.0.3` | 与 Boot 2.7 匹配；升级需整体联动 |
| Spring Cloud Alibaba | `2021.0.1.0` | Nacos/Sentinel 版本较旧 |
| MyBatis-Plus | `3.4.3` | Boot 3/Java 21 前需验证兼容性 |
| MySQL Driver | `8.0.23` | 可升级但非 Phase 1 |
| Knife4j / Swagger | `knife4j-spring-boot-starter 3.0.3`，基于 Springfox | Boot 3 不兼容风险高，后续建议迁移 springdoc-openapi |
| Redisson | `3.13.6` | 可迁移自定义锁思想，版本偏旧 |
| Elasticsearch | `7.12.1`，使用 RestHighLevelClient | RestHighLevelClient 已进入维护/废弃风险 |
| XXL-JOB | `2.3.1` | 可迁移，需后续部署验证 |
| Seata | `1.5.1` 属性存在，多个服务引用 shared-seata 配置 | 是否真实使用不完整，后续需单独审计 |

## 4. 启动类与端口

| 服务 | 启动类 | 端口 | spring.application.name |
| --- | --- | --- | --- |
| Gateway | `com.tianji.gateway.GatewayApplication` | `10010` | `gateway-service` |
| Auth | `com.tianji.auth.AuthApplication` | `8081` | `auth-service` |
| User | `com.tianji.user.UserApplication` | `8082` | `user-service` |
| Search | `com.tianji.search.SearchApplication` | `8083` | `search-service` |
| Media | `com.tianji.media.MediaApplication` | `8084` | `media-service` |
| Message | `com.tianji.message.MessageApplication` | `8085` | `message-service` |
| Course | `com.tianji.course.CourseApplication` | `8086` | `course-service` |
| Pay | `com.tianji.pay.PayApplication` | `8087` | `pay-service` |
| Trade | `com.tianji.trade.TradeApplication` | `8088` | `trade-service` |
| Exam | `com.tianji.exam.ExamApplication` | `8089` | `exam-service` |
| Learning | `com.tianji.learning.LearningApplication` | `8090` | `learning-service` |
| Remark | `com.tianji.remark.RemarkApplication` | `8091` | `remark-service` |
| Promotion | `com.tianji.promotion.PromotionApplication` | `8092` | `promotion-service` |
| Data | `com.tianji.data.DataCenterApplication` | `8093` | `data-service` |
| XXL-JOB Demo | `com.tianji.xxljobdemo.XxlJobDemoApplication` | `8099` | `xxljobdemo-service` |

## 5. 配置文件审计

仓库内未发现 `application.yml`，主要使用 `bootstrap.yml`、`bootstrap-local.yml`、`bootstrap-dev.yml`。

配置特点：

- 每个服务的 `bootstrap.yml` 声明 `server.port`、`spring.application.name`、Nacos config 的 shared configs。
- `bootstrap-local.yml` 与 `bootstrap-dev.yml` 大量硬编码 `192.168.150.101:8848` 作为 Nacos 地址。
- 多数业务配置不在仓库内，而是依赖 Nacos shared 配置：
  - `shared-spring.yaml`
  - `shared-redis.yaml`
  - `shared-mybatis.yaml`
  - `shared-logs.yaml`
  - `shared-feign.yaml`
  - `shared-mq.yaml`
  - `shared-xxljob.yaml`
  - `shared-seata.yaml`
- 仓库内未直接发现 MySQL datasource URL、Redis host、RabbitMQ host 等完整配置，说明数据库和中间件配置大概率在 Nacos 中。
- `tj-search` 的 `bootstrap-local.yml` 与 `bootstrap-dev.yml` 包含 Elasticsearch 配置入口。

迁移风险：

- 当前 Nacos 地址不符合 SkillFlow 目标虚拟机 IP `192.168.10.102`。
- 后续迁移应使用 `${SKILLFLOW_INFRA_HOST:192.168.10.102}`，不能继续硬编码旧 IP。
- Phase 1 不修改配置，仅记录风险。

## 6. SQL 与数据库配置

审计结果：

- 仓库内未发现 `.sql` 文件。
- 未发现可直接初始化 MySQL 的 schema/data 脚本。
- 仓库内有大量 MyBatis Mapper XML，但没有数据库初始化脚本。
- 数据库连接配置未在仓库本地配置文件中出现，依赖 Nacos shared config。

影响：

- 当前项目无法仅凭仓库源码完成数据库初始化。
- Phase 2/后续基础设施建设时必须补齐 `deploy/mysql/init/` 初始化脚本。
- Phase 1 不手工补 SQL。

## 7. 技术组件现状

| 组件 | 当前状态 | 证据/说明 | 可迁移性 |
| --- | --- | --- | --- |
| Redis | 已广泛使用 | 用户验证码、JWT、权限版本、学习进度缓存、签到 Bitmap、排行榜、优惠券库存 | 高 |
| RabbitMQ | 已使用 | `RabbitMqHelper`、`@RabbitListener`、失败消息 exchange/queue、学习/搜索/订单/短信/优惠券消息 | 高 |
| Redisson | 已封装 | `tj-common` 提供 `@Lock`、`LockAspect`、`RedissonConfig` | 高 |
| 自定义锁 | 已存在 | `tj-promotion` 另有 `@MyLock`、`MyLockFactory`、RedisLock 示例 | 高，但需统一 |
| Nacos | 已接入 | 所有主服务使用 Nacos discovery/config | 高 |
| Elasticsearch | 已接入 | `tj-search` 使用 `RestHighLevelClient`、课程搜索、MQ 同步索引 | 高，但客户端需升级 |
| XXL-JOB | 已接入 | 课程结束、支付/退款检查、消息通知、积分榜持久化等 JobHandler | 高 |
| Caffeine | 已使用 | `tj-api` 分类/角色缓存、`tj-message` 短信平台缓存 | 中高 |
| CompletableFuture | 已使用 | MQ helper 异步发送、短信异步、优惠券方案并行计算、学习记录延时任务 | 高 |
| CountDownLatch | 已使用 | 优惠券方案并行计算 | 中高 |
| Redis Bitmap | 已使用 | `SignRecordServiceImpl` 签到 | 高 |
| Redis ZSet | 已使用 | 学习积分排行榜相关逻辑 | 高 |
| Lua | README 宣称有优惠券 Lua 方案，但本次代码搜索未确认到 `.lua` 或 `DefaultRedisScript` 主实现 | 待后续复核 |
| MinIO | 未发现 | 当前媒资偏向腾讯 VOD/COS、阿里 OSS | 需 Phase 8 新增/替换 |

## 8. 原业务模块完整度判断

| 模块 | 完整度 | 判断 |
| --- | --- | --- |
| `tj-common` | 较完整 | 公共异常、结果、工具、自动配置较多，是迁移基础 |
| `tj-api` | 较完整 | Feign client 和 DTO 完整度较高，但 DTO 拼写存在 `leanring` 历史问题 |
| `tj-gateway` | 基本可迁移 | 有 Gateway、Swagger 聚合、鉴权 SDK；需后续适配 SkillFlow 路由 |
| `tj-auth` | 较完整 | 登录、角色、菜单、权限、JWT、资源鉴权 SDK 存在 |
| `tj-user` | 中等 | 用户/学员/教师/员工存在，但缺企业组织、部门、岗位、技能档案 |
| `tj-course` | 较完整 | 课程、分类、目录、草稿、审核/发布相关逻辑较多 |
| `tj-media` | 中等 | 媒资/文件/视频点播存在，但与 SkillFlow 目标 MinIO 不一致 |
| `tj-learning` | 较完整 | 学习记录、课表、计划、签到、积分、排行榜、问答相关较多 |
| `tj-exam` | 不完整 | 主要是题库/题目管理，完整考试流程、答卷、评分未充分体现 |
| `tj-remark` | demo/薄模块 | 点赞记录模块很薄，且学习模块也有互动/点赞逻辑，职责需后续梳理 |
| `tj-promotion` | 技术价值高 | 优惠券、兑换码、并发领取、Redis/MQ/锁/并行计算，业务语义需改成培训名额/邀请码 |
| `tj-trade` | 较完整但非核心 | 购物车、订单、退款申请完整度较高，SkillFlow 主流程不应保留支付购买语义 |
| `tj-pay` | 较完整但非核心 | 支付渠道、支付单、退款单、第三方支付，后续归档到 legacy-design 更合适 |
| `tj-search` | 中等 | 课程搜索和 MQ 索引同步存在，需升级到 SkillFlow 搜索域 |
| `tj-message` | 中等 | 短信、站内信、通知任务、模板、缓存；依赖阿里短信 SDK |
| `tj-data` | 不完整/薄模块 | 看板统计类服务，文件数量少，需重建企业学习数据分析 |
| `tj-xxljobDemo` | demo | 仅 demo，不应作为业务服务迁移 |
| `tj-aopdemo` | 空 demo | 无 Java 文件，不迁移业务 |

## 9. 原模块到 SkillFlow 模块映射

| 原天机学堂模块 | SkillFlow 目标模块 | 迁移策略 |
| --- | --- | --- |
| `tj-common` | `skillflow-common` | 渐进迁移公共异常、响应、日志、Redisson、MQ、MyBatis 配置 |
| `tj-api` | `skillflow-api` | 迁移 Feign client 和跨服务 DTO，按领域拆分，避免 common 膨胀 |
| `tj-gateway` | `skillflow-gateway` | 保留 Gateway 和文档聚合思想，重建企业培训路由 |
| `tj-auth/*` | `skillflow-auth` | 保留 JWT、权限、角色、网关鉴权 SDK，重命名角色语义 |
| `tj-user` | `skillflow-user` | 扩展企业、部门、岗位、员工、技能档案 |
| `tj-course` | `skillflow-course` | 迁移课程、分类、目录、发布审核，业务类型改为企业培训课程 |
| `tj-media` | `skillflow-media` | 由腾讯 VOD/COS/OSS 思路迁移到 MinIO 媒资 |
| `tj-learning` | `skillflow-learning` | 迁移学习进度、课表、学习计划、签到/积分部分按职责拆分 |
| `tj-learning` 问答互动 | `skillflow-interaction` | 问题、回答、评论、点赞、收藏、笔记独立成互动服务 |
| `tj-learning` 积分/签到/排行榜 | `skillflow-growth` | Bitmap、ZSet、积分事件迁移到成长服务 |
| `tj-exam` | `skillflow-exam` | 扩展考试、答卷、评分、认证联动 |
| `tj-promotion` | `skillflow-training` / `skillflow-growth` | 优惠券秒杀改为限量培训名额抢报；兑换码改为培训邀请码/课程兑换码 |
| `tj-search` | `skillflow-search` | 保留 MQ 异步建索引，搜索课程/讲师/标签/技能 |
| `tj-message` | `skillflow-notification` | 迁移站内信、提醒、模板、定时通知 |
| `tj-data` | `skillflow-data` | 重建企业学习数据分析 |
| `tj-trade` | `docs/legacy-design` | 订单/购买语义不进 SkillFlow 主流程，保留技术分析 |
| `tj-pay` | `docs/legacy-design` | 支付/退款不进主流程，保留技术分析 |
| `tj-xxljobDemo` | 无直接业务模块 | 只保留调度示例思想 |
| `tj-aopdemo` | 无 | 空 demo，不迁移 |

## 10. 技术升级风险

1. Boot 2.7 -> Boot 3 高风险：
   - `javax.*` 需要迁移到 `jakarta.*`。
   - Springfox/Knife4j 3.0.3 兼容风险高，建议迁移 springdoc-openapi。
   - MyBatis-Plus、Redisson、Nacos、XXL-JOB、Elasticsearch 客户端需联动验证。

2. Java 11 -> Java 17 中风险：
   - 当前本机 Java 17 可编译前 13 个模块及部分后续依赖解析阶段，未暴露早期语法错误。
   - Lombok 父 POM 指定 `1.18.20`，部分模块解析到 `1.18.24`，后续需统一版本。

3. Java 17 -> Java 21 高风险：
   - 需先完成 Java 17 基线，再单独验证 Java 21。

4. Elasticsearch 7.12.1 风险：
   - `RestHighLevelClient` 后续迁移到新 Java API Client 或 Spring Data Elasticsearch。

5. Nacos 配置强依赖风险：
   - 缺少本地 SQL 和 shared yaml，项目不能仅凭仓库一键运行。

## 11. 构建验证

执行命令：

```bash
mvn compile
```

本机环境：

```text
Apache Maven 3.9.10
Java 17.0.12
Windows 11
```

结果：

```text
FAIL / INTERRUPTED
```

真实情况：

- Maven 成功进入 reactor 构建。
- Reactor 共 29 个构建单元。
- 已成功编译到至少以下模块：
  - `tj-common`
  - `tj-auth-common`
  - `tj-api`
  - `tj-auth-resource-sdk`
  - `tj-auth-service`
  - `tj-auth-gateway-sdk`
  - `tj-gateway`
  - `tj-message-domain`
  - `tj-message-api`
  - `tj-user`
- 构建在 `tj-message-service` 解析 `com.aliyun:alibabacloud-dysmsapi20170525:1.0.1` 的传递依赖时长时间下载/枚举 `com.aliyun:aliyun-gateway-pop`、`core-service`、`darabonba-java-core`、`aliyun-java-core`、`aliyun-http-apache`、`aliyun-java-auth` 多个 beta 版本。
- 为避免无限等待，本次构建由人工中止，未得到完整 reactor PASS。

后续建议：

- Phase 1 不修改依赖版本。
- 后续若需要稳定构建，可单独分析阿里短信 SDK 依赖树，固定传递依赖版本或替换短信 SDK。
- 可先用 `-pl` 针对基础模块构建，但不能代替完整构建验收。

## 12. 运行验证

本阶段未启动服务，原因：

- 当前阶段目标是基线审计。
- 仓库缺少 SQL 初始化脚本。
- 本地配置依赖 Nacos shared yaml。
- VMware Docker 基础设施尚未进入 Phase 2，不能提前启动/改造。

## 13. 日志与可观测性现状

现状：

- 项目使用 SLF4J + Logback。
- 存在统一异常处理 `CommonExceptionAdvice`。
- RabbitMQ listener 会将 `REQUEST_ID_HEADER` 写入 MDC。
- 尚未发现统一 traceId/spanId 体系。
- 存在若干 `System.out.println`、`System.err.println`、`e.printStackTrace()`，部分在 test/demo，部分在 main 代码中。

后续风险：

- SkillFlow 需要按日志规范逐步替换 main 代码中的控制台输出。
- requestId 可保留并增强；traceId/spanId 应在可观测 Phase 接入。

## 14. Phase 1 结论

当前仓库具备较高迁移价值，尤其是：

- Redisson + 自定义注解锁；
- RabbitMQ 可靠消息/失败重投工具；
- Redis Bitmap 签到；
- Redis ZSet 排行榜；
- 学习进度 Redis + 延时队列异步持久化思想；
- 优惠券高并发领取、兑换码、Redis/MQ/锁/并行计算；
- Elasticsearch 课程搜索与 MQ 索引同步；
- XXL-JOB 任务调度；
- Caffeine 本地缓存；
- Gateway + Auth + Feign 微服务结构。

但它不是可直接作为 SkillFlow 运行基线的完整项目：

- `dev` 分支 README 自述未完成；
- 缺 SQL 初始化脚本；
- 运行强依赖外部 Nacos shared 配置；
- 多处配置仍指向旧 IP `192.168.150.101`；
- 企业组织、培训计划、技能认证、企业数据分析等 SkillFlow 核心业务缺失；
- 支付/交易/优惠券仍是 B2C 在线教育语义，需后续分阶段改造或归档。

Phase 1 建议通过后，Phase 2 才能开始 VMware Linux Docker 基础设施设计与配置。

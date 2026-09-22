# 优探乐享生活

一个本地生活探索平台。用户可以在这里查找附近的店铺、看真实的探店笔记、领取并抢购优惠券。平台的特别之处在于把大模型融入了日常使用链路，能用自然语言帮用户找店，也能自动整理店铺口碑、识别笔记风险。

## 这个平台做什么

找店不需要在筛选条件里一个个勾。用户直接说出自己的想法，比如“想吃清淡点的”“附近有没有能坐下喝茶的地方”，系统先理解意图，再结合位置检索附近店铺，最后由模型重新排序并给出每家店的推荐理由。

每家店铺的详情页有一份自动生成的口碑总结。它来自用户发布的探店笔记，系统先把笔记分组提炼，再聚合成一份包含高频亮点和独有特色的总结。笔记有更新时，旧总结会通过指纹校验自动失效。

发布笔记时有一道内容安全检查。发笔记页面会先给用户即时提示，后端保存前还会再强制校验一次，识别广告引流、联系方式、隐私信息、违禁内容和人身攻击等问题。模型不可用时退回本地规则，检查不会中断。

其余的基础功能包括短信验证码登录、店铺分类浏览、附近店铺检索、笔记点赞与滚动分页、用户关注与共同关注、优惠券领取与秒杀、图片上传。

## 为什么分成两个后端服务

平台包含两个独立部署的后端。

主业务服务承载全部业务逻辑，使用 Java 8 与 Spring Boot 2.3，数据访问用 MyBatis-Plus，缓存和附近检索用 Redis，秒杀异步下单用 RabbitMQ，静态页面由 Nginx 托管并反向代理接口。这个服务追求稳定，不直接依赖任何大模型 SDK。

AI 服务单独承载模型能力，使用 Java 17、Spring Boot 3.3 与 Spring AI，通过 HTTP 接口对主服务提供意图解析、摘要、重排、推荐理由和风控六类能力。它可以独立升级、独立替换模型供应商。当前接入的是阿里云 DashScope，密钥只从环境变量读取。

模型超时、返回异常或服务不可用时，AI 服务和主服务两侧都准备了本地规则兜底，用户侧功能不会直接报错。

## 目录长什么样

- dianping-nginx-1.18.0：主业务服务的全部代码、配置和 Nginx
- hmdp-ai-service：独立 AI 服务
- sql：数据库初始化脚本 open-source-full-init.sql
- 项目理解文档.md：更完整的设计思路与实现细节说明

## 本地运行需要什么

JDK 8 和 JDK 17 各一个，Maven 3.9 以上，MySQL 5.7 或 8，Redis 6 以上，RabbitMQ 3，Nginx 1.18。

第一步初始化数据库，执行 sql 目录下的 open-source-full-init.sql。脚本会创建名为 hmdp 的数据库和全部表结构，加入店铺简介字段，并导入基础数据和一批用于 AI 演示的美食店铺与笔记样本。

第二步检查主业务服务的配置文件 dianping-nginx-1.18.0/src/main/resources/application.yaml，按本机情况确认数据库、Redis、RabbitMQ 的地址与账号，AI 服务地址默认指向本机 8090。

第三步配置 AI 服务需要的环境变量。Windows 下设置 DASHSCOPE_API_KEY 和 DASHSCOPE_MODEL，例如模型填 qwen-turbo-flash。需要改端口时设置 HMDP_AI_PORT。

图片上传目录在 SystemConstants.java 中，改成自己机器上前端页面的 imgs 目录，否则上传的图片会落错位置。

Nginx 监听 8080，把 /api 开头的请求转发到 8081 的主业务服务，因此页面统一通过 Nginx 访问。

## 启动顺序与端口

先启动 MySQL、Redis、RabbitMQ，再启动 AI 服务，然后启动主业务服务，最后启动 Nginx。两个后端都可以用 mvn spring-boot:run 启动，Nginx 在其目录下执行 start nginx.exe。

启动完成后，页面入口是 http://127.0.0.1:8080 ，主业务服务在 8081，AI 服务在 8090。

数据库初始化只覆盖 MySQL，Redis 中的数据在业务首次访问时自动回填。如果希望提前准备好附近检索和店铺缓存，可以在 IDE 中运行 DemoDataSeedRunner 完成预热。

## 从哪里开始读代码

业务侧的 AI 入口在 controller 目录的 AiController，核心编排在 service 实现层的 AiServiceImpl，调用 AI 服务的客户端在 ai/client 目录，Redis 的键定义在 utils 目录的 RedisConstants。AI 服务侧从 InternalAiController 进入，提示词与全部兜底逻辑集中在 AiOrchestrationService。

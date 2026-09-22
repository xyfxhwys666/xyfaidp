# hmdp-ai-service

`hmdp-ai-service` 是优探乐享生活平台的独立 Sidecar AI 服务，职责是把大模型能力从主业务服务中拆出来，单独部署、单独演进。

当前提供 6 个内部接口：

1. `POST /internal/ai/summarize/chunk`：店铺笔记分段摘要
2. `POST /internal/ai/summarize/final`：多组分摘要聚合为最终口碑总结
3. `POST /internal/ai/intent/parse`：自然语言推荐意图解析
4. `POST /internal/ai/recommend/reason`：推荐理由生成
5. `POST /internal/ai/recommend/rerank`：候选店铺大模型重排
6. `POST /internal/ai/review/risk-check`：笔记内容质检与风控

## 运行要求

- JDK 17
- Maven 3.9+
- 可用的大模型 API Key

## 配置方式

服务默认从环境变量读取模型配置：

```bash
set DASHSCOPE_API_KEY=你的Key
set DASHSCOPE_MODEL=qwen-turbo-flash
```

可选环境变量：

```bash
set HMDP_AI_PORT=8090
set DASHSCOPE_TEMPERATURE=0.2
set HMDP_AI_LOG_LEVEL=info
```

## 启动

```bash
mvn spring-boot:run
```

默认端口：`8090`

## 说明

- 主业务服务通过 `hmdp.ai.base-url` 调用本服务。
- 当模型超时、不可用或返回格式不合法时，服务会退回本地规则兜底结果，保证主链路可用。
- 仓库根目录的 [README.md](../README.md) 负责整体项目介绍与启动说明。

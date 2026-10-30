# 架构与模块说明

## 技术栈

前端使用 Vue 3、TypeScript 和 Vite；后端使用 FastAPI、LangGraph、DuckDB、SQLGlot 和 MCP Python SDK。聊天、Embedding 和 Rerank 请求由 `backend/app/model_client.py` 发起。

当前数据源为 CSV，DuckDB 在内存连接中将文件注册为视图。Schema 文档和向量保存在本地 JSON，没有外部向量数据库依赖。MCP 使用进程内传输，无需独立端口。

## 请求处理

1. `api/routes.py` 接收请求、认证用户。
2. `services/askdata_service.py` 解析访问范围、隔离会话、组装工作流输入。
3. `preprocessing.py` 结合近期轻量上下文，路由到 `direct_response`、`data_qa` 或 `database_query`。
4. 数据问答分析已有结果；数据库查询进入字段检索。
5. `retrieval/` 完成 BM25、向量召回、RRF、Rerank、Schema 图构建，合并用户确认字段。
6. `querying/` 生成查询计划和 SQL；信息不充分时经 LangGraph 中断等待澄清。
7. `mcp_runtime/` 发现和调用工具；DuckDB 执行层校验 SQL 与访问范围。
8. 结果构建与分析模块整理表格、报告及执行信息，前端渲染。

预处理模型不可用时返回服务不可用提示，不会伪造查询结果。Schema 检索依赖模型服务，当前没有离线语义检索兜底。

## 状态与记忆

| 组件 | 职责 |
| --- | --- |
| `session_context.py` | 会话、任务、选中结果与上下文 |
| `short_term_memory.py` | 短期记忆与摘要 |
| `memory_store.py` | 用户保存的字段和表格 |
| `session_archive.py` | 可选 SQLite 会话归档 |
| LangGraph checkpointer | 工作流中断与恢复 |

令牌与部分任务状态保存在进程内。启用会话归档不等于所有状态具备跨进程恢复能力，多实例部署需要进一步设计共享状态。

## 权限与扩展

`security/access_control.py` 维护角色范围；`security/auth.py` 提供演示认证。SQL 执行层限制单条只读查询、禁止 `SELECT *`、检查引用表权限。工具按访问范围提供。

- 新增 Skill：参考 `app/skills/` 的说明文件、配置与注册器。
- 新增工具：参考 `app/mcp_runtime/tools/`，维护输入输出 Schema 和权限判断。
- 修改业务表：同步检查 CSV、Schema、表关系、角色范围和评测 SQL；仅添加 CSV 不会补齐业务元信息。
- 更换模型：分别确认聊天、Embedding、Rerank 接口兼容性，更换向量模型后重建索引。

接入真实数据前还需审查 SQL 沙箱、模型上下文传输及数据源访问权限，不能从当前演示实现推断生产安全性。

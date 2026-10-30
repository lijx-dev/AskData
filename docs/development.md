# 开发与验证

安装和启动见 [首页](../README.md)。后端命令在 `backend` 目录使用项目虚拟环境执行。

## 本地检查

```bash
python -m unittest discover -s tests -p "test_*.py"
python scripts/validate_short_video_ops.py
python -m evaluation.run_benchmark --mode gold --scope all
```

现有单元测试使用模拟客户端或本地数据。gold 评测不调用线上模型；live 模式需要 API 并产生费用。评测会写入结果文件，提交前检查差异，避免覆盖历史基线。

前端进入 `frontend` 执行 `npm ci` 和 `npm run build`。GitHub Actions 运行现有后端测试、数据校验、前端类型检查与构建，不注入模型密钥、不执行 live 评测。

## 索引与数据

发布包包含 `backend/data/schema_store.json`。索引与 Schema、数据及向量配置有关，后端执行以下命令重建：

```bash
python -m scripts.prepare_schema_index
```

也可通过管理员调用 `POST /api/schema/index/rebuild`。重建会调用 Embedding 并修改索引文件，提交前检查差异。

正常启动无需重新生成数据。需要重建时在 `backend` 执行：

```bash
python scripts/generate_short_video_ops_data.py
python scripts/generate_short_video_ops_schema.py
python scripts/validate_short_video_ops.py
```

生成命令会重写数据或 Schema，请备份个人修改，并检查索引及评测是否需要更新。

## 常见问题

| 现象 | 检查方式 |
| --- | --- |
| PowerShell 拦截 npm.ps1 | 使用 `npm.cmd`，无需修改执行策略 |
| 模型不可用 | 检查密钥、模型名称、网络和服务商权限 |
| 聊天成功但检索失败 | 分别验证 Embedding、Rerank 地址与接口格式 |
| 今天或本月没有数据 | 演示数据截止 2026-08-28，请使用历史日期 |
| 前端不能访问 API | 检查后端 8000 端口与 Vite 开发代理 |
| 登录状态丢失 | 进程内令牌在重启后失效，重新登录 |
| 更换向量模型后异常 | 确认维度并重建索引 |

`npm run preview` 用于预览产物，生产环境需单独配置 `/api` 反向代理，不能依赖开发代理。

## 提交

提交前检查 `git diff` 和 `git status`。保留前端锁文件，不提交 `.env`、虚拟环境、依赖、构建产物、会话数据库或个人记忆。新增业务逻辑应补充验证实际行为的测试。

## 本次仓库整理验证（2026-10-30）

- Windows、Python 3.11、Node.js 24 环境下，现有 17 项后端测试全部通过。
- 数据校验通过：41 张表、722,732 行，主键与关联检查无错误。
- 前端类型检查与生产构建通过，依赖按 `package-lock.json` 安装。
- API 冒烟检查通过：健康检查、登录、未登录访问限制、角色可见 Schema、MCP 工具发现、管理员接口限制。
- 文档内部链接及 JSON / JSONL 解析检查通过。

未使用真实 API 密钥运行端到端模型查询或重新执行 live 评测；历史模型评测与上述本地检查应分别理解。GitHub Actions 的 Linux 检查以实际运行结果为准。

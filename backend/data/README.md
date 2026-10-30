# 演示数据

`databases/short_video_ops/` 包含 41 张由生成脚本构建的模拟业务表，共 722,732 行；清单记录时间范围为 2026-01-01 至 2026-08-28。数字来自 `_database_manifest.json`，可通过校验脚本核验。

| 路径 | 用途 |
| --- | --- |
| `databases/short_video_ops/*.csv` | 增长、投放、内容业务模拟数据 |
| `databases/short_video_ops/_database_manifest.json` | 种子、日期范围、行数和角色范围 |
| `databases/short_video_ops/_schema.json` | 业务字段元信息 |
| `schema_store.json` | 发布包提供的字段文档和向量索引 |
| `saved_memories.json` | 保存的记忆，首次保存时创建，Git 忽略 |
| `session_archive.db` | 可选会话归档，Git 忽略 |

CSV 直接随仓库发布，最大文件约 14 MB，无需 Git LFS。模拟数据不代表真实平台经营情况。数据生成种子为 `20260829`。

从 `backend` 执行 `python scripts/validate_short_video_ops.py` 校验。重建步骤见 [开发说明](../../docs/development.md)。接入真实数据时请同时审查 CSV、Schema 样例、向量索引、记忆和评测结果，不要将真实数据直接提交到公开仓库。

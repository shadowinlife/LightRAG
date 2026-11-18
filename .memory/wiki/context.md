# 开发上下文（本机）

> 只缓存「查一下也不容易知道」的事实；命令与规则的权威来源是根 `AGENTS.md`。
> 快照日期：2026-11-18。

## 环境

- 工作站：macOS（darwin arm64，Apple M3 Pro）。所有 bash / 测试在本机执行。
- Python：`uv` 管理，虚拟环境 `.venv/`（已存在）；测试入口 `./scripts/test.sh`（自动解析 PYTHON/venv/uv）。
- 前端：`lightrag_webui/` 用 Bun；`bun test` 必须在该目录内跑（根目录跑会静默丢失 DOM preload）。
- 工作数据目录：`inputs/`、`rag_storage/`、`temp/`（当前不存在，用到时生成）。

## 工作流要点（指针）

- 测试子集规则、marker 要求、日志捕获 fixture：根 AGENTS.md *Testing*；后端测试写法：`docs/design/BackendTestingGuide.md`。
- WebUI 改动 = 全量前端检查（bun test + tsc --noEmit + lint）：根 AGENTS.md *WebUI* / *React component tests*。
- README 四语逐行对齐，改一必改四：根 AGENTS.md *README translations*。
- docstring 只写规则，机制进 `docs/design/`；80/100 行预算与 issue 引用禁令由 `tests/test_docstring_budget.py` 强制。
- PR 目标仓库是 `HKUDS/LightRAG`（本仓库即上游，非 fork 场景时按实际 remote 判断）。

## 本地工具链

- Agent 配置：`opencode.json`（未跟踪）；`.omo/`（opencode 运行状态）；`.clinerules/01-basic.md`（历史遗留规则）。
- Claude Code 配置（`CLAUDE.md`、`.claude/`）已在本地删除，迁移到 opencode；`AGENTS.md` 为唯一入口。
- 开发记忆区：`.memory/`（本目录），随 `dev` 分支提交并推送到 fork（`shadowinlife/LightRAG`）；`main` 不跟踪。

## 易踩坑（本机实证或契约明示）

- 新建 `LightRAG` 实例后必须 `await rag.initialize_storages()`，否则 `AttributeError: __aenter__` / `KeyError: 'history_messages'`。
- `lightrag` logger `propagate=False`：测试里用 `lightrag_caplog` / `lightrag_log_records` fixture，裸 caplog 在旧 pytest 上捕获不到。
- 换 embedding 模型用 `lightrag-rebuild-vdb`，不要删工作目录。

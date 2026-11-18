# 项目状态快照

> 追加式：新快照加在最上面，旧快照保留作历史。每条带日期与核实方式。

## 2026-11-18（初始快照，核实方式：git status / git log / README News）

- 仓库：`HKUDS/LightRAG`（origin = github.com/HKUDS/LightRAG.git），分支 `main`，HEAD `264b00ebf`（Merge PR #4463, fix/issue-4443）。
- 本地未提交改动：
  - `AGENTS.md` 已修改（+14/-5）——本地记忆层规则正在演进。
  - 已删除：`CLAUDE.md`、`.claude/settings.json`、`.claude/hooks/session-start.sh`（Claude Code 配置迁移到 opencode）。
  - 未跟踪：`GLOSSARY.md`（已被根 AGENTS.md 引用，属进行中的记忆层工作）、`opencode.json`、`.omo/`。
  - `.memory/` 初始为纯本地（.git/info/exclude 忽略）；2026-11-18 起改为随 `dev` 分支提交，见上方最新快照。
- 上游近期里程碑（README News）：
  - 2026.07：Word 文档 Smart Heading 识别。
  - 2026.05：RagAnything 并入（MinerU / Docling 多模态解析）；可选 chunking 策略（F/R/V/P/C）；角色级 LLM 配置（EXTRACT/QUERY/KEYWORDS/VLM）。
  - 2026.03：OpenSearch 统一存储后端；交互式 setup wizard。
- 测试 / CI：全量离线套件由 CI 分片跑；本地只跑改动模块的镜像目录（规则见根 AGENTS.md *Testing*）。

## 待办 / 已知问题

- （空——发现后记在这里，带日期；持久性问题应进 issue 或契约，不长期滞留此处）

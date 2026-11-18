# .memory — 开发记忆区

本目录是开发记忆，随 `dev` 分支版本化，推送到个人 fork（`shadowinlife/LightRAG`，remote 名 `fork`）；上游 `HKUDS/LightRAG` 与本地 `main` 不跟踪它。注意：切回 `main` 时 git 会把本目录从工作区移除（它只在 `dev` 上被跟踪），回 `dev` 即恢复。
仓库的规范记忆层是根 `AGENTS.md`、`docs/design/` 契约与 `GLOSSARY.md`；本目录只做索引、快照与迭代跟踪。两者冲突时，以规范层为准。

## 布局

- `wiki/` — 项目状态、架构地图、开发上下文。入口：`wiki/index.md`。
- `iteration/` — 目标与 big feature 的调研 / 执行跟踪。入口：`iteration/index.md`。

## 读写规则

- 会话开始需要项目背景时：读 `wiki/index.md`，按其指针取所需。继续一个进行中的 feature 时：读 `iteration/active/` 下该 feature 的 `status.md`。
- wiki 写「指针 + 快照」：架构与契约细节指向 `docs/design/` 和源码 docstring，正文留在原地（复制会漂移）；状态类内容一律带日期。
- 一个 big feature 一个目录：`iteration/active/<YYYYMMDD>-<slug>/`，从 `iteration/_template/` 复制三件套（brief / research / status）。调研结论进 `research.md`，带来源与日期；执行进展与决策进 `status.md` 的追加式日志。
- feature 完成或放弃：整目录移入 `iteration/archive/`，同步更新 `iteration/index.md` 的表格。
- 迭代中产生持久决策（难以逆转、有真实取舍）时：按根 `AGENTS.md` 的 Memory maintenance 规则晋升进对应 `docs/design/` 契约，本地只留指针。
- 笔记可用中文；代码标识符、路径、命令保持英文。

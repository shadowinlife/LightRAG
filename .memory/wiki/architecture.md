# 架构地图（指针版）

> 权威描述在根 `AGENTS.md` 与 `docs/design/` 契约；本页只做「触哪里 → 先读什么」的导航。
> 快照日期：2026-11-18。结构变化时更新本页，细节变化只更新契约。

## 核心组成

- `LightRAG` 由 mixin 组装：`LightRAG → _RoleLLMMixin → _StorageMigrationMixin → _PipelineMixin → object`（`@final`，mixin 是内部实现细节）。
- 四类存储：KV / VECTOR / GRAPH / DOC_STATUS，后端可插拔（`lightrag/kg/`，注册表在 `kg/__init__.py`，解析在 `kg/factory.py`）。
- 摄取管线在 `lightrag/pipeline.py`（`_PipelineMixin`），纯辅助函数在 `utils_pipeline.py`；解析层 `lightrag/parser/`（legacy / native / mineru / docling）。
- 查询五模式：local / global / hybrid / naive / mix（默认 mix，推荐配 reranker）。核心逻辑在 `operate.py`。
- API：`lightrag/api/lightrag_server.py`（FastAPI）；前端：`lightrag_webui/`（React 19 + Bun + Vite）。
- 术语定义：`GLOSSARY.md`（workspace、instance、business storage 等）。

## 契约索引（改对应代码前必读）

| 契约（docs/design/） | 管辖范围 |
|---|---|
| NetworkXSingleWriterContract.md | `kg/networkx_impl.py`、graph store 的 `index_done_callback` 调用方 |
| FileBackedSnapshotContract.md | `kg/nano_vector_db_impl.py`、`faiss_impl.py`、`json_kv_impl.py`、`json_doc_status_impl.py`、`file_fingerprint.py` |
| ServerInstanceContract.md | 同主机多实例共享的一切（WORKING_DIR/INPUT_DIR 文件、无 workspace 键的记录、启动锁） |
| PipelineConcurrencyContract.md | `pipeline.py`、`kg/pipeline_ingress.py`、`pipeline_status`、`/documents/*` |
| PurgeRecoveryContract.md | `_purge_kg_contributions`、`adelete_by_doc_id`、merge 锚点写入、`kg_write_state`/`kg_purge`、缓存写序 |
| ConfigurationStorageContract.md（概览：ConfigurationStorageDesign.md） | `config_store.py`、`config_anchor.py`、`config_shards.py`、anchor/working_dir 锁、`config` KV 命名空间、初始化/清理流程 |
| BackendTestingGuide.md | 写 / 改 `tests/` 下任何后端测试 |
| WebUITestingGuide.md | 写 / 改 `lightrag_webui/src/` 下任何测试 |
| LatexEscapeRepairContract.md / PdfExtractionBudget.md / PgVectorFilteredSearch.md / PostgresLegacyVectorMigration.md / VectorSpaceProvenance.md | 各专项机制（按需读） |

另：关系权重契约在 `docs/ProgramingWithCore.md#relation-weight-contract`；SDK 用法同文档。

## 跨切面原则（详见根 AGENTS.md）

- 无跨存储事务：不一致要么自愈、要么方向无害；每个接受的残留必须写进契约。
- 文件后端仅供小规模测试；生产推荐 PostgreSQL 单后端。
- 静默失败零容忍：持久化写入失败必须响亮报错，再按契约残留自愈。

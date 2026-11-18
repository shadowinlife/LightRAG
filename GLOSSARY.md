# LightRAG

LightRAG is a graph-based retrieval-augmented generation framework: it extracts entities and relations from documents, builds a knowledge graph, and answers queries from graph and vector context. This glossary names the vocabulary its contracts use; mechanisms live in `docs/design/`, rules in docstrings and `AGENTS.md`. Homonyms are disambiguated by qualifier and `_Avoid_` lines.

## Language

### Workspace and instance

**Workspace**:
The named data-isolation scope every business storage, upload directory and configuration record is partitioned by.
_Avoid_: database, collection (a workspace maps onto them but is not one)

**Instance**:
One process tree serving one workspace: a single uvicorn process, or a Gunicorn master with its forked workers.
_Avoid_: worker, host (workers of one master are one instance)

**Empty workspace**:
The default workspace, named by the empty string, whose files sit at the root of the working directory rather than a per-workspace subdirectory.

**Physical partition**:
The concrete file, table, collection, index, graph or column value a backend maps a workspace onto.

**Workspace collision**:
The condition in which two workspaces occupy one physical partition, so each instance reads, writes and deletes the other's data.

**Business storage**:
A storage holding one workspace's knowledge-base data — chunks, graph, vectors, document status — as distinct from the configuration storage.

**Server-wide record**:
A record not scoped by workspace that every instance on a host shares, and that therefore needs a guard the instances share.

### Pipeline concurrency

**Pipeline status**:
The per-workspace shared state recording the pipeline's current holder, its owner, and its ingress and scan states.

**Busy**:
The pipeline state meaning an operation currently holds the workspace; the holder may be a processing run, a destructive job, or an admin graph write.
_Avoid_: processing busy (admin writes also hold it)

**Destructive busy**:
The busy state whose holder is a document clear or a per-document delete; the subset of busy that refuses new enqueues.
_Avoid_: busy (busy alone does not block enqueue)

**Scanning exclusive**:
The scanning state during a scan's classification phase only; the phase that refuses new ingress.
_Avoid_: scanning (the whole scan lifecycle)

**Manual freeze requested**:
The ingress state of a manual FAILED retry while the pipeline is drained to idle; an owned freeze, never a bare flag.

**Ingress fence**:
One of the pipeline states that refuse new enqueue reservations: scanning exclusive, destructive busy, or manual freeze requested.

**Reservation**:
A claim recorded in pipeline status that an operation is in progress, carrying its kind and owner; the unit by which concurrent operations exclude one another.
_Avoid_: lock (a reservation is a state claim; the admin lock is a separate lock)

**Ingress mailbox**:
The per-workspace three-channel mailbox that is the pipeline's only wake-up channel, carrying document, auto-rescan and manual-retry notifications.
_Avoid_: pipeline status field (it lives beside pipeline status, never inside it)

**Auto-rescan**:
A sticky mailbox request for a processing run that was refused while the pipeline was busy; a pending request, not a running job.

**Manual retry request**:
A sticky request granting FAILED documents one re-entry into the pipeline; the only path by which FAILED documents re-enter.
_Avoid_: automatic retry (FAILED documents never resume automatically)

**Strict scan**:
A document-status query that returns a complete result or raises; the only form the scheduling control plane uses.

**Admin graph write**:
Any public write to the knowledge graph (create, edit, delete, merge, custom-KG insert), as distinct from pipeline-driven writes.

**Admin write gate**:
The exclusive path through which every admin graph write runs on a single-writer graph storage: the admin lock, a busy reservation, then the per-entity keyed locks, in that order.
_Avoid_: admin lock (the lock is one component of the gate)

**Recovery fence**:
A workspace-level block that closes the workspace until an operator clears it with a force reset.

### File-backed storage

**File-backed storage**:
A storage whose authoritative data is one whole file rewritten on each commit, kept in process memory; supported for small-scale testing and validation only.

**Two-channel fence**:
The pair of independent checks — the file fingerprint and the storage_updated flag — whose OR decides whether a process's snapshot is current.

**storage_updated**:
The per-namespace flag signaling a change to peers; it means "a peer committed" for reload-based stores and the OPPOSITE, "dirty, still to flush", for the shared-dict JSON stores.

**Tick collision**:
The file fingerprint's blind spot: two commits sharing one timestamp tick and one file size, so the second is indistinguishable from the first.

**Decline**:
The graph store's write verdict in which a process abandons its own pending mutation because a peer commit superseded its snapshot.

**Reload-then-replay**:
The vector stores' write-conflict resolution: reload the peer snapshot and replay pending work on top.
_Avoid_: decline (that is the graph store's verdict)

**Pending buffer**:
The queue of operations accepted but not yet materialized into the store.

**Redo log**:
The record of operations applied in memory but not yet durable, kept so a reload can replay them.

**namespace_init_claim**:
The one-winner claim under which a namespace's first load from its file runs, so exactly one process populates the shared in-memory copy.

**Process tree**:
All processes sharing one fork lineage; one instance.

**Accumulate-over-read**:
The property of graph payloads whereby a write accumulates over the state its author read, so replaying it over newer state loses evidence.

### Purge and recovery

**Purge**:
The removal of a document's knowledge-graph contributions — entities, relations, chunks and attribution — as opposed to just its status row.

**Recovery anchor**:
The per-document write-ahead rows recording everything a document contributed, written before the first graph mutation so a purge can answer what to remove.
_Avoid_: configuration anchor (a different concept)

**Attribution carrier**:
A stored row whose content names objects — a chunk row, an anchor row, or a chunk's llm_cache_list — so deleting it without those objects strands them.

**kg_write_state**:
The monotonic document-status marker of how far a document's KG write got, written once at enqueue and never backfilled.

**Purge journal**:
The ordered, resumable record of a purge's phases, required because the purge's last step deletes the anchors.

**Purge proof**:
One of the four evidences — anchors, pre_graph, journal, empty scope — that permits removing an attribution carrier.

**Chunk tracking**:
The per-object authoritative record of which chunks contribute to an entity or relation, outranking the graph's source_id.

**Orphan**:
A graph object no surviving record can attribute to a document.

**DELETING**:
The document status marking a deletion in progress, excluded from every scheduling allowlist so it is never resumed as ingestion.

**Extraction cache reachability**:
The property by which an extract cache row is reachable only through the llm_cache_list of the chunk that owns it.

**Rows superset of graph**:
The accepted inconsistency direction in which tracking rows claim more than the graph holds; the reverse direction loses data.

### Configuration storage

**Configuration container**:
The one fixed file, table, collection or index that holds every configuration row, which no setting and no workspace can address.

**Container identity**:
The UUID recorded inside the configuration container identifying it as the container a deployment is bound to.
_Avoid_: deployment id (explicitly not one)

**Configuration anchor**:
The file under the working directory recording which backend and container identity this deployment is bound to.
_Avoid_: recovery anchor (the purge term)

**Bind / rebind**:
Bind is the first-time establishment of a container's identity and its anchor; rebind is the sanctioned anchor deletion with every writer stopped, so the next start binds to the currently selected container.

**Anchor lock**:
The lock starters hold shared and the offline configuration migration holds exclusively.

**Bind lock**:
The exclusive lock serializing a first bind and a JSON workspace's member registration across servers sharing one working directory.
_Avoid_: anchor lock (a different lock)

**Snapshot claim**:
The lifetime claim a JSON configuration storage takes on its workspace's snapshot directory, admitting one process tree per snapshot.

**JSON configuration shards**:
The per-workspace configuration snapshots under one working directory that together form one logical container.

**Embedding baseline**:
The recorded decision of the embedding space adopted as one vector target's active baseline for one workspace.

**Sticky failure**:
A startup failure retained and re-raised by every later call on the same instance, so a new instance is the retry.

**Strict read**:
A read whose failure is a failure, never "the record is absent".

### Vector space

**Embedding space**:
The vector space defined by a particular embedding model; vectors from different models are not comparable even when their dimensions match.

**Vector-space provenance**:
The knowledge of which embedding model and dimension produced the vectors in a container.

**Model-isolation suffix**:
The suffix a named-container backend appends to its container name, derived from the current embedding model and dimension.

**Vector-space marker**:
A record of a container's embedding model kept outside the data plane, used by backends whose container names carry no model information.
_Avoid_: metadata record (a marker must never be recallable by search)

**Adoption probe**:
The evidence for adopting a non-empty container: re-embed sampled stored content with the current model and compare each result with its stored vector.

**Coverage gate**:
The startup check that refuses to serve when a vector index is empty while the data it indexes is populated.

**Fail open / fail closed**:
Fail open proceeds with a warning despite a problem; fail closed refuses to serve, names what changed, and names the recovery.

### Parsing and repair

**Legacy engine**:
The local pypdf-based parser engine, as distinct from the native and external engines.

**LaTeX escape damage**:
The corruption of a LaTeX command when a single backslash inside a JSON string decodes as a JSON escape, consuming the command's first letter.
_Avoid_: invalid escape (invalid escapes are preserved verbatim and need no repair)

**Residue**:
The tail of a LaTeX command remaining after a decoded escape letter, chosen to collide with no English word.
_Avoid_: accepted residue (a different concept)

**Content gate (LaTeX)**:
The half of the repair decision that inspects a paired span's body to decide whether it is math rather than code.
_Avoid_: content-bytes gate (the PDF term)

**Content-bytes gate (PDF)**:
The deterministic pre-parse gate on the total decoded size of a PDF's page content streams.
_Avoid_: content gate (the LaTeX term)

**Silent rewrite**:
A repair applied to text that is not math, corrupting prose without a visible failure.
_Avoid_: repair (a rewrite is corruption, not a repair)

**PDF extraction budget**:
The set of limits bounding the cost and resources of one legacy PDF extraction.

**Budget refusal**:
A PDF extraction outcome in which a limit stopped the worker.
_Avoid_: error (a budget refusal is a distinct outcome kind)

### Testing

**Contract**:
An authoritative written statement of promised behavior — a docstring, a docs/design contract, or an existing test; the only thing a test may pin.

**Pin**:
A test assertion that fixes a behavior the contract promises.
_Avoid_: incidental assertion (pinning an incidental detail is the defect)

**Mutation check**:
The practice of breaking the behavior a test pins, confirming the test goes red, and restoring it.

**Colocated test**:
A WebUI test file living beside the module it covers in src/.
_Avoid_: mirror layout (the backend convention)

**Mirror layout**:
The backend convention in which the tests/ tree mirrors the package layout under lightrag/.

**Store seeding**:
The practice of pre-populating the stores a page reads instead of stubbing its requests.

**Process-wide DOM**:
The condition that all WebUI test files share one process, and therefore one window and document.

### Cross-cutting decisions

**Accepted residue**:
A documented inconsistency accepted as a decision because it heals itself or is harmless in direction, recorded with its reason and recovery path.
_Avoid_: accepted miss, accepted limit, unsupported (different accepted outcomes)

**Accepted miss**:
A damaged shape a repair deliberately leaves unrepaired, keeping its warning.

**Accepted limit**:
A documented exposure the design accepts on a particular platform, with its reason.

**Fail-loud**:
A condition that raises rather than silently dropping or rebuilding state that exists but cannot be read back.

**Harmless in direction**:
The property of an inconsistency whose retained state is the safer of the two possible directions.

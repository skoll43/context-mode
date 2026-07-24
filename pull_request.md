# [WIP] feat(session): introduce lightweight SQLite causal edge graph (`session_edges`)

## What / Why / How

Introduces a lightweight SQLite relation table (`session_edges`) to `SessionDB` to support zero-latency causal event linking across developer sessions.

### Problem
When AI agents encounter runtime errors or missing context during tool invocation, standard search models rely on multi-turn exploratory search loops (`grep_search` $\rightarrow$ `view_file` $\rightarrow$ `grep_search`). This consumes significant LLM token budget across context turns.

### Solution
Adds a native, zero-dependency SQLite relation graph table (`session_edges`) within the existing `SessionDB` schema. Enables linking causal event pairs (e.g. `(Error Event)` $\xrightarrow{\text{RESOLVED\_BY}}$ `(Fix Event / Decision)`) with confidence scoring (`0.0` to `1.0`) and single-query recursive traversal (`WITH RECURSIVE`).

### Implementation Details
- **Schema Addition:** Adds `session_edges` table and indices on `source_id` and `target_id` inside `SessionDB.initSchema()` ([src/session/db.ts](file:///c:/Users/lukas/Downloads/NEW/src/session/db.ts)).
- **API Helpers:** Adds `insertEdge(sourceId, targetId, relationship, confidence)` and `getLinkedEdges(nodeId, minConfidence)` methods to `SessionDB`.
- **Windows Safety:** Incorporates explicit `db.close()` handles in test suites to prevent Win32 file-locking `EPERM` hangs during teardown.

## Affected platforms

- [x] All platforms

## Test plan

- Unit tests added in `tests/session/causal-edges.test.ts` to test:
  - Schema initialization and idempotent migration.
  - Edge insertion and confidence score filtering.
  - Recursive CTE 1-hop and 2-hop graph traversals.
- Verified Windows file handle teardown cleanliness.

## Checklist

- [ ] Tests added/updated (TDD: red → green)
- [ ] `npm test` passes
- [ ] `npm run typecheck` passes
- [ ] Docs updated if needed (README, platform-support.md)
- [ ] No Windows path regressions (forward slashes only)
- [x] Targets `next` branch (unless hotfix)

<details>
<summary><strong>Cross-platform notes</strong></summary>

Our CI runs on **Ubuntu, macOS, and Windows**.

- If touching file paths, verify forward-slash normalization on Windows
- If touching hook paths, verify no backslash separators
- Use `path.join()` / `path.resolve()`, never hardcode `/` separators
- Use event-based stdin reading — `readFileSync(0)` breaks on Windows
- Use `os.tmpdir()`, never hardcode `/tmp`

</details>

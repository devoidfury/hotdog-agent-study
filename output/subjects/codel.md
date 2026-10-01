# Anchor review: codel (semanser/codel, Go backend + React frontend) — D (23.0)

Tier: census says T1; protocol says **T0** (dead >12mo at HEAD). Shallow clone masked this; verified against remote: `pushed_at 2024-04-29` (GitHub API, semanser/codel, not archived but abandoned for ~2.5yr). AGPL-3.0. Module path still `github.com/semanser/ai-coder` (project rename, not a clone).

**Closest anchor: n/a — is anchor.** D reference: what it means to be an agent-*shaped* product with no agent harness inside.

## Dimensions

### architecture — 3
- There is no agent loop. The "agent" is a Postgres task queue plus a one-shot oracle: `Provider.NextTask(args) *database.Task` picks exactly one next task per poll (`backend/providers/providers.go:23-31`, impls `openai.go:57`, `ollama.go:89`); executors (`backend/executor/processor.go`) consume task rows and write results back.
- Context = every task serialized into one prompt string (`ollama.go:101`), no state model beyond DB rows.
- Credit where due: the backend itself is tidily layered Go (sqlc `database/`, gqlgen `graph/`, `executor/`); for 14.6k LOC it's readable.

### verification — 0
- Zero test files (`find -name '*_test.go'` = 0).
- Only workflow is `release.yml` (Docker image publish); nothing tests anything, ever.

### safety-enforcement — 2
- Policy is prompt text: "Always auto approve terminal commands whenever it's possible" (`backend/templates/prompts/agent.tmpl:13`); no approval code exists at all.
- The control plane (GraphQL + websocket that spawns containers and runs terminal commands) has no authentication - `backend/router/router.go` configures CORS and nothing else (:31).
- Isolation exists (each flow runs in a Docker container, `executor/container.go:45 SpawnContainer`) which reads as "sandboxed" but doesn't protect the API or host mounts; present-but-misleading fits the anchor exactly.

### token-economy — 1
- Hard failure instead of compaction: prompt > 30000 chars returns `defaultAskTask("My prompt is too long...")` (`ollama.go:101-104`); summarization acknowledged as needed but unimplemented ("TODO ... summary using gpt-3.5", :100); `summary.tmpl` exists unused for context management.
- No cache, no cost tracking, no truncation of tool outputs.

### orchestration — 4
- Durable queue is the one real machinery: tasks/containers/logs persist in Postgres (`database/tasks.sql.go`, `containers.sql.go`), queue processor resumes polling on restart (`executor/queue.go`), realtime via subscriptions + websocket (`graph/subscriptions`, `websocket/`).
- Gaps: no budgets, no crash-recovery semantics for in-flight container commands, human ("ask") tasks are the only error path.

### interop — 2
- Surfaces: GraphQL API + websocket + web UI; providers are hardcoded OpenAI and Ollama (`providers/openai.go`, `ollama.go`).
- No MCP, no ACP, no SDK, no IDE surface, no config beyond env keys.

### operability — 3
- Dockerfile + docker-compose quickstart; flows visible/editable in the web UI; DB rows make state inspectable.
- No resume/rewind, no diagnostics, errors surface as `log.Printf` + failed task rows (e.g., `executor/processor.go:21-49` swallows most failures into logs).

### originality — 5
- The 2024-era Devin-wannabe shape is genuinely distinctive for its time: agent-as-database-queue with per-flow containerized terminal/browser/editor toolset and mandatory human "ask" plan confirmation (`agent.tmpl:11`) - a fully human-supervised autonomy model.
- Original idea, thin implementation: the mechanism behind it is one prompt and a poller.

### durability — 0
- Abandoned: last push 2024-04-29 (verified via remote, defeating the shallow-clone blindfold), solo author, 28 open issues, no releases workflow beyond docker publish. Rule (b) dead-cap is moot at this total.

### docs-dx — 3
- README + DEVELOPMENT.md cover docker-compose startup; claims "Fully autonomous AI Agent" while the prompt mandates "Always output your plan as the first `ask`" (`agent.tmpl:11`) - marketing contradicts behavior; no reference docs.

## Verdict
**Strongest: originality (5).** Weakest: verification (0) / durability (0).
Weighted 23.0 -> **D**. D-anchor lesson: presence of agents-*like* vocabulary (flows, tasks, containers, browser tools) in census or README tells you nothing about harness depth; the disqualifying trio is no loop, no tests, no maintenance. Census corrections: suggested_tier T1 -> T0 (dead verified remotely); manifest_name null -> module `github.com/semanser/ai-coder` (rename, benign).

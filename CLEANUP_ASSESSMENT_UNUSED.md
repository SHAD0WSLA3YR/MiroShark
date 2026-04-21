# Cleanup Assessment — Unused / Dead Code

Agent: `agent-ad7ca49f` (worktree branch: `worktree-agent-ad7ca49f`)
Scope: Unused imports, unused local variables, unused exports, unused files.

## Tool Output Summary

### Backend — `ruff check --select F401,F841 app/ scripts/ wonderwall/`
- 76 findings in total.
- 63 auto-fixable (`ruff check --fix`).
- 12 additional hidden fixes (behind `--unsafe-fixes`).

### Backend — `vulture app/ scripts/ wonderwall/ --min-confidence 80`
- 5 findings (subset of ruff + two function-parameter variables that are actually API surface, not dead code).

### Frontend — `knip`
- 1 "unused file": `public/sw.js` (false positive — registered dynamically in `main.js`).
- 1 unresolved import: `@/api/simulation` from `src/components/EmbedDialog.vue` (pre-existing bug — no `@/` Vite alias is configured; see Out-of-scope below).
- 13 unused exports (all `src/api/*.js` helpers — public API wrapper, MEDIUM risk).

### Frontend — `depcheck`
- No issues.

---

## HIGH CONFIDENCE — Removed

All items grep-verified against `app/`, `scripts/`, `wonderwall/`, `tests/`, `frontend/`, `docs/`, root, and the `miroshark` CLI shell script.

### Unused imports (F401) — Backend

| File | Imports removed |
|------|-----------------|
| `app/api/observability.py` | `typing.Optional` |
| `app/api/report.py` | `..config.Config` |
| `app/api/simulation.py` | `tempfile` (top), inner `os`, inner `..config.Config`, inner `datetime.datetime`, inner `math` |
| `app/models/project.py` | `dataclasses.asdict` |
| `app/services/agent_mcp_tools.py` | `typing.Any`, `typing.Callable` |
| `app/services/graph_builder.py` | `..config.Config` |
| `app/services/graph_memory_updater.py` | `os`, `json`, `typing.Callable`, `..config.Config` |
| `app/services/oasis_profile_generator.py` | `time`, inner `re` |
| `app/services/ontology_generator.py` | `json` |
| `app/services/push_notification_service.py` | `pywebpush.WebPushException` (reduced `from pywebpush import webpush, WebPushException` → `from pywebpush import webpush`) |
| `app/services/report_agent.py` | `time`, `dataclasses.field`, `SearchResult`, `InsightForgeResult`, `PanoramaResult`, `InterviewResult`, inner `math` |
| `app/services/simulation_config_generator.py` | inner `re` |
| `app/services/simulation_manager.py` | `..config.Config`, `.entity_reader.FilteredEntities`, `.oasis_profile_generator.OasisAgentProfile`, `.simulation_config_generator.SimulationParameters` |
| `app/services/simulation_runner.py` | `asyncio`, `typing.Union`, `..config.Config`, `CommandType`, `IPCResponse`, inner `shutil` |
| `app/services/text_processor.py` | `typing.Optional` |
| `app/services/web_enrichment.py` | `logging` |
| `app/storage/entity_resolver.py` | `json` |
| `app/storage/neo4j_storage.py` | `neo4j.Session` |
| `app/storage/ner_extractor.py` | `typing.List` |
| `app/utils/event_logger.py` | `time`, `typing.Generator` |
| `app/utils/file_parser.py` | `os`, `typing.Optional` |
| `app/utils/llm_client.py` | `.trace_context.TraceContext` |
| `app/utils/run_summary.py` | `datetime.datetime` |
| `scripts/belief_integration.py` | `json`, `wonderwall.social_agent.round_analyzer.RoundSnapshot` |
| `scripts/round_memory.py` | `json` |
| `scripts/run_parallel_simulation.py` | `multiprocessing` (top-level; inner-scope `from multiprocessing import resource_tracker` stays), `warnings` |
| `scripts/run_reddit_simulation.py` | `update_trust_from_actions` |
| `scripts/test_full_pipeline.py` | `create_llm_client` |
| `wonderwall/simulations/base.py` | `typing.Callable`, `typing.Optional` |
| `wonderwall/simulations/polymarket/environment.py` | `json` |
| `wonderwall/simulations/polymarket/platform.py` | `json` |
| `wonderwall/social_agent/belief_state.py` | `json` |
| `wonderwall/social_agent/round_analyzer.py` | `typing.Tuple` |

All of the above were grep-verified as having zero other references in the file. None are in `__init__.py` namespace re-export blocks.

### Unused local variables (F841) — Backend

| File | Variable removed |
|------|------------------|
| `app/api/simulation.py` | `config_file` (line 1454) |
| `app/services/graph_tools.py` | `source_name`, `target_name` (lines 1107–1108) |
| `app/services/oasis_profile_generator.py` | `e` in `except json.JSONDecodeError as e` → `except json.JSONDecodeError` (line 743) |
| `app/services/report_agent.py` | `level` (line 3190), `folder` (line 3304) |
| `app/services/simulation_config_generator.py` | `entity_types_available` (line 700) |
| `app/utils/llm_client.py` | `error_info` (line 266 initial `= None`, line 270 assignment inside except) — both removed since it's never read |
| `scripts/market_media_bridge.py` | `e` in except binding |
| `scripts/run_parallel_simulation.py` | `exc` in except binding |
| `scripts/test_e2e_api.py` | two `result = ...` that are never used |
| `scripts/test_full_pipeline.py` | `config` |
| `scripts/test_pipeline_phase5_6.py` | `sim_time` |
| `wonderwall/social_agent/round_analyzer.py` | `active_set` |

All confirmed via grep — not referenced elsewhere in the function scope.

---

## MEDIUM CONFIDENCE — Flagged, NOT removed (human review)

### Frontend — unused API helper exports (`knip`)

The following exports are flagged by knip because no other file in `frontend/src/` imports them. They are part of `frontend/src/api/*.js` — a deliberate thin wrapper layer around backend endpoints. Removing them would hide live REST endpoints (`/api/observability/events`, `/api/observability/llm-calls`, `/api/report/generate/status`, `/api/simulation/*/profiles`, `/api/simulation/*/posts`, `/api/simulation/*/timeline`, `/api/simulation/*/agent-stats`, `/api/simulation/*/restart`, `/api/simulation/vapid-public-key`, `/api/simulation/subscribe`, `/api/simulation/test-push`, `/api/simulation/*/publish`, `/api/simulation/*/frame`). These endpoints may be reachable via other code paths (debug tools, browser console, Embed iframe) or planned UI work.

- `src/api/observability.js` → `getEvents`, `getLlmCalls`
- `src/api/report.js` → `getReportStatus`
- `src/api/simulation.js` → `getSimulationProfiles`, `getSimulationPosts`, `getSimulationTimeline`, `getAgentStats`, `restartEnv`, `getVapidPublicKey`, `subscribePush`, `testPushNotification`, `publishSimulation`, `getSimulationFrame`

Recommendation: leave for a frontend owner to decide (safer to trim only after confirming no live console/debug usage).

### Function parameters flagged as unused by vulture

- `wonderwall/social_agent/round_analyzer.py:285` — `belief_state: Optional[BeliefState]` parameter of `generate_agent_feedback`. Callers in `scripts/run_reddit_simulation.py` and `scripts/belief_integration.py` pass it positionally; removing it would break the call sites. Signature is part of the Wonderwall extension surface.
- `wonderwall/social_platform/recsys.py:428` — `recall_only: bool = False` parameter of a public recsys function. Preserved as a public API knob.

### Unused exports that look dead but are public API

- `app/services/__init__.py` re-exports `SimulationState` and the simulation_runner symbols (`RunnerStatus`, etc). These stay.

---

## Items that LOOKED unused but are kept (dynamic references)

| Item | Why kept |
|------|----------|
| `frontend/public/sw.js` | Registered via string in `frontend/src/main.js:13` (`navigator.serviceWorker.register('/sw.js')`). |
| `backend/scripts/*.py` (all) | Per task rules: CLI scripts meant to be invoked directly. `scripts/test_*.py` are legacy-integration tests referenced from `backend/tests/test_integration_legacy_scripts.py`. |
| `backend/cli.py::main` | Registered as console script `miroshark-cli` in `backend/pyproject.toml`. |
| `backend/mcp_server.py` | MCP server entry point (referenced from `miroshark` shell CLI). |
| `backend/run.py` | Entry point (referenced from `miroshark` shell CLI). |
| Flask blueprints in `app/api/*.py` | Registered via side-effect imports in `app/api/__init__.py`. |
| `wonderwall/` package | Imported dynamically as a bundled social-simulation package used at runtime via camel-ai / subprocess. |
| Function parameters like `belief_state`, `recall_only` | API signature, caller-compatible. |
| `pywebpush.webpush` | Kept, only `WebPushException` removed (not referenced). |
| `*.get_*`, `*.list_*` API wrappers in frontend | May be used by embed / devtools / future work (see MEDIUM above). |

---

## Out-of-scope issue surfaced (not fixed)

`frontend/src/components/EmbedDialog.vue:118` imports from `@/api/simulation`, but the `@` alias is not declared in `vite.config.js` (no `resolve.alias` stanza). This breaks `npm run build` and is unrelated to dead-code cleanup. Left to the frontend-owner agent.

Impact on this task: frontend build verification (`npm run build`) fails purely due to this pre-existing bug, not due to any removal performed here.

---

## Verification Protocol

1. `python -m pytest --collect-only` — 79 tests collected both before and after changes.
2. `ruff check --select F401,F841 app/ scripts/ wonderwall/` — zero findings after changes.
3. `npm run build` — blocked by the pre-existing `@/api/simulation` bug (out of scope). Hand-grep of every removed symbol confirms no other frontend references.

# Nanobot Study Guide for Senior Backend Engineers

## 1. Goal and Approach
This guide is optimized for quickly building a maintainer-level mental model of `nanobot` and turning that into actionable engineering output (debugging, feature work, plugin development, and API hardening).

Learning strategy:
1. Build runtime flow understanding first.
2. Map extension points (channels/tools/providers/config).
3. Use tests as executable documentation.
4. Produce artifacts while learning (notes, call graphs, hypotheses, small patches).

## 2. Fast Project Map
Core runtime layers:
1. Entry and CLI surface: `nanobot/__main__.py`, `nanobot/cli/commands.py`
2. Agent runtime engine: `nanobot/agent/loop.py`, `nanobot/agent/runner.py`, `nanobot/agent/context.py`
3. Tooling and execution boundaries: `nanobot/agent/tools/*`, `nanobot/security/network.py`
4. Messaging and transport: `nanobot/bus/*`, `nanobot/channels/*`, `nanobot/channels/manager.py`
5. Provider abstraction: `nanobot/providers/*`, `nanobot/providers/registry.py`
6. State and memory: `nanobot/session/manager.py`, `nanobot/agent/memory.py`, `nanobot/agent/autocompact.py`
7. Config system: `nanobot/config/schema.py`, `nanobot/config/loader.py`
8. External API surface: `nanobot/api/server.py`

High-value docs:
1. `README.md` (architecture, config, CLI, API, security)
2. `docs/CHANNEL_PLUGIN_GUIDE.md` (plugin extension model)
3. `docs/WEBSOCKET.md`, `docs/MEMORY.md`, `docs/PYTHON_SDK.md`

## 3. Suggested Learning Order (7 Sessions)
## Session 1: Orientation (60-90 min)
1. Read `README.md` sections: Features, Architecture, Configuration, CLI, API, Security, Project Structure.
2. Run the basics:
```bash
nanobot onboard
nanobot status
nanobot agent --message "Explain your available tools briefly"
```
3. Output artifact: one-page note of top-level components and unknowns.

## Session 2: Runtime Call Flow (90-120 min)
Trace one user message end-to-end:
1. CLI command path in `nanobot/cli/commands.py` (`agent`, `gateway`, `serve`).
2. `AgentLoop` initialization and `_register_default_tools()` in `nanobot/agent/loop.py`.
3. `process_direct(...)` and tool execution cycle in agent runtime modules.
4. Output artifact: sequence diagram (text is fine).

## Session 3: Configuration and Environment Strategy (60 min)
1. Read `nanobot/config/schema.py` and `nanobot/config/loader.py`.
2. Focus on provider selection (`Config._match_provider`) and tool constraints (`restrict_to_workspace`, exec/web settings).
3. Output artifact: config decision matrix (which fields matter for local dev vs production).

## Session 4: Channels and Plugin System (90 min)
1. Read `nanobot/channels/base.py`, `nanobot/channels/manager.py`, `nanobot/channels/registry.py`.
2. Read `docs/CHANNEL_PLUGIN_GUIDE.md` and map it to registry/discovery code.
3. Run:
```bash
nanobot plugins list
```
4. Output artifact: minimal plugin design doc (message ingestion, delivery, auth, retry, streaming support).

## Session 5: API and Integration Surface (60-90 min)
1. Read `nanobot/api/server.py`.
2. Verify session model (`api:default` or `session_id` override), timeout, and lock behavior.
3. Exercise `/v1/chat/completions` and `/v1/models` with curl.
4. Output artifact: API behavior notes + edge-case list.

## Session 6: Reliability and Safety (90 min)
1. Study execution and network risk boundaries:
   - `nanobot/agent/tools/shell.py`
   - `nanobot/agent/tools/filesystem.py`
   - `nanobot/security/network.py`
2. Review retry logic and backoff in `nanobot/channels/manager.py`.
3. Output artifact: risk register (command execution, SSRF, message routing failures, stuck channels).

## Session 7: Test-Driven Understanding (120 min)
Use tests as learning index:
1. `tests/agent/` (runtime behavior)
2. `tests/channels/` (integration contracts)
3. `tests/tools/` (tool validation/security)
4. `tests/providers/` (provider behavior)

Run targeted suites:
```bash
pytest tests/agent -q
pytest tests/channels -q
pytest tests/tools -q
pytest tests/providers -q
```

Output artifact: gap list (areas lacking regression tests for your planned changes).

## 4. Backend-Focused Deep Dives
Prioritize these themes:
1. Concurrency model: session locks, pending queues, active tasks in `AgentLoop`.
2. Message durability assumptions: queue behavior and retry semantics.
3. Failure boundaries: provider timeout/retry policy and fallback content.
4. Multi-tenant concerns: unified vs per-session isolation.
5. Operational posture: gateway lifecycle, heartbeat/cron interactions, startup/shutdown semantics.

## 5. Learning Materials Index (Curated)
## A. Architecture and Runtime
1. `README.md`
2. `nanobot/agent/loop.py`
3. `nanobot/agent/runner.py`
4. `nanobot/agent/context.py`

## B. Extension Points
1. `docs/CHANNEL_PLUGIN_GUIDE.md`
2. `nanobot/channels/base.py`
3. `nanobot/agent/tools/base.py`
4. `nanobot/providers/base.py`

## C. Operations and Safety
1. `nanobot/config/schema.py`
2. `nanobot/security/network.py`
3. `nanobot/channels/manager.py`
4. `docs/WEBSOCKET.md`

## D. Product Surfaces
1. `nanobot/cli/commands.py`
2. `nanobot/api/server.py`
3. `nanobot/nanobot.py` (Python SDK facade)
4. `docs/PYTHON_SDK.md`

## E. Test Index (Executable Specs)
1. `tests/agent/`
2. `tests/channels/`
3. `tests/tools/`
4. `tests/providers/`
5. `tests/security/`

## 6. Practical Study Checklist
1. Can you explain the full request lifecycle without opening code?
2. Can you add one custom channel plugin and make `nanobot plugins list` detect it?
3. Can you trace how tool permissions are enforced for filesystem/shell/web?
4. Can you debug a failed outbound send path with retries?
5. Can you add one focused regression test in the subsystem you touched?

## 7. First 3 High-Leverage Hands-On Tasks
1. Add a tiny observability improvement: structured log fields for session/channel in a hot path.
2. Create a minimal channel plugin skeleton locally and validate discovery.
3. Add one defensive test around tool restriction or API request validation.

## 8. Personal Notes Template
Copy this block while studying:

```md
### Topic
- Source files:
- Runtime invariant:
- Failure mode:
- Current behavior:
- Desired behavior:
- Test coverage status:
- Follow-up patch idea:
```

## 9. What to Read If You Are Time-Constrained
If you only have 2-3 hours, read in this exact order:
1. `README.md` (Architecture + Config + API)
2. `nanobot/agent/loop.py`
3. `nanobot/cli/commands.py` (`agent`, `gateway`, `serve`)
4. `docs/CHANNEL_PLUGIN_GUIDE.md`
5. `tests/agent/test_runner.py` and `tests/channels/test_channel_plugins.py`

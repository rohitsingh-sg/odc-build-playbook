---
name: odc-build-playbook
description: Reliable workflow for building, publishing and deploying OutSystems Developer Cloud (ODC) apps through the remote `outsystems` MCP (Mentor, publish_status, env_app, deploy). Use for ANY ODC task driven by that MCP, or when Mentor runs stall, publish fails or is indeterminate, or auth is missing. For O11 / Service Studio MCP use `o11-build-playbook`. For design rules use `odc-best-practices`.
---

# ODC build playbook (outsystems MCP)

Built from the MyDemo8thOct Mentor build and other ODC sessions. Items marked **(unconfirmed)** are guesses. The live `tools/list` is the source of truth over any skill doc. Load `outsystems:outsystems` for tool detail.

## Benefits for ODC work
- Skips known failures: deprecated `mentor_publish`, missing `portfolioKey`, wrong `app_info` argument name.
- Prevents publishing an unfinished Mentor run, and concurrent publishes that can wedge an app.
- Chunked prompts keep Mentor runs smaller and easier to check.
- Honest status: what was applied, validated, warned, and not verified.

## 0. Pre-flight
1. `outsystems` authorized? If listed as needing authentication, stop and tell the user to authorize it (claude.ai connector settings or `/mcp`). Never ask for codes or tokens.
2. Confirm the target app/portfolio and environment before building.
3. Ask before anything that changes the tenant (publish, deploy, rollback, delete). An earlier "yes" does not cover a different action.

## 0b. Plan and chunk (do this before the first Mentor prompt)
Mentor's real size limit is unknown (not measured), so plan by size and adapt.
1. **Analyze the request** into pieces: entities, screens, logic/actions, integrations, roles, navigation.
2. **Order by dependency:** data, then logic, then screens, then navigation/home screen, then publish. Screens reference data; build data first.
3. **Chunk small to start:** one coherent unit per prompt (one entity group, or one screen with its widgets). Typically 2-4 chunks for a small app, more for a larger one.
4. **Show the user the chunk list** (numbered, with what each builds) before starting. Skip only for trivial one-chunk requests.
5. **Per chunk:** `mentor_prompt`, poll `mentor_get_run` to a terminal state, read the result (applied, errors, warnings), then continue.
6. **Adapt:**
   - Clean result: the next two small chunks may be merged.
   - Failure, stall or partial result: cancel with `mentor_cancel_prompt` if needed, halve the chunk, retry once.
   - Second failure on the same chunk: stop and report the exact result.
7. **Publish once at the end** (or at milestones the user picks), not after every chunk. Ask first.
8. **Keep a progress list** in the reply (done / in progress / pending) so a timeout midway can resume from the last finished chunk instead of restarting.
Example: "Claims portal" = (1) Claim and Employee entities, (2) Dashboard screen with static layout, (3) Claim list + detail screens bound to data, (4) Submit Claim form with validation, (5) roles/navigation, then publish.

## 1. Mentor workflow
Mentor is an async, multi-turn, server-side session.
1. `mentor_start_session`.
2. `mentor_create_asset` needs a **`portfolioKey`** (else `Portfolio ID is required`). Get it from an existing app: `app_list`, then `app_info` with arg **`key`** (not `app_key`).
3. `mentor_prompt`: one bounded request each, in 2-3 chunks (data, then screens, then logic). Never one giant prompt.
4. Poll `mentor_get_run` (`runId`, `cursor`). UI runs take minutes; use 30-60 s background waits, no tight loops.
5. Wait for a **terminal state** and read the result (applied? errors/warnings?) before the next prompt or publish.
6. **Publish:** `mentor_publish` is deprecated/removed. Use `mentor_prompt` with `"Publish"`, poll that run, then `publish_status` with the `publication_key`.
7. After success call `env_app` (`env_key`, `application_key`) and report the runtime URL.
8. `mentor_close_session` when done.

### UI prompts
Name the screen, data (static sample or entity), widgets (header, menu, cards, table, form), navigation and home screen. One screen or small group per prompt. State what to verify. Say what not to change.

## 2. Publish rules (from the server)
- Never re-publish while the previous publish is not terminal; two revisions building at once can crash the build engine (repeated `OS-DPL-*` errors; may need recovery in Studio).
- `failed` with an `OS-*` code is the build's real result: surface the code, don't blindly retry.
- `failed` with `indeterminate: true`: outcome unknown. Re-poll with the `publication_key` or check the deployed inventory. Do not re-publish.
- Read `publish_logs`, `deploy_messages`, `deploy_logs` before retrying. Check `deploy_impact` before changing shared/public elements.

## 3. Recovery and honesty
- Timeout or error: retry once, then stop and say exactly what was and wasn't applied.
- Report: what was done, validation counts (errors/warnings), what is pending, what you couldn't verify (e.g. app not opened in a browser, an unexplained Mentor warning, menu items without screens).
- Don't fill waiting time with unrelated research.

## 4. Checklist
- [ ] `outsystems` authorized; target app and portfolio known
- [ ] Chunked prompts; run finished and read before next step
- [ ] Publish via `mentor_prompt "Publish"` then `publish_status` then `env_app`; never concurrent
- [ ] Ask before tenant-changing actions
- [ ] Report limits honestly

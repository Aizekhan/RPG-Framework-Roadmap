# Project Status

## Source project
- Game repository: `Aizekhan/MythHunter`
- Roadmap repository: `Aizekhan/RPG-Framework-Roadmap`
- Master plan: `RPG_FRAMEWORK_MASTER_PLAN.md`
- Architecture baseline: `Audits/EPIC_02.8_Final_Architecture_Baseline.md`
- EPIC 04 candidate audit: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`

## Current position
- **EPIC 04 is COMPLETE** — the candidate audit is recorded at `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`. Decision: avoid a speculative empty `RPGFramework.Contracts` assembly.
- **Active task: EPIC 05.3 — explicit network event-bus selection and role-aware authorization.** PR #23 is merged: https://github.com/Aizekhan/MythHunter/pull/23; merge commit `8e8ad7b16f813dcdebc4b2d057616f75f73f2fe8`. The registration-order bug is fixed and user confirmed full project compile, Console and bootstrap clean. Unity for PR #23: **23 passed, 0 failed, 0 skipped**.
- A follow-up implementation is in progress on `feature/epic-05-3-network-eventbus-selection`; no source PR yet. The branch adds explicit Offline/Client/Server mode selection in GameBootstrapper, selected role instance binding to `INetworkSystem`, mode-aware authority policy and characterization tests. **Unity validation of this new branch is pending; do not merge it based on PR #23's old results.** Combined Host is deliberately not supported by this slice because current role systems prohibit simultaneous client/server operation.
- Handoff for future chats: [HANDOFF_NEXT_CHAT.md](HANDOFF_NEXT_CHAT.md). End-of-epic exit gates now require a committed handoff file.
- EPIC 05.2 — DI scope lifecycle complete: PR #21 and PR #22 merged; Unity EditMode 22/22 passed for PR #22, and full-project/bootstrap/scope smoke confirmed clean.
- Logging PR #20: https://github.com/Aizekhan/MythHunter/pull/20
- Logging merge commit: `ec967a2944ba83db69655d4415d416f49a2ce122`.
- CI on logging pre-merge head `0a0aa0ccc4d04e6bd4e413b21167153aed4bc1e7`: static boundary OK; 13 Framework/Test asset GUIDs checked; .NET tests 9 passed, 0 failed, 0 skipped; no C# compiler warnings.
- Unity CLI EditMode, Unity `6000.0.45f1`, XML `D:\\MythHunter-Git\\EPIC-05-1-Logging-EditMode.xml`: **10 passed, 0 failed, 0 skipped**. Logging contract test passed.
- User confirmed logging full-project compile/Console and bootstrap smoke clean; same logger instance is bound under both `IMythLogger` and `RPGFramework.Logging.ILogger`.
- Validation API audit found no external production callers of `IValidator<T>`/`Validator<T>`; no Framework validation extraction justified.
- DI source audit: `Audits/EPIC_05.2_DI_Candidate_Audit.md`. Characterization and scope lifecycle fixes are merged and validated.

## Completed roadmap work
### EPIC 03.5 — Standalone Framework ECS runtime
- PR #18 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/18
- Merge commit: `b6e046895938d2dd72026827c4f72fd44ced1fdf`
- Unity EditMode on source commit `3917e6a48344d4469fd95a2049b1a2e60d983223`: all 8 ECS tests passed; full EditMode run 9/9 passed.

### EPIC 03.6 — Migrate ECS consumers
- PR #19 merged to `dev`: https://github.com/Aizekhan/MythHunter/pull/19
- Merge commit: `d3b81484f42d084cae150b5b60fc64185cfb70c2`
- Unity EditMode on migration source `21d8d89938bc4c165aa485ae3abf96e003744cc1`, Unity `6000.0.45f1`: 9 passed, 0 failed, 0 skipped.
- User confirmed full-project compile/Console and game bootstrap/ECS smoke OK. Duplicate legacy ECS contracts/manager removed; game-owned world/system lifecycle remains in MythHunter.

### EPIC 04 — Core contract candidate audit
- Audit complete: `Audits/EPIC_04_Core_Contract_Candidate_Audit.md`.
- Decision: no speculative empty `RPGFramework.Contracts`; first implementation slice was neutral logging under EPIC 05.1.

### EPIC 05.1 — Logging and validation
- Logging contract implemented; PR #20 merged to `dev`: `ec967a2944ba83db69655d4415d416f49a2ce122`.
- Unity EditMode: 10 passed, 0 failed, 0 skipped; compile/bootstrap smoke confirmed.
- Generic validation extraction deferred by source audit.

### EPIC 05.2 — Dependency injection
- PR #21 merged: `f57cebb9e0dba11c3e6fbeefc8620627a9363428`.
- PR #22 scope lifecycle merged: `9c6f03675a85085fe5f713dfd9a9be871b5c1e59`.
- Unity 6000.0.45f1 EditMode for PR #22: **22 passed, 0 failed, 0 skipped**; user confirmed full-project compile/Console and bootstrap/DI-scope smoke.
- Async-flow scope propagation and transient/singleton disposal ownership remain deferred.

### EPIC 05.3 — Event dispatch (active)
- PR #23 merge commit: `8e8ad7b16f813dcdebc4b2d057616f75f73f2fe8`.
- Unity 6000.0.45f1 EditMode: **23 passed, 0 failed, 0 skipped**.
- GitHub Actions on source head `280882f1550cd3befcd58def29f76d08fd93b163` passed.
- Fix: register the final event bus before resolving `ISystemRegistry`, preventing the registry from capturing a cached `SimpleEventBus` singleton in the non-network configuration.
- Next: map the selected client/server `INetworkSystem` binding and composition-root policy. Do not extract event contracts until network selection and behavior semantics are specified.

## Rules
1. One active roadmap item at a time.
2. Do not reset, clean, or discard local worktree changes.
3. Do not modify `dev` directly; use pull requests.
4. Framework runtime/contracts must not depend on MythHunter, UnityEngine, UnityEditor, DI, logging sinks or providers.
5. Record actual validation evidence; successful .NET CI does not imply Unity assembly import/compile passed.
6. Avoid speculative empty modules and broad rewrites.

## Next action
Investigate the `INetworkSystem` composition-root gap with source evidence: trace client/server implementations, configuration/game mode selection, and current bootstrap registrations; propose a small explicit binding design and tests. Do not choose an arbitrary client/server binding or begin event contract extraction prematurely.

## Source of truth
- Master plan: milestone ordering and checklists.
- `STATUS.md`: current active task.
- `Tasks/`: scope and acceptance criteria.
- `Audits/`: source evidence and architectural decisions.

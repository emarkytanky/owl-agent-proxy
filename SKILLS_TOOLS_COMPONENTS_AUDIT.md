# OWL Agent Proxy — Skills, Tools & Components Audit (Pre-Code Gate)

**Mode:** 🦉 Depth (audit integrity) routed through 🐜 Systematic (sequenced gates)
**Operating instructions adopted:** `opencode-os/profiles/smp5.4pd.md` (v5.4 PD, Tactical Merge)
**Sequence executed:** Silent Layer → Mode → Discover → Brainstorm → Deep Research → Plan → Orchestrate → Validate → Review → Complete
**Evidence date:** 2026-11-17. Every claim below is tied to a command output or a URL. Nothing here is asserted from memory.
**Confidence:** ≥ 0.75 on all recorded findings. Items below 0.75 are marked ⚠️ UNVERIFIED and are not used as a basis for a recommendation.

---

## 1. Silent Layer

| Question | Answer |
|---|---|
| **Actual need** | Freeze the capability surface — which skills, tools and components exist, which are executable, which are stale, which are unsafe — before any further code is written on this repo. |
| **Blind spot** | "30 skills in `skills/`" reads as a rich capability set. It is actually **24 authored stubs + 6 symlinked third-party skills**, and of the 6, only a subset is executable in a credential-free environment. Directory count overstates capability. |
| **Irreducible truth** | A skill definition that cannot run is documentation, not capability. The audit must separate *defined* from *executable*, or the stack will be planned against phantom abilities. |

---

## 2. Component inventory (verified)

| Layer | Component | Evidence |
|---|---|---|
| Entry points | `proxy_defense.py`, `mcp_proxy_server.py`, `reverse_proxy.py`, `run.sh` | root listing |
| Gateway | `gateway/{app,auth,router,cache,billing,circuit_breaker,audit}.py` + `gateway/free_catalog/` | root listing of `gateway/` |
| Edge | `nginx.conf`, `Caddyfile`, `docker-compose.yml` | root listing |
| Config | `config.json`, `config/{AGENTS.md,config.json,opencode.json,*.template.json}` | root listing of `config/` |
| Tests | `tests/` — 571 collected per AGENTS.md (`pytest -q`) | AGENTS.md §Test Results |
| Scripts | `scripts/{verify-install.py, verify-skills-lock.py, gen-docs.py, gen-architecture.py, runtime-check.py, smoke-stealth.py, audit-verify.sh}` | directory listing |
| Skills | `skills/` (24 authored + 5 symlinks), `.agents/skills/` (5 vendored), `.claude/skills/` (5 vendored) | `find` + `ls -ld` |
| Provenance | `skills-lock.json` v2 (commit + sha256 per vendored skill) | file read |
| Aux | `tools/grill-engineering`, `docs/{SKILL_RESOLUTION.md,skills-framework/,agents/,architecture/}` | directory listing |

---

## 3. Skills inventory

### 3.1 Vendored third-party skills (locked, symlinked, hash-verified)

Layout is **one physical copy** in `.agents/skills/<name>/` with a relative symlink from `skills/<name>`; `.claude/skills/<name>` mirrors it. Confirmed by `ls -ld` on all six.

| Skill | Upstream | Commit (lock) | Executable here? | Blocking dependency |
|---|---|---|---|---|
| `parallel-deep-research` | `parallel-web/parallel-agent-skills` | `8fc1fc4` (2026-09-25) | ❌ | `parallel-cli` ≥ 0.3.0 + Parallel auth |
| `parallel-findall` | `parallel-web/parallel-agent-skills` | `8fc1fc4` (2026-09-25) | ❌ | `parallel-cli` ≥ 0.6.0 + Parallel auth |
| `github-research` | `lingzhi227/agent-research-skills` | `4e5f117` (2026-02-23) | ⚠️ partial | `gh` auth + 13 Python scripts at `~/.claude/skills/github-research/scripts/` that are **not vendored** |
| `free-search-aggregator` | `vulcanusalex/free-search-aggregator` | `72526e7` (2026-03-06) | ⚠️ | 12 provider keys optional; OpenClaw-oriented |
| `you-web` | `youdotcom-oss/agent-skills` | `a1f3172` (2026-09-30) | ⚠️ partial | You.com MCP; free profile is keyless `you-search` only |

### 3.2 Authored skill stubs (local, no lock entry, no external dependency)

By declared pipeline stage (`pipeline_stage` frontmatter):

- **DEFINE:** `agent-reach`, `agent-skills`, `codemap`, `duckduckgo-search`, `exa-web-search-free`, `meta-skill`, `research`
- **PLAN:** `brainstorm`, `caveman`, `chain-of-thought`, `compare-a-b`, `decision-tree`, `headroom`, `orchestrate`, `ralph-prompt-multi-task`, `simulator`
- **BUILD:** `code-generation`, `parallel-agent-skills`
- **VERIFY:** `audit-optimizer`, `explain-analyzer`
- **REVIEW:** `devils-advocate`, `impeccable`
- **Unstaged:** `owl-proxy`, `owl-proxy-mcp` (project-specific, correctly scoped)

These are prose contracts (handoff contract in / handoff to) rather than executable tooling. They cost context and no runtime.

---

## 4. Findings

| # | Finding | Evidence | Severity |
|---|---|---|---|
| **F1** | The requested `opencode-perplexity-research-agent` **does not exist** under that name. No GitHub, npm, or skills.sh match. | Independent research agent: exact-name search returned nothing | 🔴 blocks the named step |
| **F2** | Perplexity's real substitutable surface is the hosted MCP server `https://api.perplexity.ai/mcp` (`perplexity_research` tool) and `POST https://api.perplexity.ai/v1/agent`. | `docs.perplexity.ai/docs/getting-started/integrations/mcp-server`, `docs.perplexity.ai/docs/agent-api` | 🟡 substitution required |
| **F3** | `parallel-deep-research` and `parallel-findall` are **defined but not executable** without `parallel-cli` + Parallel credentials. Deep research in this environment must be fanned out differently. | Skill frontmatter `compatibility: Requires parallel-cli >= 0.3.0`; no credential present | 🟡 known limitation, not a bug |
| **F4** | `freeunlimited-websearch` carries **Snyk FAIL / CRITICAL (E005)** upstream, triggered by the `YOUR_USERNAME` clone URL in its install block. Verified the exact string is present in the vendored file. It is a docs placeholder, not a live malicious endpoint — but it is a hard audit fail on a skill the repo ships. | `skills.sh/.../freeUnlimited-websearch/security/snyk`; read of `.agents/skills/freeunlimited-websearch/SKILL.md` | 🔴 **REMEDIATED** — removed from `.agents/`, `.claude/`, `skills/` and the lock |
| **F5** | **Trigger collision:** `freeunlimited-websearch`, `free-search-aggregator` and `duckduckgo-search` all claim "free DuckDuckGo web search, no API key". Three skills competing for one trigger degrades selection accuracy (the agent picks by `description`). | Three `description` fields read and compared | 🟠 **REMEDIATED** — `duckduckgo-search` is now the sole keyless DDG trigger; `free-search-aggregator` defers to it |
| **F6** | `parallel-deep-research` Snyk verdict is **Warn (W011, MEDIUM)** — indirect prompt-injection exposure via fetched web content, risk 0.10. Exposure, not a compromise. `github-research`, `free-search-aggregator`, `you-web` also Warn. | skills.sh security pages | 🟡 operational control (treat fetched content as data) |
| **F7** | **Documentation drift.** `RESEARCH_SKILL_LAYERING.md` states "`parallel-findall` does not exist as a public skill page (404)" and "no `docker-compose.yml` in the repo". Both are now false: `parallel-findall` has 12.7K installs and is vendored here, and `docker-compose.yml` exists at root. | file read vs root listing | 🟠 audit docs are stale |
| **F8** | **Spec conformance.** Authored stubs put `pipeline_stage` / `source` at the **top level** of frontmatter. The Agent Skills spec allows only `name`, `description`/`license`/`compatibility`/`metadata`/`allowed-tools`; extras belong under `metadata`. Strict clients may skip or mis-rank these. | `agentskills.io/specification`; frontmatter dump of 24 stubs | 🟡 tolerable, worth fixing in bulk |
| **F9** | `skills/` count (30) overstates real capability. Only 6 are third-party capability and 0 of those 6 are fully executable without extra auth/binaries. | symlink resolution + dependency matrix §3.1 | 🟡 planning risk |

---

## 5. Validation evidence (commands actually run)

| Check | Command | Result |
|---|---|---|
| Symlinks resolve | `[ -e skills/<n>/SKILL.md ]` per link | 6/6 OK |
| Provenance gate | `python3 scripts/verify-skills-lock.py` | `offline: 6 of 6 vendored files match the lock` → `PASSED: 6 skill(s) verified`, exit 0 |
| Every `skills/` entry has `SKILL.md` | loop over `skills/*/` | 0 missing |
| Frontmatter present on all authored stubs | `sed -n '1,6p'` over `skills/*/SKILL.md` | 24/24 have `name` + `description` |

**Not run (and why):** `parallel-cli research run` — binary and credentials absent in this environment (F3). `pytest` — no code changed by this audit, so the suite is not the covering check. Perplexity MCP — no credentials.

---

## 6. Recommended sequence (Orchestrate stage)

```
G1 Discover  ✅ this document (inventory + findings F1–F9)
G2 Plan      → decide F4/F5 removals + F2 research substitution  [needs your call]
G3 Build     → only after G2: agent-stack changes
G4 Validate  → scripts/verify-skills-lock.py + skills count assertion
G5 Review    → devils-advocate on the stack, impeccable on the diff
G6 Complete  → terminate
```

The single blocking decision is the research substitution (F1/F2): the named agent does not exist, so **parallel-deep-research must be executed as (a) `parallel-cli` + Parallel key, (b) Perplexity MCP `perplexity_research`, or (c) fan-out of independent research agents** — which is what produced this document.

---

## 7. Assumptions

1. `docker-compose.yml` at root is intended and tracked (it contradicts the older layering doc, F7).
2. The six `.agents/skills` vendored files are the canonical copies and `skills/` symlinks are the intended single-copy layout.
3. "Executable" means runnable in a credential-free Linux sandbox with no vendor account.
4. Snyk verdicts are treated as review signals, not proof of malice.
5. The `github-research` scripts directory is intentionally not vendored (the skill documents `~/.claude/skills/github-research/scripts/`).

---

## ⚡ Next Step

F4/F5 are now closed (see Remediation log). The only open decision is **F2**: Perplexity MCP vs `parallel-cli` — pick one and the stack is frozen for implementation.

---

## 8. Remediation log (2026-11-17)

**F4 — `freeunlimited-websearch` removed.** Deleted from `.agents/skills/`, `.claude/skills/`, the `skills/` symlink, and the `skills-lock.json` entry. Live doc references stripped from `AGENTS.md`, `docs/skills-framework/{10-PIPELINES,README,QUICK_REFERENCE}.md` and the `DEFAULT_STAGE` map in `scripts/gen-docs.py`. Historical records (`docs/architecture/RESTRUCTURE_PLAN.md`, `REVIEW_PR39.md`, `CHECKER_AUDIT.md`, and the `FreeUnlimited-websearch` fixture string in `tests/test_skills_lock.py`) were deliberately **left untouched**: they describe past states or test case-normalisation generically, and editing them would falsify the record.

**F5 — one DuckDuckGo path.** `duckduckgo-search` is the sole skill claiming the keyless DuckDuckGo trigger. `free-search-aggregator`'s frontmatter was narrowed to multi-provider aggregation plus an explicit deferral to `duckduckgo-search`. That is a **frontmatter-only** edit, so its vendored body remains byte-identical to the pinned upstream commit and the lock records the change under `localModification`, exactly as the verifier's vocabulary requires.

| Before | After |
|---|---|
| 30 `skills/` entries · 6 vendored · 3 competing DDG triggers · 1 Snyk FAIL shipped | 29 entries · 5 vendored · 1 DDG trigger · 0 Snyk FAIL shipped |

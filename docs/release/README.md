# Release engineering

> **Status:** Stable. Updated when the release train cadence or
> environments change.
>
> **Governed by:** ADR-0010 (Coolify), Plan §35–§38 (CI/CD, deploy,
> environments), §40 (error budget policy).

This directory holds the **release engineering contract**. The skill
files describe how to *do* releases (`release-canary`,
`release-hotfix`, `add-feature-flag`); these docs describe *what*
the system commits to.

---

## 1. Environments

| Env | Purpose | URL pattern | Source |
| --- | --- | --- | --- |
| **Local** | Developer machine | `localhost` | working tree + `.env.local` |
| **Preview** | Per-PR ephemeral | `pr-<num>.preview.teskel.dev` | PR head; auto-tear-down on close |
| **Staging** | Mirror of prod, integration testing | `staging.teskel.dev` | `main` branch on green CI |
| **Canary** | 5–10% prod traffic, 1–4 h soak | `app.teskel.app` (5–10% of users) | release tag `vX.Y.Z` |
| **Production** | 100% traffic | `app.teskel.app` | release tag `vX.Y.Z` after canary promotion |

Per-tenant region pinning (Phase 6) is orthogonal to this — see ADR
when it lands.

---

## 2. Release train

Default cadence: **two scheduled releases per week** (Tue + Thu),
plus on-demand hotfixes (per `release-hotfix` skill).

```
Mon                 Tue                 Wed                 Thu                 Fri
─────────────────────────────────────────────────────────────────────────────────
                cut & deploy                          cut & deploy
                staging                                staging
                ↓ (8 h soak)                           ↓ (8 h soak)
                deploy canary 5%                       deploy canary 5%
                ↓ (1 h)                                ↓ (1 h)
                ramp 25% → 50% → 100%                  ramp 25% → 50% → 100%
                ↓                                      ↓
                tag, monitor 24 h                      tag, monitor 24 h
```

A release that fails canary is rolled back **automatically** (auto-
revert flag flips at -50% on any of: 5xx rate, p99, error budget
burn). Engineer triages within 30 minutes.

---

## 3. Versioning

We use **semver** for public-facing artifacts:

| Artifact | Versioning | Source of truth |
| --- | --- | --- |
| Public API | semver `vMAJOR.MINOR.PATCH`; URL prefix `/v1` | `apps/api/openapi.yml` + ADR for breaking changes |
| Public SDK (`@teskel/sdk`) | semver, follows API | `packages/sdk/package.json` |
| CLI (`@teskel/cli`) | semver, decoupled from API | `packages/cli/package.json` |
| Marketplace template manifest | own version field, see [`docs/marketplace/manifest-spec.md`](../marketplace/manifest-spec.md) | manifest |
| Internal packages | not externally versioned; bumped on each release | git tag |
| Plan | `vN.M` documented in `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md` Changelog | self |
| AGENTS.md | `vN.M.P` documented at footer | self |

**Public API breaking change:** requires a new major version
(`v2`), runs in parallel with `v1` for at least 12 months, and a
deprecation header on `v1`. Always an ADR.

---

## 4. Release checklist (per train)

Cut from `main` after a green CI run. Run by the **release captain**
(rotating role; see [`.agents/state/owners.md`](../../.agents/state/owners.md)).

1. Confirm error budget posture green/yellow (per
   [`slo-sli.md`](../observability/slo-sli.md) §6). If orange/red,
   skip this train unless the change reduces risk.
2. Review the diff since last tag — **no migrations** in train if
   any need exclusive locks; reschedule those for off-hours.
3. Tag `vX.Y.Z` on `main`.
4. Coolify auto-deploys to **staging**.
5. Run e2e suite against staging. Soak 8 h (one workday) **except**
   for hotfix.
6. Promote to **canary**: 5% traffic for 1 h.
7. Monitor canary dashboards
   (`/d/platform-overview`, `/d/api-golden`, `/d/llm-cost`,
   `/d/error-budget`). Auto-rollback rules on:
   - 5xx rate increase > 50% over baseline.
   - p99 latency increase > 50% over baseline.
   - error-budget burn ≥ 14× over 5m.
   - Sentry "release health" crash-free sessions < 99.5%.
8. Ramp 25% → 50% → 100% with 30 min between steps if all green.
9. Post `#deploys` summary: tag, diff link, who, when.
10. Watch for 24 h; cancel any non-critical change requests.

The full step-level checklist is in
[`release-canary` skill](../../.agents/skills/release-canary/SKILL.md).

---

## 5. Hotfix path

Hotfixes bypass the train but **not** the controls. See
[`release-hotfix` skill](../../.agents/skills/release-hotfix/SKILL.md).
Summary:

- One small, focused commit cut from the prod tag, **not** from main.
- No DB migrations, no refactors, no flag changes.
- Canary skip is permitted only when the bug is more dangerous than
  the canary skip.
- Reconciled back into `main` within 24 h.
- Two hotfixes in 7 days = release-process bug → retro.

---

## 6. Feature flags & dark launches

All risky changes go behind a PostHog flag. See
[`add-feature-flag` skill](../../.agents/skills/add-feature-flag/SKILL.md).

| Rollout | Window | Owner |
| --- | --- | --- |
| internal-only | 1–7 days | feature owner |
| beta tenants | 1–4 weeks | feature owner + PM |
| 1% / 10% / 50% / 100% | 1–4 weeks | feature owner |
| flag removal | within **30 days** of 100% | feature owner |

A flag older than 30 days at 100% is a bug — automation files a
story to remove it.

---

## 7. Database migrations

- Use Drizzle migrations only.
- **Expand-then-contract** for schema changes that touch hot tables:
  add nullable column → backfill → switch reads → make NOT NULL →
  drop old. Each step is a separate PR.
- No exclusive locks during business hours. If a migration needs
  one, schedule a maintenance window and announce 48 h in advance.
- Roll-forward, not roll-back, by default. To "undo" a migration,
  write a new migration that reverses it.
- Backfills for ≥10k rows go through `data-backfill-job` skill, not
  inline migrations.

---

## 8. Rollback playbook

| Scenario | Action |
| --- | --- |
| Canary regression | Auto-rollback (canary controller flips traffic back, tag stays). |
| Post-100% regression, additive change | Disable feature flag; the bad code path is dormant. |
| Post-100% regression, structural change | Deploy previous tag immediately; investigate; **only then** decide on roll-forward. |
| Migration bad outcome | Roll forward with a corrective migration; restore from PITR only on data corruption (per `db-restore-pitr` skill). |
| Auth / payments regression | Page Sev1; commander decides between auto-rollback, hotfix, or feature flag. |

---

## 9. Comms

| Channel | When |
| --- | --- |
| `#deploys` | Every train tag + canary stage transition + final 100%. |
| `#incidents` | Any rollback, any auto-revert, any Sev2/Sev1. |
| Status page | Customer-impacting incidents only (Sev1 always; Sev2 if external impact > 5 min). |
| Customer email | Sev1 with personal data implications **and** for major API deprecations. |

---

## 10. References

- [`slo-sli.md`](../observability/slo-sli.md) — what counts as a regression.
- `release-canary`, `release-hotfix`, `add-feature-flag` skills.
- ADR-0010 (Coolify) and ADR-0010-future (multi-region, Phase 6).
- Plan §35–§38 + §40.

# Security Policy

We take TESKEL's security seriously — multi-tenant data, customer
funds, and AI workloads all flow through this platform. This page
covers responsible disclosure, our coverage, and what to expect.

## Reporting a vulnerability

- **Email:** `security@teskel.dev`
- **PGP key:** [`/.well-known/pgp.asc`](https://teskel.dev/.well-known/pgp.asc)
  (fingerprint published when Phase 0 ships)
- **GitHub Security Advisories:** [Open a private advisory](https://github.com/yahyaxbt/teskel/security/advisories/new)
  if you're reading this on GitHub.

**Do NOT:**

- Open a public issue.
- Post details in PR comments.
- Test on production without prior coordination.
- Run automated scanners against production endpoints.

## What we want to know

When reporting, please include:

1. A clear description of the issue and impact.
2. Reproduction steps (or proof-of-concept).
3. Affected component / endpoint / version.
4. CVSS estimate if you have one.
5. Whether you've attempted to access data of users other than
   yourself; if so, what you found, and confirmation that you did
   not retain it.

We commit to:

- Acknowledge receipt **within 2 business days**.
- Provide a triage result **within 5 business days**.
- Keep you updated on remediation cadence.
- Credit you publicly (if you want) once the fix is shipped.

## Scope

In scope:

- TESKEL-operated services on `*.teskel.dev` and `*.teskel.app`.
- The TESKEL CLI and SDK packages.
- TESKEL workflow runtime, marketplace, and visual builder.
- Self-hosted Helm chart (Phase 6+).

Out of scope (please report to the third party):

- Stripe, Resend, OpenRouter, Coolify, Cloudflare, AWS, GitHub —
  these are upstream vendors. Their security policies apply.
- Customer-deployed templates (the marketplace creator is
  responsible for their own template's security).
- DDoS / volumetric attacks (handled by Cloudflare WAF and not
  considered vulnerabilities).
- Social engineering of TESKEL employees.

## What's NOT a vulnerability (by itself)

To save your time and ours:

- Missing security headers without a demonstrated impact.
- Self-XSS that requires the user paste hostile input into their
  own console.
- Rate-limit bypasses where the underlying action is rate-limited
  by another layer (DB, vendor, etc.).
- Missing rate-limit on cosmetic endpoints (`/healthz`, marketing
  pages).
- Output of `nmap` / similar scanners without a corresponding
  exploit.
- Vulnerabilities in dependencies that don't affect TESKEL's code
  paths — please file with the dependency upstream.
- Issues in frameworks themselves (Next.js, Hono, Drizzle) — file
  with the framework.
- Known CVEs we're already tracking (we publish status in
  `docs/security/cves.md` once Phase 1 ships).

## Severity & response targets

| Severity | Description | Acknowledge | Mitigate | Fix in prod |
| --- | --- | --- | --- | --- |
| **Critical** | Active exploitation, data exfil, RCE, auth bypass affecting many tenants | 1 hour | 24 hours | 72 hours |
| **High** | Cross-tenant data access, persistent XSS in admin areas, secrets exposure | 1 business day | 7 days | 14 days |
| **Medium** | XSS in user content, info-disclosure of metadata, CSRF on non-critical endpoints | 5 business days | 30 days | Next quarterly release |
| **Low** | Best-practice deviations, missing headers without impact, rate-limit edge cases | 5 business days | Next train | Next quarterly release |

Critical reports trigger a Sev1 incident — see
[`incident-sev1`](./.agents/skills/incident-sev1/SKILL.md).

## Safe-harbor

If you make a good-faith effort to comply with this policy:

- We won't take legal action against you.
- We'll work with you, not against you.
- We'll credit you if you want.

A "good-faith effort" includes:

- Avoiding privacy violations, data destruction, and service
  disruption.
- Not running automated scanners against production without prior
  coordination.
- Reporting privately and in a reasonable time.
- Not exfiltrating data beyond what's necessary to demonstrate the
  vulnerability.
- Deleting any test data once the issue is reported.

## Bounty

We don't run a paid bounty program yet — we may add one in Phase 5
(see [Plan §53](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#53-compliance-roadmap)).
Until then, we offer:

- Public credit (Hall of Fame at `https://teskel.dev/security/credits`).
- TESKEL swag.
- For severe issues, a discretionary cash reward.

## Compliance program

TESKEL's compliance roadmap is documented in the plan:

- **Phase 1+:** GDPR, UK-GDPR, PDP Indonesia readiness.
- **Phase 4:** SOC 2 Type 1.
- **Phase 5:** SOC 2 Type 2, ISO 27001, HIPAA + BAA, EU AI Act
  mapping.
- **Phase 6:** SOC 2 Type 2 renewal, FedRAMP investigation (TBD).

Customer-facing compliance posture (DPAs, certifications, sub-
processor list) lives at `https://teskel.dev/legal/`.

## Internal practices

These pages document how we operate:

- [`docs/security/secrets.md`](./docs/security/secrets.md) — secrets
  registry + rotation policy.
- [`docs/security/rbac-matrix.md`](./docs/security/rbac-matrix.md) —
  permissions × roles map.
- [`docs/runbooks/security/`](./docs/runbooks/) — incident runbooks
  for security alerts.
- [`docs/incidents/`](./docs/incidents/) — public-redacted post-
  mortems for security incidents.
- [`.agents/skills/rotate-secret`](./.agents/skills/rotate-secret/SKILL.md)
  — how secrets get rotated.
- [`.agents/skills/gdpr-data-request`](./.agents/skills/gdpr-data-request/SKILL.md)
  — DSAR handling.

## Questions

For non-security questions about TESKEL, see
[`README.md`](./README.md) and [`CONTRIBUTING.md`](./CONTRIBUTING.md).

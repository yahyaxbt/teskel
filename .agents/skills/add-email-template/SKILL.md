---
name: add-email-template
description: Add a transactional email (Resend + react-email) — typed payload, MJML/react-email source, plaintext fallback, tenant branding, locale support, unsubscribe/CAN-SPAM compliance, render snapshot test, and observability. Use for any email TESKEL sends to a human.
---

# add-email-template — add a transactional email

> Plan ref: [Sec. 30 (Integrations)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#30-integrations--external-apis),
> [Sec. 60 (Notifications & onboarding)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#60-onboarding--lifecycle),
> [Sec. 49 (PII / privacy)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#49-pii-and-privacy).

> **Hard rule:** all email goes through `packages/notifications/email`
> + Resend. Never `nodemailer` or vendor SDKs ad-hoc. Every send is
> queued (outbox) — handlers do not `await resend.send()`.

## Decide the email type

| Type | Examples | Notes |
| --- | --- | --- |
| **Transactional** | Verify email, magic link, invoice, receipt, password reset, security alert | Default; no consent required |
| **Lifecycle** | Welcome, day-7 nudge, trial ending | Requires consent (`marketing_emails_opt_in`) |
| **Operational** | Workflow run failed, KB indexing done | Triggered by user action |
| **Digest** | Weekly summary | Aggregated, opt-out by default |

Marketing/lifecycle emails MUST be opt-in (CASL/EU/US best practice).
Transactional + operational emails sent regardless, but with
unsubscribe link to that **category**.

## File layout

```
packages/notifications/
  email/
    templates/
      verify-email/
        template.tsx       # react-email JSX
        copy.en.json       # subject + body strings
        copy.id.json
        meta.ts            # id, category, schema, allowedRoles
        template.test.tsx  # snapshot + a11y test
      invoice-paid/
        ...
    render.ts              # render JSX + MJML → HTML + text
    send.ts                # queue producer
    types.ts               # TemplateId union, payload registry
  index.ts
```

## Steps

1. **Pick the template id.** kebab-case, scoped:
   `verify-email`, `invoice-paid`, `workflow-run-failed`.

2. **Define meta + schema.**

   ```ts
   // packages/notifications/email/templates/verify-email/meta.ts
   import { z } from 'zod';

   export const VerifyEmailPayload = z.object({
     userId: z.string(),
     orgId: z.string().nullable(),
     verifyUrl: z.string().url(),
     expiresAt: z.string().datetime(),
   });

   export const meta = {
     id: 'verify-email' as const,
     category: 'transactional' as const,
     audience: 'individual' as const,
     locales: ['en', 'id'] as const,
     fromAddress: 'auth@notifications.teskel.dev',
     replyTo: null,
     payloadSchema: VerifyEmailPayload,
   };
   ```

3. **Write the JSX (react-email).**

   ```tsx
   // packages/notifications/email/templates/verify-email/template.tsx
   import { Html, Head, Body, Container, Heading, Text, Button } from '@react-email/components';
   import type { z } from 'zod';
   import type { VerifyEmailPayload } from './meta';
   import en from './copy.en.json';
   import id from './copy.id.json';

   export function VerifyEmail(props: z.infer<typeof VerifyEmailPayload> & { locale: 'en' | 'id' }) {
     const t = props.locale === 'id' ? id : en;
     return (
       <Html lang={props.locale}>
         <Head><title>{t.subject}</title></Head>
         <Body style={{ fontFamily: 'system-ui, sans-serif', backgroundColor: '#f6f8fa' }}>
           <Container style={{ maxWidth: 560, padding: 24 }}>
             <Heading as="h1">{t.heading}</Heading>
             <Text>{t.body_lead}</Text>
             <Button href={props.verifyUrl} style={{ background: '#111', color: '#fff', padding: '12px 18px', borderRadius: 8 }}>
               {t.cta}
             </Button>
             <Text style={{ color: '#666', fontSize: 12 }}>{t.footer}</Text>
           </Container>
         </Body>
       </Html>
     );
   }
   ```

4. **Plaintext fallback.** Always send `multipart/alternative`. Auto-
   generate plaintext from MJML/react-email; verify it's coherent.

   ```ts
   // render.ts
   import { renderAsync } from '@react-email/render';
   const html = await renderAsync(<VerifyEmail {...payload} locale={locale} />);
   const text = await renderAsync(<VerifyEmail {...payload} locale={locale} />, { plainText: true });
   ```

5. **Queue the send (do NOT inline).**

   ```ts
   // packages/notifications/email/send.ts
   import { queue } from '@teskel/queue';
   import { meta as verifyMeta } from './templates/verify-email/meta';

   export async function sendVerifyEmail(payload: z.infer<typeof verifyMeta.payloadSchema>) {
     verifyMeta.payloadSchema.parse(payload);
     await queue.add('email.send', { id: 'verify-email', payload }, {
       attempts: 5,
       backoff: { type: 'exponential', delay: 1_000 },
       removeOnComplete: { age: 86_400 },
     });
   }
   ```

6. **Email worker.** Single worker handles all template ids. Looks up
   payload schema → renders → calls Resend.

   ```ts
   // packages/jobs/workers/email.ts
   import { worker } from '@teskel/queue';
   import { templates } from '@teskel/notifications/email';
   import { resend } from '@teskel/integrations/resend';
   import { recordEmailEvent } from '@teskel/notifications/audit';

   worker('email.send', async (job) => {
     const tpl = templates[job.data.id];
     if (!tpl) throw new Error(`unknown_template:${job.data.id}`);
     const payload = tpl.meta.payloadSchema.parse(job.data.payload);
     const locale = await resolveLocale(payload);
     const { html, text, subject } = await tpl.render(payload, locale);

     const { id } = await resend.send({
       from: tpl.meta.fromAddress,
       to: payload.email,
       subject,
       html, text,
       headers: {
         'List-Unsubscribe': `<mailto:unsubscribe@teskel.dev?subject=${tpl.meta.id}>, <https://app.teskel.dev/u?t=${unsubToken(payload)}>`,
         'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
         'X-Entity-Ref-ID': job.id,
       },
     });

     await recordEmailEvent({
       templateId: tpl.meta.id,
       resendMessageId: id,
       to: hash(payload.email),
       userId: payload.userId,
       orgId: payload.orgId,
       category: tpl.meta.category,
     });
   });
   ```

7. **Compliance headers (always set).**
   - `List-Unsubscribe` (RFC 8058 one-click).
   - `List-Unsubscribe-Post: List-Unsubscribe=One-Click`.
   - Physical mailing address in footer (CAN-SPAM).
   - For lifecycle/marketing: visible unsubscribe link in body.

8. **Tenant branding (paid plans).**
   - `apps/api` resolves brand by `orgId` → `{ logoUrl, primaryColor, fromName }`.
   - Render uses brand for header/CTA color.
   - From address stays on TESKEL-controlled domain unless DKIM-verified
     custom domain configured via `add-integration` (vendor: Resend).

9. **Locales.** Resolve in order: `user.locale` → `org.defaultLocale`
   → `'en'`. Each `copy.<locale>.json` must have all keys; CI lints
   for missing keys.

10. **Unsubscribe / preferences.** All categories must be
    independently opt-out-able via signed token URL:
    - `notification_preferences (user_id, category, opted_out_at)`.
    - Worker reads prefs before send; skips for opted-out users (still
      logs `skipped` outcome for visibility).
    - One-click unsubscribe from List-Unsubscribe header writes the
      pref row directly.

11. **Idempotency.** Job key encodes `(template_id, recipient_id,
    deduplicate_key)`. Example: `verify-email:user_123:verifyTokenHash`.
    Re-queueing the same key within 1h is a no-op.

12. **Render tests + snapshots.**
    - Snapshot HTML output per locale (golden file under
      `__snapshots__/`).
    - Run `axe-core` against rendered HTML; assert no critical
      issues (color-contrast, alt text, document language).
    - Manual visual check via `pnpm email:dev` (react-email preview).

13. **Spam-score check.** CI runs `mailtrap` or `mail-tester` against
    a sample render and fails if score <8/10. Avoid:
    - Words: `free`, `winner`, ALL-CAPS, exclamation marks.
    - Single huge image, no text.
    - Mismatched From and Reply-To.

14. **Observability.**
    - `email_sent_total{template_id, locale, outcome}`.
    - `email_render_duration_seconds`.
    - Webhook from Resend → bounce/complaint/delivered events
      (handle via `add-webhook-receiver`).
    - Alert: bounce rate >2% over 1h → page (deliverability risk).
    - Alert: complaint rate >0.1% → page.

15. **Hard limits.**
    - DKIM, SPF, DMARC must be configured for `notifications.teskel.dev`.
    - Custom-domain emails require DKIM verified before send.
    - No more than 1 high-priority email per user per 30 min, except
      truly time-critical (security alert).

## Pitfalls

- Sending HTML-only without text part → spam folder.
- Forgetting `List-Unsubscribe` → Gmail/Yahoo block in 2024+.
- Inlining vendor SDK call in request handler → request blocks on
  vendor latency.
- Adding new copy without updating all locales → fallback returns
  `[missing key]` in production.
- Hard-coded recipient list for ops alerts → use a notification
  channel/group, not a list of personal emails.
- Sending lifecycle email without consent → CASL/CAN-SPAM violation.
- Including PII (full names, account numbers) in subject lines —
  visible in notifications and inbox previews.
- Using a free address (`@gmail.com`) as From — DMARC will reject.

## Done when

- Template file + meta + payload schema in place.
- All locales filled.
- Worker renders + sends + records audit event.
- Compliance headers set (List-Unsubscribe, etc.).
- Render snapshot + axe + spam-score tests pass.
- Bounce/complaint webhook handler updated if new template category.
- Dashboard + alerts wired.
- DKIM/SPF/DMARC verified for sending domain.
- Manual preview rendered correctly in `pnpm email:dev`.

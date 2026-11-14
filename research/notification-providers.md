# Which free-tier third-party notification providers fit Cadence?

Researched 2026-11-15. All citations are primary sources (official pricing pages, official docs, official GitHub repos), and every one was checked 2026-11-15. Where a page could not be fetched or a fact could not be verified, the text says so.

## Question and constraints

Cadence is a React Router v7 (framework mode), Node, SQLite/Drizzle video-course app with **Students** and **Instructors**. We need a third-party notification provider:

- **Hard constraints:** third-party (no build-our-own); the free tier covers dev and demo scale with **no credit card**. A second free, no-card email service (Resend, Brevo, and so on) alongside it is acceptable.
- **Needs:** an in-app feed plus at least one out-of-app channel (email at minimum). It must handle **event-driven** sends ("your question got a reply") and **time-driven** sends (inactivity nudges, digests, delays, scheduled sends).

## Summary comparison

| Provider | Free tier, no card? | Free limit | React in-app feed | Email | Delay on free | Digest on free | Scheduled / recurring on free | Node SDK | User sync | Lock-in |
|---|---|---|---|---|---|---|---|---|---|---|
| **Knock** | Yes ("no credit card required") | 10,000 messages/mo (per user per channel); no hard cap | Yes, `@knocklabs/react` | Bring your own (Resend, Postmark, SES, Mailgun, SendGrid, SMTP...; not Brevo). "Knock (testing only)" provider for dev | Yes, up to 365 days (no plan restriction in docs) | Yes (batch, up to 90 days) | Yes ("Scheduled workflows": recurring and one-time, recipient timezone) | `@knocklabs/node` (Apache-2.0) | Inline identify at trigger time, or identify ahead of time | Proprietary SaaS, no self-host; workflows can be pulled and pushed as files via CLI |
| **Novu** | Yes ("No credit card required") | 10,000 workflow runs/mo, **5 workflows**, 2 envs | Yes, `@novu/react` `<Inbox />` | Bring your own (Resend, Brevo, Postmark, SES, Mailgun, SendGrid, SMTP...). Demo email integration for testing | Yes, but **max 1 day** on Free | Yes, but **max 1 day** on Free | Scheduled delay exists, capped at 1 day; no recurring cron schedule found | `@novu/api` | Subscriber created on first Inbox session; `to` accepts an inline object | **MIT core, self-hostable** community edition; code-first `@novu/framework` |
| **Courier** | **Not stated** on pricing page | 10,000 sends/mo (one send = one template to one recipient, all channels) | Yes, `@trycourier/courier-react` | Bring your own (Resend, Postmark, SES, Mailgun, SendGrid, SMTP...; not Brevo). Test provider sends only to signup address | Journeys delay node (90 days per node in one doc, 93 in another) | **Pricing page lists "Digest & batching" under Enterprise only** | Delay-until timestamp; Journeys on Free | `@trycourier/courier` | Inline `to: { email }` works | Proprietary SaaS, no self-host |
| **SuprSend** | Yes ("No Credit Card required") | 10,000 notifications/mo | Yes, `@suprsend/react` (SuprSend branding on Free) | Bring your own ("you will need downstream messaging providers") | Delay node documented; pricing table has no delay row (unverified for Free) | **No** ("Batching" and "Digest" show "—" on Free) | Timezone and work-hours sending **not on Free** | `@suprsend/node-sdk` | Inline recipients upserted at trigger time | Proprietary SaaS |
| **MagicBell** | **Not stated** | **1,000 deliveries/mo** (in-app and email each count) | Yes, `@magicbell/react` | Bring your own (SES, Mailgun, SendGrid, SMTP, Ping Email) | Workflow `wait` step (in seconds) | Not documented | Not documented (no calendar scheduling) | Browser/Node client | Not verified | Proprietary SaaS |
| **Engagespot** | **Unverifiable**: marketing site returns "Site Not Found" | Unverifiable | Docs exist | Docs mention a default provider | Docs list a Delay Function | Docs list a Batch Function | Unverifiable | `@engagespot/node` | `identifier` field | GitHub org inactive since Jan 2025. **Excluded** |
| **Pingram** (formerly NotificationAPI; extra candidate) | Yes ("no credit card required") | 3,000 emails/in-app + 100 SMS per month | Yes, `@notificationapi/react` popup | **Built in** (own SMTP), but external recipients need a verified domain | One-off `schedule` (ISO time) per send; update and delete supported | Per-channel delivery option: hourly/daily/weekly/monthly (Free availability not stated) | One-off schedule only; no recurring or workflow engine found | `pingram` | `to.id` (+ inline `email`) | Proprietary SaaS |

## Per-provider findings

### Knock

1. **Free tier:** Free costs $0/month, covers "Up to 10,000" messages sent, and starts "for free, no credit card required." A message is "a single message delivered to a single user on a single channel": an in-app message plus an email is two. "We don't hard cap your messages"; Knock contacts you about adding billing if you go over. Excluded on Free: email client previews, translations, removing Knock branding, and per-tenant features. Source: https://knock.app/pricing (checked 2026-11-15).
2. **React feed:** `@knocklabs/react` provides `KnockProvider`, `KnockFeedProvider`, `NotificationIconButton`, `NotificationFeedPopover` and `NotificationFeed`. By default a feed is readable by anyone who has the feed ID. For production, turn on enhanced security mode and pass a signed `userToken`. Source: https://docs.knock.app/in-app-ui/react/feed (checked 2026-11-15). The pricing page lists in-app feeds and pre-built components on Free (https://knock.app/pricing, checked 2026-11-15).
3. **Email:** Knock does not send email itself; it sends through downstream providers. Supported: Amazon SES, Cloudflare Email, MailerSend, Mailgun, Mailjet, Mailtrap, Mandrill, Postmark, **Resend**, SendGrid, SMTP, SparkPost. **Brevo is not listed.** A "Knock (testing only)" provider exists for development. Source: https://docs.knock.app/integrations/email/overview (checked 2026-11-15).
4. **Delays, digests, schedules:** The delay function has four modes: fixed, dynamic, relative to a timestamp, and time window (such as business hours in the recipient's timezone). Delays can last up to 365 days, and delayed runs can be cancelled through the API. The docs state no plan restriction. Source: https://docs.knock.app/designing-workflows/delay-function (checked 2026-11-15). The batch (digest) function supports fixed, dynamic, relative and sliding windows, up to 90 days. The only plan restriction is rendering more than 10 activities (Enterprise). Source: https://docs.knock.app/designing-workflows/batch-function (checked 2026-11-15). Schedules are "a managed, recipient-timezone-aware cron job", recurring or one-time, created with `client.schedules.create`. Source: https://docs.knock.app/concepts/schedules (checked 2026-11-15). The pricing page lists "Batching" and "Scheduled workflows" as included on Free (https://knock.app/pricing, checked 2026-11-15).
5. **Node SDK:** `@knocklabs/node` (Apache-2.0), using `client.workflows.trigger('key', { recipients, data })`. Source: https://github.com/knocklabs/knock-node (checked 2026-11-15).
6. **User identity:** Users can be identified "ahead of time, or lazily via inline-identification in your workflow triggers." Pre-sync is not required. Source: https://docs.knock.app/concepts/recipients (checked 2026-11-15).
7. **Lock-in:** The CLI pulls and pushes workflows and templates to a local directory, so they can be versioned in git. Source: https://docs.knock.app/developer-tools/knock-cli (checked 2026-11-15). The file format is not stated on that page (unverified). I found no self-host option. Workflow definitions are Knock-specific.

### Novu

1. **Free tier:** Free costs $0/month with 10,000 workflow runs, "Up to 5 workflows", 2 environments and 3 team members. The page says "No credit card required." Source: https://novu.co/pricing (checked 2026-11-15). The pricing comparison table did not extract cleanly. The limits doc is authoritative: Workflows 5, **Delay duration 1 day**, **Digest duration 1 day**, Throttle 1 hr, Inbox feed retention 30 days, Activity feed retention 1 day, Layouts 1. Source: https://docs.novu.co/platform/developer/limits (checked 2026-11-15).
2. **React feed:** `@novu/react` provides `<Inbox />` and needs React 18 or later. You pass `applicationIdentifier` and `subscriberId`. "New subscribers are created on the first session." HMAC (`subscriberHash`) is used for production security. Source: https://docs.novu.co/platform/inbox/overview (checked 2026-11-15). I found no plan restriction on the Inbox itself; the pricing page implies Novu branding stays on Free (removing it is a Pro feature).
3. **Email:** Bring your own: SendGrid, Amazon SES, **Resend**, Postmark, **Brevo**, Mailgun, Mailjet, SparkPost, Mandrill, MailerSend, Mailtrap, Plunk, custom SMTP and others. A "demo email integration" exists for testing; its limits are not documented on that page (unverified). Source: https://docs.novu.co/platform/integrations/email (checked 2026-11-15).
4. **Delays, digests, schedules:** Delay modes are fixed, scheduled (a time in the subscriber's timezone) and dynamic (from the payload). The Free plan allows "1 day on Free, 7 days on Pro, and 30 days on Team". Source: https://docs.novu.co/platform/workflow/add-and-configure-steps/action-steps/delay (checked 2026-11-15). The digest cap is also 1 day on Free (https://docs.novu.co/platform/developer/limits, checked 2026-11-15). **So on Free, a "nudge after 7 days inactive" or a weekly digest cannot be built inside Novu.** Cadence would have to trigger at the right moment itself. I found no recurring cron-style schedule feature comparable to Knock Schedules.
5. **Node SDK:** `@novu/api`, using `novu.trigger({ workflowId, to, payload })`. `to` accepts a subscriber object (`{ subscriberId, email }`). The page does not explicitly say that an inline `to` creates a missing subscriber (unverified). Source: https://docs.novu.co/api-reference/events/trigger-event (checked 2026-11-15).
6. **User identity:** Subscribers keyed by your own user ID; auto-created on first Inbox session (see 2).
7. **Lock-in:** **Lowest of the group.** The core is MIT, and the `enterprise` folder is under a commercial license. The repo has about 40.3k stars and was pushed 2026-11-14. The latest release is `@novu/framework@v2.14.0` (2026-09-28). Sources: https://github.com/novuhq/novu and the GitHub API (checked 2026-11-15). A community self-hosted edition exists, but "some features exclusive to Novu's cloud-managed solution will not be available". Source: https://docs.novu.co/community/self-hosting-novu/overview (checked 2026-11-15). Whether the 1-day delay cap applies when self-hosted is not stated (unverified). Workflows can also be written in TypeScript with `@novu/framework`. Source: https://docs.novu.co/framework/introduction (checked 2026-11-15).

### Courier

1. **Free tier:** The Developer plan is "Free", "up to 10k / mo." "One send is one template sent to one recipient", and multi-channel still counts as one send. Test-environment sends count the same as production. **The pricing page does not say whether a card is required for Developer.** It only says "Add a credit card to move to Courier's Business plan." **"Digest & batching" appears only in the Enterprise feature list.** Source: https://www.courier.com/pricing (checked 2026-11-15). A no-card claim appeared only in search snippets of Courier marketing pages, which I could not verify on the pricing page.
2. **React feed:** `@trycourier/courier-react` (v9, React 18+) provides `<CourierInbox />`. Auth uses a backend-minted JWT and `courier.shared.signIn({ userId, jwt })`. Source: https://www.courier.com/docs/platform/inbox/inbox-overview (checked 2026-11-15).
3. **Email:** Bring your own: Amazon SES, SendGrid, Postmark, Mailgun, **Resend**, SMTP, Mandrill, SparkPost, Mailjet, MailerSend, Amply, Gmail, OneSignal. **Brevo is not listed.** Source: https://www.courier.com/docs/integrations/email/overview (checked 2026-11-15). The Courier Test Email Provider ships in Test environments and delivers only to your signup address. Source: https://www.courier.com/docs/getting-started/quickstart (checked 2026-11-15).
4. **Delays, digests, schedules:** Journeys have `send`, `delay`, `branch`, `fetch`, `throttle`, `batch`, `add-to-digest` and `exit` nodes. The delay limit is "90 days or less" per node, which conflicts with 93 days in other Courier docs surfaced by search. Source: https://www.courier.com/docs/journeys/overview (checked 2026-11-15). Digest docs exist (https://www.courier.com/docs/journeys/automations/digest, checked 2026-11-15) and state no plan restriction. That **conflicts with** the pricing page listing digest and batching under Enterprise, so treat digest on Free as unavailable until confirmed.
5. **Node SDK:** `@trycourier/courier` (v7+), using `client.send.message({ message: { to: { email }, ... } })`. Source: https://www.courier.com/docs/getting-started/quickstart (checked 2026-11-15).
6. **User identity:** Inline `to` with an email or user ID works; no pre-created profile is required (same source).
7. **Lock-in:** Proprietary SaaS. I found no self-host or open-source server.

### SuprSend

1. **Free tier:** "$0/month", "10k Notifications/mo", "No Credit Card required". Log retention is 30 days. Source: https://www.suprsend.com/pricing (checked 2026-11-15).
2. **React feed:** `@suprsend/react` (the older `@suprsend/react-inbox` has reached end of support), with drop-in or headless options and JWT user tokens. Source: https://docs.suprsend.com/docs/inbox-react (checked 2026-11-15). On Free, the "In-app notification feed (with SuprSend branding)" (https://www.suprsend.com/pricing, checked 2026-11-15).
3. **Email:** Bring your own. The FAQ says "For SMS and Email, you will need downstream messaging providers." Source: https://www.suprsend.com/pricing (checked 2026-11-15). I did not fetch the vendor list.
4. **Delays, digests, schedules:** Delay node: fixed, dynamic and relative. The docs give no max duration or plan restriction. Source: https://docs.suprsend.com/docs/delay (checked 2026-11-15). **Batching and Digest show "—" for Free** and are listed under Business ($250/mo). "Recipient's Timezone" and "Send in Work hours" are also "—" on Free. Source: https://www.suprsend.com/pricing (checked 2026-11-15).
5. **Node SDK:** `@suprsend/node-sdk`, using `supr_client.workflows.trigger(new WorkflowTriggerRequest(body, opts))`. Source: https://github.com/suprsend/suprsend-node-sdk (search result; README not fetched in full).
6. **User identity:** Recipients can be inline in the trigger and are created or updated on trigger. Source: https://docs.suprsend.com/docs/node-trigger-workflow-from-api (search snippet, checked 2026-11-15).
7. **Lock-in:** Proprietary SaaS.

### MagicBell

1. **Free tier:** "Builder" costs "$0 /month" with **1,000 deliveries**, 1 project and unlimited users. "To count deliveries, we add up all the channels a user receives a notification on." Overage is "charged per use". **Card requirement not stated**, and no trial is mentioned. Source: https://www.magicbell.com/pricing (checked 2026-11-15).
2. **React feed:** `@magicbell/react` provides `FloatingInbox`, `Inbox`, `UserPreferences` and others. "You'll need a User JWT." Source: https://www.magicbell.com/docs/libraries/magicbell-react (checked 2026-11-15).
3. **Email:** Through Ping Email, SMTP, SendGrid, Mailgun and Amazon SES. **Resend and Brevo are not listed**; Brevo could only work through generic SMTP. Source: https://www.magicbell.com/docs/channels/email (checked 2026-11-15).
4. **Delays, digests, schedules:** The workflow `wait` step pauses "for a specified duration (in seconds)". Digest is not mentioned, and calendar scheduling is not described. Source: https://www.magicbell.com/workflows (checked 2026-11-15).
5. **Node SDK:** a "Browser/Node Client" (https://www.magicbell.com/docs, checked 2026-11-15).
6. **User identity:** Not verified.
7. **Lock-in:** Proprietary SaaS.

### Engagespot (excluded)

- `https://engagespot.co/`, `/pricing` and `www.` all return **HTTP 404 "Site Not Found | Framer"** (curl, 2026-11-15). **Pricing cannot be verified from a primary source.** Only third-party aggregators describe a "Launch – Free Forever" 10k-trigger plan, and those are not used here.
- The docs site still serves pages (https://docs.engagespot.co/docs/introduction/quick-start/, checked 2026-11-15). It shows `@engagespot/node`, an `identifier` per recipient, and Delay and Batch functions in the sidebar.
- The most recent push in the GitHub org `Engagespot` was 2025-01-25 (GitHub API, checked 2026-11-15).
- Verdict: the vendor's viability is doubtful, so I would not build on it.

### Pingram (formerly NotificationAPI; extra candidate)

1. **Free tier:** "Every account includes 100 SMS, 100 call minutes, and 3,000 emails per month at no cost — no credit card required." Source: https://www.pingram.io/pricing (checked 2026-11-15). `llms.txt` says "Free: 3,000 emails/in-app + 100 SMS/month, no credit card" (https://www.pingram.io/llms.txt, checked 2026-11-15). `notificationapi.com/pricing` redirects 301 to pingram.io.
2. **React feed:** `@notificationapi/react` provides `NotificationAPIProvider` and `NotificationPopup`. Source: https://www.pingram.io/docs/channels/inapp (search result, checked 2026-11-15).
3. **Email:** **Built in.** "No third-party accounts needed. Built-in SMTP" (https://www.pingram.io/llms.txt). However, "The shared Pingram address can only email the account itself"; reaching other people needs a verified domain. Source: https://www.pingram.io/docs/reference/node (checked 2026-11-15).
4. **Delays, digests, schedules:** `send()` takes `schedule` (ISO 8601), and `sender.updateSchedule()` and `sender.deleteSchedule()` modify or cancel a scheduled notification. That supports "nudge in 7 days unless they come back": schedule it, then delete it on activity. Notification types have a per-channel `defaultDeliveryOption` (`off`, `instant`, `hourly`, `daily`, `weekly`, `monthly`), which acts as a digest. Source: https://www.pingram.io/docs/reference/node (checked 2026-11-15). The pricing page does not say whether these features are on Free (unverified). There is no multi-step workflow engine (branching, chained delays) like Knock or Novu.
5. **Node SDK:** `pingram` (same source).
6. **User identity:** `to.id` is required, and `to.email` can be passed inline (same source).
7. **Lock-in:** Proprietary SaaS. It is simpler (a send API plus notification types), so there is less proprietary workflow logic to port.

## Email service side table (for bring-your-own providers)

| Service | Free tier | No card? | Restriction for demo | Works with |
|---|---|---|---|---|
| **Resend** | 3,000/mo, **100/day**, 3 domains; "free-forever plan" ([pricing](https://resend.com/pricing), checked 2026-11-15) | **Not stated** on the pricing page (unverified) | `onboarding@resend.dev` delivers only to your own account email; others return 403 until you verify a domain ([errors](https://resend.com/docs/api-reference/errors), [403 guide](https://resend.com/docs/knowledge-base/403-error-resend-dev-domain), checked 2026-11-15) | Knock, Novu, Courier (not MagicBell) |
| **Brevo** | 300 emails/day after account approval ([pricing](https://www.brevo.com/pricing/), checked 2026-11-15) | **Yes**: "Free forever, no credit card needed"; FAQ: "Do I have to enter my credit card...? No." | Account must be "approve[d] ... for sending" | Novu natively; Knock, Courier and MagicBell only via SMTP, if at all |
| **Mailgun** | $0/mo, 100 emails/day, 1 custom domain, 1-day logs ([pricing](https://www.mailgun.com/pricing/), checked 2026-11-15) | Not stated (unverified) | Sandbox rules not stated on the pricing page | Knock, Novu, Courier, MagicBell |
| **Postmark** | 100 emails/month, never expires, no overages ([pricing](https://postmarkapp.com/pricing), checked 2026-11-15) | **Yes**: "No Credit Card Required" | Until manual approval you can send only to your own verified domains ([approval](https://postmarkapp.com/support/article/1084-how-does-the-account-approval-process-work), checked 2026-11-15) | Knock, Novu, Courier |
| **SendGrid** | **Free trial only.** Pages say "Sign up for a free trial—no credit card required"; I found no ongoing free plan and the trial's length and limits are not stated ([sendgrid.com/en-us/pricing](https://sendgrid.com/en-us/pricing), which redirects to twilio.com, checked 2026-11-15) | Yes (trial) | Trial, not a lasting free tier | All |
| **Amazon SES** | No SES-specific free quota; new AWS accounts get "up to $200 in AWS Free Tier credits", and the free plan lasts 6 months ([pricing](https://aws.amazon.com/ses/pricing/), checked 2026-11-15) | Not stated on the page (AWS sign-up normally takes a card, but that is unverified here) | Sandbox not described on the page | Knock, Novu, Courier, MagicBell |

**Takeaway:** every service needs a **verified sending domain**, or account approval, before mail reaches arbitrary Students and Instructors. Pingram's built-in email has the same limit. For a demo that only emails team-owned addresses, Resend's sandbox or a provider's test sender is enough.

## Recommended shortlist

1. **Knock + Resend (or Postmark).** This is the only option where every need is confirmed on the free, no-card tier from primary sources: an in-app feed with a React component, a long delay (365 days) with cancel, digest/batch (90 days), and recipient-timezone recurring **schedules**. It allows inline user identification, so Cadence's SQLite stays the source of truth, and 10,000 messages a month is ample for dev and demo. The cost is proprietary lock-in: no self-host, and workflows only portable as CLI-pulled files. Brevo is not a native Knock email provider, so pair Knock with Resend or Postmark.
2. **Novu + Resend or Brevo.** It fits on everything except time-driven needs. It is the best choice for portability (MIT core, self-hostable, code-first workflows) and offers a no-card free tier, `<Inbox />`, and native Brevo/Resend support. But Free caps **delays and digests at 1 day** and allows **5 workflows**. Inactivity nudges and weekly digests would need Cadence to decide *when* to fire, for example with its own scheduler or a cron job, which drifts toward building part of it ourselves. Self-hosting might lift the caps (unverified).
3. **Pingram (simplest).** It is the only option with built-in email, so no second service is needed. It has a no-card free tier (3,000 emails/in-app a month), a React popup, cancellable scheduled sends, and hourly/daily/weekly digest delivery options. It is not a workflow engine, and Free availability of scheduling and digests is not stated on the pricing page, so it needs a hands-on check.

Not recommended: **SuprSend** (no digest or timezone sending on Free), **Courier** (digest and batching listed as Enterprise; card requirement not stated), **MagicBell** (1,000 deliveries; no digest; card requirement not stated), and **Engagespot** (site down, repos stale).

## Surprises

- **Engagespot's marketing site is gone** (Framer "Site Not Found" on every domain variant), while its docs remain up. Its GitHub has had no pushes since Jan 2025.
- **Novu's Free plan caps delay and digest at 1 day** and allows 5 workflows. That is a big limit for time-driven use despite the generous 10k runs.
- **Courier's pricing puts "Digest & batching" under Enterprise**, while its docs show no such restriction. The two official sources conflict, and the delay max is also inconsistent (90 vs 93 days).
- **SuprSend excludes digest and batching from Free** (Business tier, $250/mo).
- **SendGrid no longer advertises a permanent free plan**, only a free trial, and its pricing now lives on twilio.com.
- **NotificationAPI rebranded to Pingram** and now bundles its own email delivery.
- No provider's free tier lets you email arbitrary people without a verified domain.

## Does reality contradict the assumption?

Partly. The assumption that "a free, no-card tier with in-app feed + email + delays/digests exists" **holds for Knock only, fully verified**. In every other case the free tier is missing something:

- **Novu:** delays and digests are capped at 1 day.
- **SuprSend:** no digest.
- **Courier:** digest is Enterprise per the pricing page, and the card requirement is unknown.
- **MagicBell:** no digest, 1k deliveries, and the card requirement is unknown.
- **Pingram:** Free availability of scheduling and digests is not stated.

Across all of them, "email" really means "email through a provider with a verified domain".

## Open questions

- **Is Knock the clear winner?** On paper, yes. Novu is close only if we accept the 1-day caps or self-host. A short hands-on trial is worth it on two points:
  - Confirm that Knock's Free account really exposes delay, batch and schedules in the dashboard, and that sign-up asks for no card.
  - Check whether Pingram's scheduling and digest options work on Free.
- Does an inline `to` object in Novu's trigger create a missing subscriber? The docs do not say explicitly.
- Do Novu self-hosted delay and digest limits match Cloud's?
- Card requirements for Resend, Mailgun and Courier Developer are not stated on their pricing pages; confirm at sign-up.
- What is the file format of Knock's CLI-pulled workflows (JSON + Liquid?), which determines template portability? Not verified.
- Can a demo domain be verified for email? If not, demo email can only reach the account owner's address.

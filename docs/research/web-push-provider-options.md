# Browser Web Push provider options for Cadence

**Question:** Which external providers can deliver browser Web Push for Cadence, and how do the strongest options compare on browser/device support, EU data handling, security, per-recipient audience APIs, deep links, templates/localization, delivery observability, cost, rate limits, SDK/maintenance burden, and migration/exit risk?

**Research date:** 2026-11-20  
**Scope:** Product planning only; browser Web Push, not native mobile push or an in-app feed. Facts below are labeled **Fact**. Recommendations and trade-offs are labeled **Judgment**. “Unknown” means the cited first-party material did not establish the point; no claim is inferred.

## Recommendation

**Judgment: choose OneSignal as the preferred managed direction for Cadence v1, with direct Web Push as the deliberate escape hatch if EU residency or portability becomes a hard gate.**

Why OneSignal is the strongest overall fit: its first-party Web SDK explicitly covers Chrome, Firefox, Edge, and Safari; it supports per-user aliases and subscription targeting, launch URLs/click behavior, reusable templates, multi-language push variants, delivery/click reports and webhooks, and a low-friction free/growth price point. OneSignal also states that its primary data centers are in the EU, supports GDPR controls and a DPA on paid plans, and has published SOC 2/ISO controls. The trade-off is provider ownership of the SDK/service worker lifecycle and difficult subscription export between Web Push setups.

**Judgment: do not use Knock as the sole Web Push provider.** Knock explicitly says it has no first-party Web Push channel; its documented browser path is FCM plus Knock. That combination is credible if Cadence needs Knock’s orchestration, but it adds two vendors and two SDK/data models. Knock currently documents EU residency only as “coming” by the end of 2026 and only Enterprise has translations. FCM is a good no-cost transport, but Firebase’s published delivery reporting is strongest for Android/iOS rather than browser Web Push, and Cadence would own registration freshness, observability, templates/localization, and most product-level workflow concerns.

## Option comparison

| Criterion | OneSignal (managed) | FCM direct (managed transport) | Knock + FCM (orchestration over transport) | Standards-direct Web Push |
|---|---|---|---|---|
| Browser/device support | **Fact:** Web push docs cover Chrome, Firefox, Edge, Safari; HTTPS, same-origin service worker, user permission required. iOS Web Push needs iOS 16.4+ and Home Screen setup. | **Fact:** FCM Web works in browsers supporting the W3C Push API; HTTPS/service workers and permission are required. | **Fact:** Knock documents browser delivery through FCM and says Knock has no first-party Web Push channel. It therefore inherits FCM/browser limits. | **Fact:** determined by each user agent’s Push API support, not by the sender. Delivery follows the W3C Push API and RFC 8030/8291/8292 protocols. |
| EU data handling | **Fact:** OneSignal says its primary data centers are in the EU, acts as processor, supports GDPR rights, does not store EU/UK IPs by default, and offers a DPA on paid plans. | **Fact:** Firebase says Google generally acts as processor and publishes GDPR/transfer terms. It retains FCM installation IDs until deletion, then live/backup removal within 180 days. **Unknown:** no EU-region selection was established for FCM in the cited docs. | **Fact:** Knock says GDPR/CCPA controls and hard user deletion; EU data residency is expected by end of 2026, not stated as current. Firebase terms also apply downstream. | **Judgment:** subscription records can remain in Cadence-selected infrastructure; browser push services (for example, the browser vendor’s service) still receive the opaque encrypted delivery request. This is not necessarily EU-only delivery. |
| Security | **Fact:** TLS in transit and AES-256 at rest are claimed; SOC 2 Type II, ISO 27001/27701, 2FA/SSO, and API-key guidance are documented. | **Fact:** FCM uses authorized Admin SDK/HTTP v1 sends and short-lived OAuth access from service-account credentials; Google warns to protect service-account keys. FCM says transport is encrypted but not end-to-end between sender and receiver for provider-visible data. | **Fact:** Knock documents TLS 1.2 cross-service, encryption at rest, SOC 2 Type 2, GDPR, HIPAA, least privilege, regular penetration tests, and 2FA. FCM credentials remain a downstream trust boundary. | **Fact:** RFC 8291 protects content from inspection/modification by the push service; RFC 8292 binds sends with VAPID; RFC 8291 requires subscription keys/secrets to be exchanged over authenticated, confidential transport. **Judgment:** controls are strong but operational security and key rotation are Cadence’s responsibility. |
| Per-recipient audience APIs | **Fact:** users can be targeted by external alias or subscription; up to 20,000 aliases/subscription IDs per request. | **Fact:** API sends can target an individual registration token/FID, topic, or condition. | **Fact:** a workflow trigger accepts one or more recipient identifiers and recipient-specific trigger data; Knock stores one or more FCM tokens per user/device. | **Fact:** a subscription is a single capability-like endpoint plus key material; the protocol itself defines no user/audience model. **Judgment:** Cadence would build recipient-to-subscription lookup and fan-out. |
| Deep links | **Fact:** Web notifications support configurable click behavior and launch URLs, plus action buttons. | **Fact:** FCM Webpush options expose a required-HTTPS `link` for notification clicks. | **Fact:** Knock passes FCM `data` and `webpush` payload overrides, enabling a destination field, but the documented Knock Web Push flow leaves click/display handling in the Firebase client/service worker. | **Fact:** deep-link handling is represented in the Web Push/Notification payload and service-worker/client logic; it is not a managed provider feature. |
| Templates/localization | **Fact:** reusable push templates, Liquid personalization, API template IDs, and language-specific push variants are documented. | **Fact/limitation:** FCM exposes payloads for title/body/image and web-specific payload configuration, but no first-party template or localization workflow was established in cited FCM docs. | **Fact:** Knock has workflows/templates and Liquid; translations exist only on Enterprise. Knock can render a payload per recipient and pass data to FCM. | **Judgment:** Cadence owns template storage/versioning and localization; maximum portability and control, highest burden. |
| Delivery observability | **Fact:** push reports include sent/delivered-to-push-service, failed, confirmed receipt (plan/SDK dependent), clicked, audience activity, and 30-day API-message retention; Web Push webhooks can report events. | **Fact:** the v1 response confirms provider acceptance. Firebase console/BigQuery reports are documented, but the cited console report metrics explicitly cover Android/iOS; web-browser device receipt/opens are not established. | **Fact:** Knock logs requests, workflow/message lifecycle, and events. Critically, Knock explicitly says push never advances beyond `sent`: it does not support provider/device delivery tracking. | **Fact/judgment:** standards responses can expose HTTP acceptance and 404/410 stale subscriptions. End-user display/click confirmation requires Cadence telemetry; “accepted by push service” must not be presented as “seen by user.” |
| Cost | **Fact:** OneSignal Free: 10,000 web subscribers per send; Growth starts $19/month plus $0.004/web subscriber. Web push sends are unlimited on those plans. DPA and some security/legal features vary by plan. | **Fact:** FCM is shown as no-cost on both Firebase Spark and Blaze plans; associated analytics/storage products can have their own pricing. | **Fact:** Knock Free includes up to 10,000 successfully delivered messages/month. Starter is $250/month including 50,000, then $0.005/message. Enterprise translation/security features are custom. FCM adds no listed messaging fee. | **Judgment:** no provider message charge, but Cadence pays in infrastructure and engineering/operations. This should only be selected if avoiding control/risk costs is worth that burden. |
| Rate limits / reliability controls | **Fact:** OneSignal message create/cancel: 150 req/s/app Free, 6,000 req/s/app paid. In addition, >10× subscribed subscriptions delivered in a rolling 15 minutes can temporarily disable an app; idempotent retries are recommended. | **Fact:** FCM default downstream quota is 600,000 messages/minute/project; it returns 429 on quota exhaustion. Quota can be increased under published criteria. | **Fact:** Knock endpoint limits are assigned by tier (document lists 1–1,000 requests/second depending on endpoint) and returns 429. Trigger requests support 24-hour idempotency keys. FCM’s downstream quotas still apply. | **Fact:** RFC 8030 notes push services may store messages and impose quantity/size limits. **Unknown/judgment:** exact limits are push-service/browser specific; Cadence must handle 4xx/410/429 and retry. |
| SDK/maintenance burden | **Judgment: Low-to-medium.** Web SDK + OneSignal service worker; less orchestration than DIY. Still requires consent, user identity, service-worker coordination, and provider-specific API models. | **Judgment: Medium-high.** Firebase Web SDK/service worker plus token storage/freshness, templates, recipient mapping, and delivery telemetry remain Cadence concerns. | **Judgment: Medium-high overall, despite useful workflow abstraction.** Two control planes (Knock and FCM), Firebase browser SDK/service worker, Knock channel data, plus provider credentials. | **Judgment: Highest.** Cadence owns VAPID keys, subscriptions, expiry/rotation, encryption library, browser endpoints, retries, telemetry, templates, and service worker. |
| Migration / exit risk | **Judgment: High.** Browser subscriptions are origin/provider/SDK-bound; OneSignal states web subscriptions cannot be imported and migration registers users again on a return visit with the same permission. The API/user/template/analytics model is provider-specific. | **Judgment: Medium-high.** Existing VAPID keys can be imported into FCM, but browser tokens/FIDs and Firebase client service-worker ownership still require migration handling. | **Judgment: Medium-high.** Knock can conceptually change downstream channels, but its workflows/templates/recipient profiles are another migration surface; the documented browser client remains FCM-owned. | **Judgment: Lowest sender-platform lock-in.** Keeping Cadence’s VAPID key pair and raw subscriptions follows standards, but changing browser push-service endpoint ownership may still force client resubscription; this does not eliminate browser/platform constraints. |

## Key facts that change the decision

1. **Knock is not itself a browser transport.** Its official Web Push tutorial states: “Knock does not have a first-party web push channel” and routes through FCM. The choice is therefore not “Knock vs FCM”; it is “Knock + FCM vs another stack.”
2. **Managed observability is not equivalent to end-user delivery.** OneSignal distinguishes provider handoff from SDK-confirmed receipt. Knock explicitly stops push at `sent`. FCM’s cited analytics dashboard is not a complete browser Web Push receipt product. Cadence should track provider acceptance, confirmed display (if available), and click as separate states.
3. **EU handling and EU data residency are different questions.** OneSignal states EU primary data centers and a GDPR/DPA story. Knock currently promises future EU residency, while Firebase provides GDPR terms and transfer mechanisms but the cited FCM material does not establish an EU FCM region. Legal review is still required; this research is not legal advice.
4. **Browser migration is harder than backend migration.** Web permissions may survive, but subscriptions and service-worker ownership do not simply export into arbitrary providers. OneSignal explicitly says subscriptions cannot be imported; existing same-origin users must revisit to register.
5. **Direct Web Push is the portability benchmark, not the v1 default.** It minimizes provider workflow/vendor lock-in and can retain subscriptions/Cadence-controlled VAPID keys, but shifts security, key management, stale-subscription cleanup, retries, telemetry, templates, and client service-worker maintenance to Cadence.

## Judgment-based shortlist

1. **OneSignal — recommended default.** Best balance for Cadence’s user-level transactional notifications: broad browser coverage, individual audience APIs, deep links, templates/localization, reports, and current EU-centered posture at the lowest known paid setup cost.
2. **FCM direct — lowest-cost infrastructure fallback.** Best if avoiding paid managed capabilities and accepting more custom recipient, content, and observability work. It is already aligned with Knock’s own browser path.
3. **Knock + FCM — choose only for workflow-layer value.** Sensible if Cadence expects many event-to-channel rules, preferences, batching, or future channels to outweigh two-vendor cost and the current localization/observability/EU-residency gaps. For v1’s single channel and simple audience rules, that additional layer is not justified by the ticket criteria alone.
4. **Standards-direct — control/exit option.** Keep as a contingency, especially if a hard requirement emerges for Cadence-owned EU data location or provider-independent subscription storage. Do not select it merely for lower nominal message cost.

## Unknowns / validation before commitment

- Confirm the contract DPA, subprocessors, data locations, and transfer terms for the actual plan; public claims are not a substitute for procurement/legal review.
- Confirm whether OneSignal’s Growth plan includes the needed confirmed-receipt retention, audit/security features, and DPA under Cadence’s intended account.
- Confirm Knock’s actual EU-residency release and Enterprise quote if Knock is retained; its localization feature is not included in Free/Starter.
- Test the target browsers/OS versions Cadence supports, especially Safari/iOS Home Screen behavior. Public docs do not guarantee the same notification controls across browsers.
- Decide what “delivery observability” means: accepted by provider, received/displayed, or clicked. Providers expose materially different evidence.

## Primary sources

### Standards / direct Web Push

- [W3C Push API](https://www.w3.org/TR/push-api/) — architecture, subscriptions, service worker delivery, offline storage, push-service limits.
- [RFC 8030: Generic Event Delivery Using HTTP Push](https://datatracker.ietf.org/doc/html/rfc8030) — Web Push protocol, TTL/urgency, reliability and security/authorization considerations.
- [RFC 8291: Message Encryption for Web Push](https://datatracker.ietf.org/doc/html/rfc8291) — ECDH/AES-GCM confidentiality and integrity, subscription-secret handling.
- [RFC 8292: VAPID for Web Push](https://datatracker.ietf.org/doc/html/rfc8292) — sender identity, JWT expiry, VAPID/subscription restriction.
- [web-push-libs/web-push (official library repository)](https://github.com/web-push-libs/web-push) — representative maintained standards-direct Node library; API, VAPID and TTL/urgency/topic controls.

### OneSignal

- [Web Push setup](https://documentation.onesignal.com/docs/en/web-push-setup.md)
- [Web SDK setup](https://documentation.onesignal.com/docs/en/web-sdk-setup.md)
- [Create message / targeting](https://documentation.onesignal.com/reference/create-message.md)
- [Templates](https://documentation.onesignal.com/docs/en/templates.md)
- [Multi-language messaging](https://documentation.onesignal.com/docs/en/multi-language-messaging.md)
- [Push message reports](https://documentation.onesignal.com/docs/en/push-notification-message-reports.md)
- [Web Push webhooks](https://documentation.onesignal.com/docs/en/webhooks.md)
- [Rate limits and error handling](https://documentation.onesignal.com/reference/rate-limits.md)
- [Data collection and security FAQ](https://documentation.onesignal.com/docs/en/data-questions.md)
- [Handling personal data](https://documentation.onesignal.com/docs/en/handling-personal-data.md)
- [GDPR and individual rights](https://documentation.onesignal.com/docs/en/gdpr-compliance.md)
- [SOC 2 Type II](https://documentation.onesignal.com/docs/en/soc-2-type-ii.md)
- [Migration](https://documentation.onesignal.com/docs/en/migrating-to-onesignal.md)
- [Pricing](https://onesignal.com/pricing)

### Firebase Cloud Messaging

- [FCM Web client setup](https://firebase.google.com/docs/cloud-messaging/web/get-started)
- [FCM server environment](https://firebase.google.com/docs/cloud-messaging/server-environment)
- [FCM HTTP v1 send API](https://firebase.google.com/docs/cloud-messaging/send/v1-api)
- [FCM message schema, including WebpushConfig/link](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#webpushconfig)
- [Manage FCM registrations](https://firebase.google.com/docs/cloud-messaging/manage-tokens)
- [FCM message types/security note](https://firebase.google.com/docs/cloud-messaging/customize-messages/set-message-type)
- [FCM delivery reporting](https://firebase.google.com/docs/cloud-messaging/understand-delivery)
- [FCM throttling and quotas](https://firebase.google.com/docs/cloud-messaging/throttling-and-quotas)
- [Firebase privacy and security](https://firebase.google.com/support/privacy)
- [Firebase pricing](https://firebase.google.com/pricing)

### Knock

- [Sending Web Push with FCM](https://docs.knock.app/tutorials/sending-web-push-notifications-with-fcm.md)
- [Knock FCM integration](https://docs.knock.app/integrations/push/firebase.md)
- [Knock push overview](https://docs.knock.app/integrations/push/overview.md)
- [Trigger workflows / per-recipient data](https://docs.knock.app/send-notifications/triggering-workflows/api.md)
- [Message statuses / no push delivery tracking](https://docs.knock.app/send-notifications/message-statuses.md)
- [Translations (Enterprise)](https://docs.knock.app/template-editor/translations.md)
- [Data retention](https://docs.knock.app/manage-your-account/data-retention.md)
- [Rate limits](https://docs.knock.app/api-reference/overview/rate-limits.md)
- [Security](https://docs.knock.app/developer-tools/security.md)
- [Pricing](https://knock.app/pricing)

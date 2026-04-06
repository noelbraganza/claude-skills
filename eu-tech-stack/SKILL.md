---
name: eu-tech-stack
description: >
  EU-sovereign tech stack reference. Covers European alternatives to US cloud
  services: Hetzner (servers), UpCloud (Postgres), AppSignal (observability),
  Brevo (email), 46elks (SMS/OTP), OpenCage (geocoding), Mistral AI (LLMs),
  Mollie (payments). Use when building with GDPR compliance in mind, avoiding
  US data transfer concerns, or when asked about EU alternatives to AWS,
  Vercel, Resend, Stripe, Twilio, Google Maps, OpenAI, Datadog, etc.
---

# EU Tech Stack

A privacy-first, GDPR-native alternative to the typical US SaaS stack. All services below are headquartered and operated within the EU — no Schrems II concerns, no US CLOUD Act exposure.

---

## Stack overview

| Layer | EU Service | 🇺🇸 Equivalent | HQ |
|---|---|---|---|
| Servers / VPS | Hetzner | AWS EC2, DigitalOcean | 🇩🇪 Germany |
| Managed Postgres | UpCloud | Supabase, Neon, Railway | 🇫🇮 Finland |
| Observability | AppSignal | Datadog, Sentry, New Relic | 🇳🇱 Netherlands |
| Transactional email | Brevo | Resend, SendGrid | 🇫🇷 France |
| SMS / OTP | 46elks | Twilio, AWS SNS | 🇸🇪 Sweden |
| Geocoding | OpenCage | Google Maps, Mapbox | 🇩🇪 Germany |
| LLMs | Mistral AI | OpenAI, Anthropic | 🇫🇷 France |
| Payments | Mollie | Stripe | 🇳🇱 Netherlands |

---

## 🇩🇪 Hetzner — Servers & Infrastructure

**hetzner.com** · Data centers: Nuremberg, Falkenstein, Helsinki, Ashburn (US optional)

Hetzner is the go-to for cost-effective EU compute. A CX22 (2 vCPU, 4 GB RAM) costs ~€4/mo vs ~€35/mo on AWS.

### What it offers
- **Cloud VPS** (CX series) — ideal for self-hosting Next.js, databases, Docker apps
- **Dedicated servers** — bare metal from €35/mo
- **Object Storage** (S3-compatible) — cheap, fast, EU-resident
- **Load Balancers**, **Firewalls**, **Private Networks** (VPC equivalent)
- **Managed Kubernetes** (K3s on Hetzner is popular in EU indie dev community)

### Deployment patterns for Next.js on Hetzner
- **Coolify** — self-hosted Heroku/Vercel-like PaaS, one-click Next.js deploys, free OSS
- **Kamal** (by 37signals) — Docker-based zero-downtime deploys via SSH, no orchestration needed
- **Docker Compose** + Caddy (reverse proxy + auto TLS) — simple and solid for small teams

### Hetzner vs Vercel
| | Vercel | Hetzner + Coolify |
|---|---|---|
| Setup | Zero | 30–60 min initial |
| Edge/CDN | Yes (global) | Manual (Cloudflare free tier) |
| Auto-deploy | Yes | Yes (via Coolify) |
| Cost at scale | Expensive | Very cheap |
| Serverless functions | Yes | No — always-on server |
| EU data residency | Optional | Yes, always |

**Recommendation:** For full EU data sovereignty, Hetzner + Coolify + Cloudflare (CDN/DNS only) is a solid stack. Add Hetzner Object Storage for file uploads.

---

## 🇫🇮 UpCloud — Managed Postgres

**upcloud.com** · Data centers: Helsinki, Frankfurt, London, Amsterdam, Warsaw + more

Finnish cloud provider with a strong managed database offering. MaxIOPS block storage is notably fast.

### Managed Databases
- Postgres (primary choice), MySQL, Redis, OpenSearch
- Automated backups, point-in-time recovery, read replicas
- Connection pooling via PgBouncer built-in
- VPC peering with UpCloud compute

### Comparison with Supabase
Supabase bundles Postgres + Auth + Realtime + Storage + RLS. UpCloud is just the database. For a full EU equivalent of Supabase you'd need:

| Supabase feature | EU alternative |
|---|---|
| Postgres | UpCloud Managed Postgres |
| Auth (magic link, social) | Ory (🇩🇪), Keycloak (self-hosted), Zitadel (🇨🇭) |
| Realtime subscriptions | Supabase is actually EU-hostable (Stockholm region) or use Soketi (self-hosted Pusher) |
| Storage | Hetzner Object Storage (S3-compatible) |
| RLS | Built into Postgres — works with any host |

**Note:** Supabase itself can be self-hosted and run on Hetzner. The managed service's eu-north-1 (Stockholm) region is already within the EU.

### Connection string pattern
```
postgresql://user:password@host:5432/dbname?sslmode=require
```
With Prisma or Drizzle ORM, or use the `postgres` npm package directly.

---

## 🇳🇱 AppSignal — Observability

**appsignal.com** · HQ: Amsterdam, Netherlands

Application monitoring with error tracking, performance monitoring, and host metrics. GDPR-native, all data in EU.

### What it covers
- **Error tracking** — exceptions with full stack traces, breadcrumbs, user context
- **APM** — request performance, N+1 detection, slow queries, background jobs
- **Host metrics** — CPU, memory, disk, network for Hetzner VMs
- **Dashboards & Alerts** — anomaly detection, Slack/email notifications
- **Log management** (newer feature)

### Integration for Node.js / Next.js
```bash
npm install @appsignal/nodejs @appsignal/next
```
```ts
// instrumentation.ts (Next.js 14)
import { Appsignal } from '@appsignal/nodejs'
export const appsignal = new Appsignal({ name: 'your-app-name', pushApiKey: process.env.APPSIGNAL_PUSH_API_KEY })
```

### AppSignal vs Sentry
| | Sentry (🇺🇸) | AppSignal (🇳🇱) |
|---|---|---|
| Error tracking | ✅ Excellent | ✅ Good |
| APM / performance | ✅ | ✅ Better integrated |
| Host metrics | ❌ | ✅ |
| GDPR | Configurable | Native |
| Pricing | Free tier | From €18/mo |

---

## 🇫🇷 Brevo — Transactional Email

**brevo.com** (formerly Sendinblue) · HQ: Paris, France

Full-featured email platform: transactional, marketing, SMS, WhatsApp. GDPR-native, EU data centres.

### Transactional email (Resend equivalent)
```bash
npm install @getbrevo/brevo
```
```ts
import * as SibApiV3Sdk from '@getbrevo/brevo'

const apiInstance = new SibApiV3Sdk.TransactionalEmailsApi()
apiInstance.authentications['api-key'].apiKey = process.env.BREVO_API_KEY!

await apiInstance.sendTransacEmail({
  sender: { name: 'Your Company', email: 'noreply@your-domain.com' },
  to: [{ email: 'user@example.com', name: 'User Name' }],
  subject: 'You\'re on the list',
  htmlContent: '<h1>Hello</h1>',
})
```

### Key differences from Resend
| | Resend (🇺🇸) | Brevo (🇫🇷) |
|---|---|---|
| API simplicity | Very simple | More verbose |
| Email templates | Via React Email | Built-in drag-and-drop |
| Rate limits | 5/sec | 300/hour free, higher on paid |
| Marketing campaigns | No | Yes |
| SMS | No | Yes (same platform) |
| GDPR | Configurable | Native |
| Free tier | 3,000/mo | 9,000/mo (300/day) |
| EU data residency | US-based | EU (Paris) |

**Note:** Brevo's API is older and more complex than Resend. If email simplicity is a priority, Resend with EU Standard Contractual Clauses is acceptable for many GDPR use cases.

---

## 🇸🇪 46elks — SMS & OTP

**46elks.com** · HQ: Gothenburg, Sweden

Simple, developer-friendly SMS/voice API. Swedish telecom company — extremely clean API, pay-per-use, no monthly fees.

### SMS send
```ts
const response = await fetch('https://api.46elks.com/a1/sms', {
  method: 'POST',
  headers: {
    Authorization: 'Basic ' + Buffer.from(`${process.env.ELKS_API_USER}:${process.env.ELKS_API_PASSWORD}`).toString('base64'),
    'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: new URLSearchParams({
    from: 'YourApp',
    to: '+46701234567',
    message: 'Your OTP is 123456',
  }),
})
```

### OTP / Magic link via SMS
For passwordless auth via SMS instead of email:
1. Generate OTP with `crypto.randomInt(100000, 999999)`
2. Store in `otp_codes` table with expiry
3. Send via 46elks
4. Verify on submit

### 46elks vs Twilio
| | Twilio (🇺🇸) | 46elks (🇸🇪) |
|---|---|---|
| API complexity | High (many SDKs) | Very simple REST |
| SMS pricing (SE) | ~€0.08/msg | ~€0.06/msg |
| EU numbers | Yes | Yes (strong Nordic coverage) |
| Monthly fees | No | No |
| Voice / calls | Yes | Yes |
| Verification / OTP | Verify product | Manual (or simple) |
| GDPR | Configurable | Native |

**Alternative:** Sinch (🇸🇪 Stockholm) — larger scale, also EU-native.

---

## 🇩🇪 OpenCage — Geocoding

**opencagedata.com** · HQ: Essen, Germany

Forward and reverse geocoding API built on OpenStreetMap data. Privacy-respecting, GDPR-compliant.

### Forward geocoding (address → coordinates)
```ts
const response = await fetch(
  `https://api.opencagedata.com/geocode/v1/json?q=${encodeURIComponent(address)}&key=${process.env.OPENCAGE_API_KEY}&language=en&countrycode=se`
)
const data = await response.json()
const { lat, lng } = data.results[0]?.geometry ?? {}
```

### Reverse geocoding (coordinates → address)
```ts
const response = await fetch(
  `https://api.opencagedata.com/geocode/v1/json?q=${lat}+${lng}&key=${process.env.OPENCAGE_API_KEY}`
)
```

### Rate limits & pricing
- Free: 2,500 requests/day, 1 req/sec
- Small: €50/mo → 10,000/day, no rate limit
- Results include: formatted address, country, timezone, what3words, plus codes

### OpenCage vs Google Maps Geocoding
| | Google Maps (🇺🇸) | OpenCage (🇩🇪) |
|---|---|---|
| Data source | Proprietary | OpenStreetMap |
| Accuracy | Very high | High (improving) |
| GDPR | Complex | Native |
| Pricing | Pay-per-use (cheap) | Flat monthly |
| Terms | Restrictive | Open |

---

## 🇫🇷 Mistral AI — LLMs

**mistral.ai** · HQ: Paris, France

European LLM company with open-weight models. API is OpenAI-compatible, making migration easy.

### Models
| Model | Use case | Context |
|---|---|---|
| `mistral-small-latest` | Fast, cheap, good for simple tasks | 128k |
| `mistral-medium-latest` | Balanced capability/cost | 128k |
| `mistral-large-latest` | Most capable, complex reasoning | 128k |
| `open-mistral-7b` | Open-weight, self-hostable | 32k |
| `open-mixtral-8x7b` | Open-weight MoE, powerful | 32k |
| `codestral-latest` | Code generation specialist | 256k |

### API (OpenAI-compatible)
```ts
// Drop-in from OpenAI — just change baseURL and model
import OpenAI from 'openai'

const client = new OpenAI({
  apiKey: process.env.MISTRAL_API_KEY,
  baseURL: 'https://api.mistral.ai/v1',
})

const response = await client.chat.completions.create({
  model: 'mistral-large-latest',
  messages: [{ role: 'user', content: 'Summarise this text...' }],
})
```

Or use Mistral's own SDK:
```bash
npm install @mistralai/mistralai
```

### Mistral vs OpenAI / Anthropic
| | OpenAI (🇺🇸) | Anthropic (🇺🇸) | Mistral (🇫🇷) |
|---|---|---|---|
| Best model | GPT-4o | Claude Sonnet/Opus | Mistral Large |
| Open-weight models | No | No | Yes (7B, 8x7B) |
| Self-hostable | No | No | Yes |
| EU data residency | Optional | No | Yes |
| API compatibility | — | Partial | OpenAI-compatible |
| GDPR | Configurable | Configurable | Native |
| Pricing (large) | ~$15/M tokens | ~$15/M tokens | ~$8/M tokens |

**Note:** For EU sovereignty, Mistral's open-weight models can be self-hosted on Hetzner GPU instances (A100/H100 available), eliminating any third-party data processing.

---

## 🇳🇱 Mollie — Payments

**mollie.com** · HQ: Amsterdam, Netherlands

European payment processor with excellent coverage of local EU payment methods.

### What it supports
- **Cards**: Visa, Mastercard, Amex
- **EU local methods**: iDEAL (NL), Bancontact (BE), SEPA Direct Debit, Sofort (DE), Przelewy24 (PL)
- **Buy now pay later**: Klarna, in3
- **Digital wallets**: Apple Pay, Google Pay, PayPal
- **Recurring**: subscriptions, mandate-based

### Payment flow (Next.js)
```bash
npm install @mollie/api-client
```
```ts
import createMollieClient from '@mollie/api-client'

const mollie = createMollieClient({ apiKey: process.env.MOLLIE_API_KEY! })

// Create payment
const payment = await mollie.payments.create({
  amount: { currency: 'EUR', value: '10.00' },
  description: 'Order #1234',
  redirectUrl: `${process.env.NEXT_PUBLIC_BASE_URL}/orders/1234/confirmed`,
  webhookUrl: `${process.env.NEXT_PUBLIC_BASE_URL}/api/webhooks/mollie`,
  metadata: { order_id: orderId },
})

return redirect(payment.getCheckoutUrl()!)
```

```ts
// Webhook handler
export async function POST(request: Request) {
  const body = await request.formData()
  const paymentId = body.get('id') as string
  const payment = await mollie.payments.get(paymentId)

  if (payment.isPaid()) {
    // Mark order as paid, send confirmation
  }
}
```

### Mollie vs Stripe
| | Stripe (🇺🇸) | Mollie (🇳🇱) |
|---|---|---|
| EU local methods | Good | Excellent (especially NL/BE) |
| API quality | Best in class | Very good |
| Dashboard UX | Excellent | Good |
| Disputes/chargebacks | Good | Good |
| Subscription billing | Excellent | Good |
| EU data residency | Optional | Always |
| GDPR | Configurable | Native |
| Transaction fees (EU) | 1.5% + €0.25 | 1.2% + €0.29 (iDEAL: €0.29 flat) |

**Recommendation:** If your users are primarily Dutch/Belgian, Mollie wins on payment methods and price. For global reach or complex subscription logic, Stripe still leads.

---

## Full EU stack example (Next.js + Supabase equivalent)

| Typical US Stack | EU Alternative |
|---|---|
| Vercel | Hetzner VPS + Coolify |
| Supabase Auth | Supabase self-hosted on Hetzner OR Ory Kratos |
| Supabase Postgres | UpCloud Managed Postgres |
| Supabase Realtime | Soketi (self-hosted Pusher) on Hetzner |
| Resend | Brevo |
| Twilio / SMS | 46elks (SMS OTP) |
| Stripe | Mollie (payments) |
| Google Maps | OpenCage (venue geocoding) |
| OpenAI | Mistral AI (LLM features) |
| Datadog / Sentry | AppSignal (observability) |

**Cloudflare** (🇺🇸 but widely accepted for GDPR) as CDN/DNS/DDoS — used even in EU-first stacks. EU-native alternative: **Bunny.net** (🇸🇮 Slovenia).

---

## EU compliance notes

- **GDPR Article 44–49**: Transferring personal data outside EEA requires Adequacy Decision or Standard Contractual Clauses (SCCs). US services can use SCCs but add legal complexity.
- **Schrems II (2020)**: Invalidated Privacy Shield. US services now rely on SCCs, but enforcement varies by country (stricter in AT, DE, FR).
- **EU AI Act**: In force 2024. Affects AI feature design — Mistral is subject to EU rules, which may be a feature not a bug.
- **For Swedish/Nordic users**: All services above have at minimum EU coverage. 46elks has native Nordic numbers and routing.

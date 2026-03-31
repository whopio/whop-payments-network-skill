---
name: whop-payments-network
description: Integrate the Whop Payments Network into your platform — pay-ins, payouts, checkout, embedded components, API patterns, and webhooks.
---

# Whop Payments Network Integration

Whop provides a full payments network: accept payments (pay-ins), send payouts, embed checkout and wallet components, handle webhooks, and integrate chat. This skill covers the patterns you need to integrate Whop into any platform.

## SDK Packages

| Package | Purpose | Install |
|---------|---------|---------|
| `@whop/sdk` | Server-side API client (TypeScript) | `npm install @whop/sdk` |
| `whop-sdk` | Server-side API client (Python) | `pip install whop-sdk` |
| `whop_sdk` | Server-side API client (Ruby) | `gem install whop_sdk` |
| `@whop/checkout` | Embedded checkout React component | `npm install @whop/checkout` |
| `@whop/embedded-components-react-js` | Embedded payout/wallet/chat components (React) | `npm install @whop/embedded-components-react-js` |
| `@whop/embedded-components-vanilla-js` | Embedded components (Vanilla JS) | `npm install @whop/embedded-components-vanilla-js` |

## API Authentication

Base URL: `https://api.whop.com/api/v1`

All authenticated requests use Bearer token in the `Authorization` header:

```bash
curl https://api.whop.com/api/v1/payments?company_id=biz_xxx \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Key Types

| Type | When to use | How to get |
|------|-------------|------------|
| **Company API Key** | Access your own company's data, or connected accounts under your platform | Dashboard > Developer > Company API Keys > Create |
| **App API Key** | Access data on companies that installed your app | Dashboard > Developer > Create App > Environment Variables > `WHOP_API_KEY` |
| **OAuth Token** | Act on behalf of a specific user (sign-in with Whop) | OAuth 2.1 + PKCE flow. See [OAuth guide](https://docs.whop.com/developer/guides/oauth) |

### SDK Initialization

```typescript
import Whop from "@whop/sdk";

const client = new Whop({
  apiKey: process.env.WHOP_API_KEY,      // required
  appID: "app_xxxxxxxxxxxxxx",           // only for app API keys
});
```

```python
from whop_sdk import Whop

client = Whop(api_key="your_api_key")
```

```ruby
require "whop_sdk"

whop = WhopSDK::Client.new(api_key: ENV["WHOP_API_KEY"])
```

## Quick Decision Guide

| Need | Embedded (in your UI) | Hosted (redirect to Whop) |
|------|----------------------|--------------------------|
| **Checkout** | `<WhopCheckoutEmbed>` from `@whop/checkout/react` or Vanilla JS loader | Create a plan, redirect to `plan.purchase_url` |
| **Payouts dashboard** | `@whop/embedded-components-react-js` wallet elements | Create account link with `use_case: "hosted_payouts"` |
| **KYC onboarding** | N/A — always hosted | Create account link with `use_case: "hosted_kyc"` |
| **Chat** | `<ChatElement>` from `@whop/embedded-components-react-js` | N/A |

## Payment Integration (Pay-ins)

### Option A: Checkout Links (Simplest)

Create a plan to get a shareable `purchase_url`:

```typescript
const plan = await client.plans.create({
  company_id: "biz_xxxxxxxxxxxxx",
  access_pass_id: "pass_xxxxxxxxxxxxx",
  initial_price: 10.0,
  plan_type: "one_time", // or "renewal" for subscriptions
});

console.log(plan.purchase_url); // redirect customers here
```

### Option B: Embedded Checkout (Custom UI)

Two-step process: create a checkout configuration server-side, render the component client-side.

**Step 1 — Server: Create checkout configuration**

```typescript
const checkoutConfig = await client.checkoutConfigurations.create({
  company_id: "biz_xxxxxxxxxxxxx",
  plan: {
    initial_price: 10.0,
    plan_type: "one_time",
  },
  metadata: { order_id: "order_12345" },
});
// Pass checkoutConfig.id to the client as sessionId
```

**Step 2 — Client: Render checkout**

```tsx
import { WhopCheckoutEmbed } from "@whop/checkout/react";

export function Checkout({ sessionId }: { sessionId: string }) {
  return (
    <WhopCheckoutEmbed
      sessionId={sessionId}
      returnUrl="https://yoursite.com/checkout/complete"
      onComplete={(paymentId) => {
        console.log("Payment complete:", paymentId);
      }}
    />
  );
}
```

Or use `planId` directly without a checkout configuration:

```tsx
<WhopCheckoutEmbed
  planId="plan_XXXXXXXXX"
  returnUrl="https://yoursite.com/checkout/complete"
/>
```

### Option C: Vanilla JS Checkout (Non-React)

Add the loader script and a `div` with data attributes:

```html
<script async defer src="https://js.whop.com/static/checkout/loader.js"></script>

<div
  data-whop-checkout-plan-id="plan_XXXXXXXXX"
  data-whop-checkout-return-url="https://yoursite.com/checkout/complete"
></div>
```

See `references/checkout-embed.md` for full prop reference, programmatic controls, and sandbox testing.

## Payout Integration

### Embedded Wallet Components

Use `@whop/embedded-components-react-js` to embed payout dashboards (balance, withdrawal, payout methods) directly in your platform.

Requires an **access token** created server-side:

```typescript
const token = await client.accessTokens.create({
  company_id: "biz_xxxxxxxxxxxxx",  // the sub-merchant's company
});
// Pass token to client-side components
```

### Hosted Payouts Dashboard

For a fully hosted experience, create an account link:

```typescript
const link = await client.accountLinks.create({
  company_id: "biz_xxxxxxxxxxxxx",
  use_case: "hosted_payouts",
  return_url: "https://yourplatform.com/dashboard",
  refresh_url: "https://yourplatform.com/refresh",
});
// Redirect user to link.url
```

### Hosted KYC Onboarding

```typescript
const link = await client.accountLinks.create({
  company_id: "biz_xxxxxxxxxxxxx",
  use_case: "hosted_kyc",
  return_url: "https://yourplatform.com/dashboard",
  refresh_url: "https://yourplatform.com/refresh",
});
```

See `references/payouts.md` for embedded component setup and the interactive playground.

## Chat SDK

Embed real-time chat in your platform using React, Vanilla JS, or Swift.

**React quick start:**

```tsx
import { ChatElement, ChatSession, Elements } from "@whop/embedded-components-react-js";
import { loadWhopElements } from "@whop/embedded-components-vanilla-js";

const elements = loadWhopElements();

async function getToken() {
  const res = await fetch("/api/token");
  return (await res.json()).token;
}

function ChatPage() {
  return (
    <Elements elements={elements}>
      <ChatSession token={getToken}>
        <ChatElement
          options={{ channelId: "chat_XXXXXXXXXXXXXX" }}
          style={{ height: "100dvh", width: "100%" }}
        />
      </ChatSession>
    </Elements>
  );
}
```

See `references/chat-sdk.md` for Vanilla JS, Swift, and full configuration.

## Webhook Handling

Listen for events like `payment.succeeded`, `membership.activated`, `membership.deactivated`.

**Setup:** Dashboard > Developer > Create Webhook > select events > provide URL.

**Handling with SDK (Next.js example):**

```typescript
import type { NextRequest } from "next/server";
import { whopsdk } from "@/lib/whop-sdk";

export async function POST(request: NextRequest): Promise<Response> {
  const body = await request.text();
  const headers = Object.fromEntries(request.headers);
  const webhookData = whopsdk.webhooks.unwrap(body, { headers });

  if (webhookData.type === "payment.succeeded") {
    // handle payment
  }

  return new Response("OK", { status: 200 }); // return 2xx quickly
}
```

**SDK setup with webhook secret:**

```typescript
import { Whop } from "@whop/sdk";

export const whopsdk = new Whop({
  appID: process.env.NEXT_PUBLIC_WHOP_APP_ID,
  apiKey: process.env.WHOP_API_KEY,
  webhookKey: btoa(process.env.WHOP_WEBHOOK_SECRET || ""),
});
```

See `references/webhooks.md` for available events, app vs company webhooks, and validation details.

## Permissions System

Apps must request permissions before accessing company data. Each API endpoint documents its required permission scopes.

**Setup flow:**
1. Dashboard > Developer > select app > Permissions tab
2. Add required permissions with justification
3. Mark permissions as required or optional
4. Save and install the app on your company
5. Approve the permissions when prompted

**When updating permissions:** Creators see a "Re-approve" button. Until re-approved, new permission API calls will fail — handle these errors gracefully.

See `references/api-reference.md` for SDK setup patterns and the MCP server.

## MCP Server Access

AI agents can access the Whop API via MCP:

| Transport | URL |
|-----------|-----|
| HTTP Streaming (Cursor) | `https://mcp.whop.com/mcp` |
| SSE (Claude) | `https://mcp.whop.com/sse` |
| Docs MCP | `https://docs.whop.com/mcp` |

## Connected Accounts (Platforms)

If you are a platform with sub-merchants, use your Company API key to manage connected accounts:

```typescript
// Create a connected account (sub-merchant)
const company = await client.companies.create({
  parent_company_id: "biz_yourplatform",
  // ...company details
});

// List connected accounts
const companies = await client.companies.list({
  parent_company_id: "biz_yourplatform",
});
```

## Common Gotchas

1. **`returnUrl` is required** for embedded checkout when using external payment methods (Apple Pay, Google Pay, PayPal). Without it, redirects from payment providers will fail.

2. **`setAddress` only works with `hideAddressForm`** — You must set `hideAddressForm={true}` (React) or `data-whop-checkout-hide-address="true"` (Vanilla JS) before calling `setAddress()`.

3. **Sandbox vs Production** — Use `environment="sandbox"` prop and sandbox plan IDs from `sandbox.whop.com/dashboard`. Production is the default.

4. **Webhook secret encoding** — Pass `btoa(process.env.WHOP_WEBHOOK_SECRET)` (base64-encoded) to the SDK's `webhookKey` option.

5. **Return 2xx quickly from webhooks** — If your webhook handler takes too long, Whop will retry. Use `waitUntil()` or a background job for heavy processing.

6. **Permission re-approval** — After adding new permissions to your app, API calls requiring those permissions will fail until the company re-approves. Handle `403` errors gracefully.

7. **Access tokens expire** — Tokens from `accessTokens.create()` default to 1 hour, max 3 hours. Refresh before expiry for embedded components.

8. **`setupFutureUsage: "off_session"`** — Required on the checkout embed when you plan to charge the user later via `chargeUser` API. Filters out incompatible payment methods.

## File Reference

| File | Contents |
|------|----------|
| `references/checkout-embed.md` | Full prop reference, programmatic controls, Vanilla JS, sandbox, Apple Pay |
| `references/payouts.md` | Embedded wallet components, hosted payouts, account links, playground |
| `references/chat-sdk.md` | Chat SDK for React, Vanilla JS, Swift with full examples |
| `references/api-reference.md` | SDK initialization, key endpoints, MCP server setup |
| `references/webhooks.md` | Webhook events, validation, company vs app webhooks |
| `codebase-scan.md` | Prompt to analyze a client's codebase for Whop integration planning |

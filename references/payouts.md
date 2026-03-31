# Payouts Integration Reference

Whop provides two approaches for payout integration: embedded wallet components within your UI, or hosted payout dashboards via account links.

## Embedded Wallet Components (React)

Use `@whop/embedded-components-react-js` to embed payout elements directly in your platform's UI.

### Install

```bash
npm install @whop/embedded-components-react-js @whop/embedded-components-vanilla-js
```

### Access Token (Required)

Embedded components require an access token created server-side. The token is scoped to a specific company (sub-merchant):

```typescript
import Whop from "@whop/sdk";

const client = new Whop({ apiKey: process.env.WHOP_API_KEY });

// Create an access token for the sub-merchant
const token = await client.accessTokens.create({
  company_id: "biz_xxxxxxxxxxxxx",  // sub-merchant company ID
  // expires_at: defaults to 1 hour, max 3 hours
  // scoped_actions: optional, inherits all permissions if omitted
});

// Pass token.token to the client
```

### Component Setup

```tsx
import { Elements } from "@whop/embedded-components-react-js";
import { loadWhopElements } from "@whop/embedded-components-vanilla-js";

const elements = loadWhopElements();

async function getToken() {
  const res = await fetch("/api/payout-token");
  const data = await res.json();
  return data.token;
}

function PayoutDashboard() {
  return (
    <Elements elements={elements}>
      {/* Wallet/payout components go here */}
    </Elements>
  );
}
```

### Vanilla JS Setup

```typescript
import { loadWhopElements } from "@whop/embedded-components-vanilla-js";

async function getToken() {
  const res = await fetch("/api/payout-token");
  return (await res.json()).token;
}

const whopElements = await loadWhopElements();

// Create and mount elements programmatically
const element = whopElements.createElement("payout-element", {
  // options
});
element.mount("#payout-container");
```

## Hosted Payouts Dashboard

For a fully hosted experience, generate an account link that redirects the sub-merchant to their Whop payouts portal:

```typescript
const link = await client.accountLinks.create({
  company_id: "biz_xxxxxxxxxxxxx",   // sub-merchant company ID
  use_case: "hosted_payouts",
  return_url: "https://yourplatform.com/dashboard",
  refresh_url: "https://yourplatform.com/refresh",
});

// Redirect the user to link.url
```

The hosted payouts dashboard shows:
- Available balance
- Payout history
- Payout methods management
- Withdrawal requests

## Hosted KYC Onboarding

For sub-merchant verification and onboarding:

```typescript
const link = await client.accountLinks.create({
  company_id: "biz_xxxxxxxxxxxxx",
  use_case: "hosted_kyc",
  return_url: "https://yourplatform.com/dashboard",
  refresh_url: "https://yourplatform.com/refresh",
});
```

## Platform (Connected Accounts) Setup

If you run a platform with sub-merchants:

### Create Connected Accounts

```typescript
const subMerchant = await client.companies.create({
  parent_company_id: "biz_yourplatform",
  // additional company details
});
```

### List Connected Accounts

```typescript
const companies = await client.companies.list({
  parent_company_id: "biz_yourplatform",
});
```

### Fee Markups

Configure platform fees on connected accounts:

```typescript
const markups = await client.feeMarkups.list({
  company_id: "biz_yourplatform",
});
```

## Payout API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/withdrawals` | GET | List withdrawals |
| `/api/v1/withdrawals` | POST | Create withdrawal |
| `/api/v1/payout_methods` | GET | List payout methods |
| `/api/v1/payout_accounts` | GET | List payout accounts |
| `/api/v1/ledger_accounts` | GET | List ledger accounts (balances) |
| `/api/v1/transfers` | GET | List transfers |
| `/api/v1/account_links` | POST | Create account link (hosted dashboard) |
| `/api/v1/access_tokens` | POST | Create access token (embedded components) |
| `/api/v1/verifications` | GET | List verification status |

## Interactive Playground

Try embedded payout components live at: [docs.whop.com/developer/platforms/playground](https://docs.whop.com/developer/platforms/playground)

## Important Notes

- Access tokens expire (default 1 hour, max 3 hours). Implement token refresh logic for long sessions.
- The `company_id` in access token creation must be a sub-merchant of the API key's company.
- `scoped_actions` on access tokens can restrict permissions to a subset of the API key's permissions. Omit to inherit all.
- Hosted payout dashboards handle all UI, KYC, and compliance flows. Use when you want minimal frontend work.
- Embedded components give full control over look and feel but require more integration work.

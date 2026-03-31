# API Reference

Base URL: `https://api.whop.com/api/v1`

## SDK Installation

### TypeScript / JavaScript

```bash
npm install @whop/sdk
```

```typescript
import Whop from "@whop/sdk";

const client = new Whop({
  apiKey: process.env.WHOP_API_KEY,      // required
  appID: "app_xxxxxxxxxxxxxx",           // only for app API keys
});
```

### Python

```bash
pip install whop-sdk
```

```python
import os
from whop_sdk import Whop

client = Whop(
    api_key=os.environ.get("WHOP_API_KEY"),
    app_id="app_xxxxxxxxxxxxxx",  # only for app API keys
)
```

### Ruby

```bash
gem install whop_sdk
```

```ruby
require "whop_sdk"

whop = WhopSDK::Client.new(
  api_key: ENV["WHOP_API_KEY"],
  app_id: "app_xxxxxxxxxxxxxx",  # only for app API keys
)
```

## Authentication

Include your API key in the `Authorization` header using Bearer scheme:

```bash
curl https://api.whop.com/api/v1/payments?company_id=biz_xxx \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### API Key Types

| Type | Use case | Where to get |
|------|----------|--------------|
| **Company API Key** | Your own company's data + connected accounts | Dashboard > Developer > Company API Keys |
| **App API Key** | Data on companies that installed your app | Dashboard > Developer > App > Environment Variables |
| **OAuth Token** | Act on behalf of a user | OAuth 2.1 + PKCE flow |

### Company API Key Creation

1. Go to [developer dashboard](https://whop.com/dashboard/developer)
2. Click "Create" in the Company API Keys section
3. Name it (e.g., "Data pipeline", "GHL Integration")
4. Select a role or custom permissions
5. Copy the key from the modal

### App API Key Location

1. Go to [developer dashboard](https://whop.com/dashboard/developer)
2. Create or select an app
3. Find `WHOP_API_KEY` in the Environment Variables section
4. Use the reveal button to show and copy it

## Key API Endpoints

### Payments

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/payments` | GET | List payments for a company |
| `/payments/{id}` | GET | Retrieve a specific payment |

```typescript
const payments = await client.payments.list({
  company_id: "biz_xxxxxxxxxxxxxx",
});
```

### Plans

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/plans` | POST | Create a plan (returns `purchase_url`) |
| `/plans` | GET | List plans |
| `/plans/{id}` | GET | Retrieve a plan |

```typescript
const plan = await client.plans.create({
  company_id: "biz_xxxxxxxxxxxxx",
  access_pass_id: "pass_xxxxxxxxxxxxx",
  initial_price: 10.0,
  plan_type: "one_time",  // or "renewal"
});
```

### Checkout Configurations

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/checkout_configurations` | POST | Create a checkout config (for embedded checkout) |

```typescript
const config = await client.checkoutConfigurations.create({
  company_id: "biz_xxxxxxxxxxxxx",
  plan: { initial_price: 10.0, plan_type: "one_time" },
  metadata: { order_id: "order_12345" },
});
```

### Memberships

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/memberships` | GET | List memberships |
| `/memberships/{id}` | GET | Retrieve a membership |

```typescript
const memberships = await client.memberships.list({
  company_id: "biz_xxxxxxxxxxxxxx",
});
```

### Companies (Connected Accounts)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/companies` | POST | Create a company (connected account) |
| `/companies` | GET | List companies |
| `/companies/{id}` | GET | Retrieve a company |

```typescript
// Create connected account
const company = await client.companies.create({
  parent_company_id: "biz_yourplatform",
});

// List connected accounts
const companies = await client.companies.list({
  parent_company_id: "biz_yourplatform",
});
```

### Access Tokens

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/access_tokens` | POST | Create access token for embedded components |

```typescript
const token = await client.accessTokens.create({
  company_id: "biz_xxxxxxxxxxxxx",
  // expires_at: optional, default 1h, max 3h
  // scoped_actions: optional, inherits all permissions if omitted
});
```

### Account Links

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/account_links` | POST | Generate hosted dashboard URL |

```typescript
const link = await client.accountLinks.create({
  company_id: "biz_xxxxxxxxxxxxx",
  use_case: "hosted_payouts",  // or "hosted_kyc"
  return_url: "https://yourplatform.com/return",
  refresh_url: "https://yourplatform.com/refresh",
});
```

### Users

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/users/{username}` | GET | Get public user profile (no auth required) |

```bash
curl https://api.whop.com/api/v1/users/j
```

### Webhooks

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/webhooks` | POST | Create a webhook |
| `/webhooks` | GET | List webhooks |

## MCP Server

AI agents can access the full Whop API via MCP servers:

### Whop API MCP

| Transport | URL |
|-----------|-----|
| HTTP Streaming (Cursor) | `https://mcp.whop.com/mcp` |
| SSE (Claude) | `https://mcp.whop.com/sse` |

Provide your API key when connecting. Supports all API operations.

### Whop Docs MCP

| Transport | URL |
|-----------|-----|
| All clients | `https://docs.whop.com/mcp` |

Search and read Whop documentation. No auth required.

### Setup Examples

```json
// Cursor: ~/.cursor/config/mcp.json
{
  "mcpServers": {
    "whop-docs": { "url": "https://docs.whop.com/mcp" },
    "whop-api": { "url": "https://mcp.whop.com/mcp" }
  }
}
```

```json
// Claude Code: ~/.claude/mcp.json
{
  "mcpServers": {
    "whop-docs": { "url": "https://docs.whop.com/mcp" },
    "whop-api": { "url": "https://mcp.whop.com/sse" }
  }
}
```

## Pagination

List endpoints return paginated responses. Access results via `.data`:

```typescript
const page = await client.payments.list({ company_id: "biz_xxx" });
const payments = page.data;
```

## Error Handling

- **401** — Invalid or missing API key
- **403** — Missing permissions (app needs re-approval, or scope not granted)
- **404** — Resource not found
- **422** — Invalid request parameters
- **429** — Rate limited

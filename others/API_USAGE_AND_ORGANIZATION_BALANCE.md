# API Usage Tracking & Organization Balance — Combined Backend Specification

## Document purpose

This is the **single combined reference** for:

1. **API Usage Tracking** — How tokens and cost are recorded per AI call (`api_usage`), usage statistics APIs, and pricing.
2. **Organization Balance** — Workspace balance, top-up, deduction from balance when API is used, optional free credits on workspace creation, and balance APIs.

Use this document when implementing or maintaining either system. The two systems work together: usage is recorded first (Part I); balance deducts that cost from the workspace (Part II).

---

## Table of contents

| Section | Content |
|---------|---------|
| **Credits vs USD — Terminology** | Storage (USD), “credits” as UI term, consumption/balance in APIs |
| **Part I: API Usage Tracking** | Entities, pricing, recording flow, usage stats APIs, consumption summary, transaction history, database, config |
| **Part II: Organization Balance** | Balance model, top-up, deduction, free credits, **AI usage cap (org-level)**, balance APIs, database |
| **Part III: End-to-end flow** | How usage recording and balance deduction work together |
| **Combined API reference** | All endpoints (usage + balance) in one table |
| **Combined checklist** | Implementation checklist for both systems |
| **Future: Coupon system** | Planned coupon feature (summary) |
| **Configuration & troubleshooting** | Shared config and troubleshooting |

---

## Credits vs USD — Terminology

This section clarifies how **credits** and **USD** are used so implementation and APIs stay consistent.

### Storage: everything is USD

All monetary values in the system are **stored and processed in USD**:

| Where | Field / concept | Unit | Notes |
|-------|------------------|------|--------|
| **api_usage** | `cost` | USD | Cost of the AI call (from ApiPricingService). |
| **workspace_balance** | `balance_usd` | USD | Current prepaid balance for the workspace. |
| **workspace_balance_transaction** | `amount_usd`, `balance_before`, `balance_after` | USD | Positive = credit added, negative = deduction. |
| **Deduction on API use** | Amount deducted | USD | Same value as `api_usage.cost` for that call. |

There is **no separate “credits” column** in the database. Any “credits” notion is a **display or product convention**, not a second currency.

### “Credits” as a product/UI term

- **AI credits / consumption** — In the UI, you may label usage as “AI credits used” or “consumption.” In this spec, that value is the **same as cost in USD** unless you define a conversion.
- **Free credits** — “Free credits” = balance granted (e.g. on workspace creation). The amount is still stored and applied in **USD** (e.g. `initial_credit_usd` or `amount_usd` in the balance transaction).
- **Top-up** — “Adding credit” or “top-up” means adding to `balance_usd`; the amount is in **USD**.

### Recommended approach: USD end-to-end

- **Backend:** Store and return all amounts in **USD** (e.g. `consumption` in Consumption Summary and Transaction History = negative USD spent).
- **Frontend:** You can:
  - Show USD directly (e.g. “-$0.25”), or
  - Treat **1 credit = 1 USD** and show “-250 credits” when the stored value is 250 (or -250 for consumption), or
  - Use a configurable ratio (e.g. 1 credit = $0.01) and convert only for display.

If you use a ratio, keep it in config or the front end; the backend does not need a “credits” unit.

### In this document

- **Cost** — Always USD (e.g. `api_usage.cost`, deduction amount).
- **Balance** — Always USD (`balance_usd`).
- **Consumption** — In API responses, **consumption** is the amount used in the same unit as storage: **USD**. Negative values represent usage (e.g. `-0.25` = $0.25 spent). If your UI shows “credits,” apply your display rule (e.g. 1 credit = $1) on the client.
- **Free credits / top-up** — Stored and processed as USD; “credits” is just the product name for “balance added.”

---

# Part I: API Usage Tracking

## I.1 Overview

The API Usage Tracking system automatically records OpenAI (and other provider) API usage: token counts and calculated cost per call, per user and per workspace. It provides usage statistics APIs (tokens, cost, breakdown by model) for users and workspaces.

**Note (credits vs USD):** Currently, 1 credit = 1 USD. Internally, all values are stored as USD (cost, balance_usd). The term *credits* is a UI/UX abstraction and may diverge in the future (e.g. coupons or bonus credits). See **Credits vs USD — Terminology** for full clarification.

## I.2 Features

- **Automatic tracking** — Token usage recorded when OpenAI API calls are made
- **Cost calculation** — Real-time cost from ApiPricingService (OpenAI pricing)
- **User-level tracking** — Per-user usage and stats
- **Workspace-level tracking** — Aggregate usage per workspace
- **Model breakdown** — Statistics grouped by AI model
- **Date range queries** — Filter usage by date range
- **Pagination** — Usage history with pagination

## I.3 Data model — Usage

### Table: `api_usage`

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR(36) PK | UUID |
| user_id | VARCHAR(36) | User who made the call (FK users) |
| workspace_id | VARCHAR(36) | Workspace context (optional) |
| provider | VARCHAR(50) | e.g. "openai", "gemini" |
| model | VARCHAR(100) | e.g. "gpt-4o-mini", "dall-e-2" |
| api_type | VARCHAR(50) | e.g. "text", "image" |
| prompt_tokens | INT | Prompt tokens |
| completion_tokens | INT | Completion tokens |
| total_tokens | INT | Total tokens |
| cost | DECIMAL(10,6) | Cost in USD (from ApiPricingService) |
| request_id | VARCHAR(100) | Optional OpenAI request id |
| endpoint | VARCHAR(200) | API endpoint used |
| feature | VARCHAR(80) | **Optional.** Feature/category for reporting: e.g. `COURSE_GENERATION`, `IMAGE_CREATION`, `TEXT_ENHANCEMENT`, `VIDEO_GENERATION`. Set when recording usage (caller knows context). Used for Consumption Summary and Transaction History. |
| created_at | DATETIME(6) | NOT NULL |

**Indexes:** `user_id`, `created_at`, `(provider, model)`, `workspace_id`, `feature` (optional, for summary queries).

**Feature values (suggested):** Map to display names: COURSE_GENERATION → "AI Course Generation", IMAGE_CREATION → "AI Image Creation", TEXT_ENHANCEMENT → "AI Text Enhancement", VIDEO_GENERATION → "AI Video Generation". If `feature` is null, show as "Other" or derive from `api_type` (e.g. text → Text Enhancement, image → Image Creation).

**Migration:** e.g. `V42__Create_api_usage_table.sql`

## I.4 Components (usage)

| Component | Purpose |
|-----------|---------|
| **ApiUsage** (entity) | Maps to `api_usage` |
| **ApiUsageRepository** | Find by user/workspace, aggregates (total tokens, total cost, by model), date range |
| **ApiPricingService** | calculateTextCost(model, promptTokens, completionTokens), calculateImageCost(model, imageCount); pricing per model (gpt-4o-mini, gpt-4o, dall-e-2, etc.) |
| **ApiUsageService** | recordUsage(...), recordUsageForCurrentUser(...); getUserUsageStats, getCurrentUserUsageStats, getWorkspaceUsageStats, getUserUsageHistory, getUserUsageStatsByDateRange |
| **OpenAIProvider** | After each API call, extracts `usage` from response, calls ApiUsageService.recordUsageForCurrentUser(...) with tokens and cost |
| **ApiUsageController** | GET /api/usage/stats, /api/usage/history, /api/usage/stats/range, /api/usage/stats/user/{userId}, /api/usage/stats/workspace/{workspaceId} |

## I.5 Pricing (reference)

- **Text (per 1M tokens):** gpt-4o-mini (input $0.15, output $0.60), gpt-4o (input $2.50, output $10), gpt-4-turbo, gpt-3.5-turbo.
- **Image:** dall-e-2 $0.02/image, dall-e-3 $0.04/image (1024x1024).

Pricing is configured in `ApiPricingService` (e.g. `initializePricing()`).

## I.6 Usage recording flow

1. User triggers an AI call (e.g. via OpenAIProvider).
2. Provider calls OpenAI (or other); response includes `usage` (prompt_tokens, completion_tokens, total_tokens).
3. ApiPricingService calculates cost from model + tokens (or image count).
4. ApiUsageService.recordUsageForCurrentUser(provider, model, apiType, promptTokens, completionTokens, totalTokens, requestId, endpoint) — uses AuthUtil for current user and workspace, calculates cost if not passed, saves ApiUsage row.

So every AI call results in one `api_usage` row with `workspace_id`, `user_id`, `cost`, and token counts.

## I.7 Usage API endpoints

| Method | Endpoint | Description | Who |
|--------|----------|-------------|-----|
| GET | `/api/usage/stats` | Current user usage stats (tokens, cost, model breakdown) | **Author** (own only) |
| GET | `/api/usage/history` | Paginated usage history (page, size, sortBy, sortDir) | **Author** (own only) |
| GET | `/api/usage/stats/range` | User stats in date range (startDate, endDate) | **Author** (own only) |
| GET | `/api/usage/stats/user/{userId}` | User stats | SUPER_ADMIN or **that user** (author sees only self) |
| GET | `/api/usage/stats/workspace/{workspaceId}` | Workspace stats (all users in workspace) | SUPER_ADMIN or **TENANT** (org admin) |

All require JWT. Responses use ApiResponseDto with data (totalTokens, totalCost, modelBreakdown or list of usage records).

**Visibility by role (usage):**

| Role | What they see |
|------|----------------|
| **Author** | **Only own data:** `/api/usage/stats`, `/api/usage/history`, `/api/usage/stats/range`, consumption-summary, transaction-history — all **filtered by current user** (user_id = self). |
| **TENANT (org admin)** | **All users in workspace:** `/api/usage/stats/workspace/{workspaceId}`, consumption-summary and transaction-history for workspace (all users), and per-user stats when allowed. *Currently org admin = TENANT role only.* |
| **SUPER_ADMIN** | All workspaces and all users; any workspaceId / userId. |

**Default date range:** For endpoints that accept `startDate` and `endDate` (e.g. `/api/usage/stats/range`, `/api/usage/consumption-summary`, `/api/usage/transaction-history`): if both are omitted, default to the current month (start of month 00:00:00 to end of month 23:59:59).

## I.8 Usage response examples

**GET /api/usage/stats** — Example data:
```json
{
  "totalTokens": 15000,
  "totalCost": 0.002250,
  "modelBreakdown": [
    { "model": "gpt-4o-mini", "totalTokens": 15000, "totalCost": 0.002250, "requestCount": 25 }
  ]
}
```

**GET /api/usage/history** — Paginated list of usage records (id, userId, workspaceId, provider, model, apiType, promptTokens, completionTokens, totalTokens, cost, requestId, endpoint, feature, createdAt).

---

## I.9 Consumption Summary API

**Purpose:** A summary of AI credits (consumption) used in a selected period, grouped by feature (e.g. AI Course Generation, AI Image Creation, AI Text Enhancement, AI Video Generation).

**Endpoint:** `GET /api/usage/consumption-summary`

**Query parameters:**

| Parameter | Type | Required | Description |
|------------|------|----------|-------------|
| startDate | ISO 8601 datetime | Yes | Start of period |
| endDate | ISO 8601 datetime | Yes | End of period |
| workspaceId | UUID | No | Workspace scope; if omitted, use current user's workspace. Admin may pass any workspace. |

**Who:** **Author** (own consumption only for current workspace) or **TENANT** (org admin) / SUPER_ADMIN (all users in workspace; optional workspaceId).

**Response (data object):**

| Field | Type | Description |
|-------|------|-------------|
| period | object | `{ startDate, endDate }` |
| byFeature | array | One entry per feature with consumption in the period |
| totalConsumption | number | Sum of all consumption (negative = credits used) |

**For SUPER_ADMIN responses only (optional):** Include split totals: `totalActualCost`, `totalCapFee`, `totalDeducted` (mirrors transaction logic; org users see only totalConsumption / totalDeducted).

**byFeature item:**

| Field | Type | Description |
|-------|------|-------------|
| feature | string | Display name, e.g. "AI Course Generation", "AI Image Creation", "AI Text Enhancement", "AI Video Generation" |
| consumption | number | Credits/cost consumed in period (negative number, e.g. -250) |
| requestCount | number | Optional: number of requests for this feature in the period |

**Example response:**

```json
{
  "success": true,
  "data": {
    "period": {
      "startDate": "2024-01-01T00:00:00",
      "endDate": "2024-01-31T23:59:59"
    },
    "byFeature": [
      { "feature": "AI Course Generation", "consumption": -250, "requestCount": 12 },
      { "feature": "AI Image Creation", "consumption": -25, "requestCount": 5 },
      { "feature": "AI Text Enhancement", "consumption": -45, "requestCount": 8 },
      { "feature": "AI Video Generation", "consumption": -256, "requestCount": 3 }
    ],
    "totalConsumption": -576
  }
}
```

**Implementation notes:**

- Query `api_usage` for the workspace and date range; group by `feature` (or derived feature); sum `cost` per feature; return consumption as negative (e.g. `-sum(cost)` or in a "credits" unit if you define one).
- If `feature` is null on a row, group under "Other" or map from `api_type` (e.g. text → "AI Text Enhancement", image → "AI Image Creation") per product rules.
- Use display names in response (e.g. "AI Course Generation") via a small mapping from stored feature code.

---

## I.10 Transaction History API

**Purpose:** A complete history of AI credit consumption for the selected period (for "Transaction History" UI: Feature Used, Date & Time, Used By, Consumption).

**Endpoint:** `GET /api/usage/transaction-history`

**Query parameters:**

| Parameter | Type | Required | Description |
|------------|------|----------|-------------|
| startDate | ISO 8601 datetime | Yes | Start of period |
| endDate | ISO 8601 datetime | Yes | End of period |
| page | int | No | Page number (0-based); default 0 |
| size | int | No | Page size; default 20 |
| workspaceId | UUID | No | Workspace scope; if omitted, use current user's workspace. Admin may pass any workspace. |

**Who:** **Author** (own transaction history only) or **TENANT** (org admin) / SUPER_ADMIN (all users in workspace; optional workspaceId).

**Response (data):** Paginated list of transaction items. Each item:

| Field | Type | Description |
|-------|------|-------------|
| id | string | api_usage id (optional, for debugging) |
| featureUsed | string | Display name, e.g. "AI Course Generation", "AI Image Creation", "AI Text Enhancement", "AI Video Generation" |
| dateTime | string | Formatted date/time, e.g. "12 Jan 2024, 10:30 AM" (locale-friendly; backend can return ISO and let front-end format) |
| usedBy | string | User display name (e.g. firstName + lastName from users table), or "System" if user_id null or user not found |
| consumption | number | Credits/cost for this record (negative, e.g. -120) |

**Example response:**

```json
{
  "success": true,
  "data": [
    { "featureUsed": "AI Course Generation", "dateTime": "2024-01-12T10:30:00", "usedBy": "Ravi Ranjan", "consumption": -120 },
    { "featureUsed": "AI Image Creation", "dateTime": "2024-01-11T16:15:00", "usedBy": "Ravi Ranjan", "consumption": -15 },
    { "featureUsed": "AI Text Enhancement", "dateTime": "2024-01-10T18:45:00", "usedBy": "System", "consumption": -25 },
    { "featureUsed": "AI Video Generation", "dateTime": "2024-01-09T14:10:00", "usedBy": "Ravi Ranjan", "consumption": -180 }
  ],
  "count": 8
}
```

**Implementation notes:**

- Query `api_usage` for the workspace and date range; order by `created_at` DESC; paginate.
- Join with `users` (by user_id) to get firstName, lastName for "Used By"; if user_id null or user missing, use "System".
- Map `feature` (or api_type) to display name (e.g. "AI Course Generation").
- Consumption = negative cost (or negative credits if you use a credits unit). Format dateTime as ISO or locale string per front-end need.
- Optional: return both `dateTime` (ISO) and `dateTimeDisplay` (e.g. "12 Jan 2024, 10:30 AM") if backend does formatting.

---

## I.11 Recording usage with feature

When recording usage (ApiUsageService.recordUsage or recordUsageForCurrentUser), accept an optional **feature** parameter so callers can tag the usage (e.g. course generation flow passes `COURSE_GENERATION`, image flow passes `IMAGE_CREATION`). If not passed, leave null; Consumption Summary and Transaction History can still show rows under "Other" or derive from `api_type`.

**Suggested feature codes:** `COURSE_GENERATION`, `IMAGE_CREATION`, `TEXT_ENHANCEMENT`, `VIDEO_GENERATION` (and optionally `OTHER`). Store in `api_usage.feature`; map to display names in API responses.

**Feature enum governance:** Feature codes must be defined in a shared enum or config (e.g. `ApiUsageFeature` enum). New features must be added there before usage is recorded. This avoids typos, inconsistent grouping, and bugs like "AI_COURSE" vs "COURSE_GENERATION".

---

# Part II: Organization Balance

## II.1 Overview

Organization Balance adds a **prepaid balance per workspace (organization)**. When a user in that workspace uses AI APIs, the **cost already recorded in `api_usage`** is deducted from the workspace balance. Users can **top up** balance; optionally, **free credits** are granted on workspace creation only. No grant-credit API or coupon system in current scope (coupons are future).

## II.2 Goals (current scope)

1. **Balance per organization** — One balance (USD) per workspace.
2. **Top-up** — Add credit to workspace balance (e.g. admin).
3. **Deduction on API use** — Deduct `api_usage.cost` from workspace balance after recording usage.
4. **Remaining balance** — APIs to read balance; optionally block API calls when balance <= 0.
5. **Audit trail** — All balance changes (top-up, deduction) in `workspace_balance_transaction`.
6. **Free credits (no API)** — Optional one-time credit **on workspace creation only** (e.g. by workspace type).
7. **AI usage cap — deduction surcharge (org-level)** — Configurable surcharge % (e.g. 10%) on each deduction: deduct actual_cost + cap_fee (e.g. $1 + 10% = $1.10). **Super admin** sees split (actual + cap) in all reports; **org users** see only total consumed ($1.10) in all reports (see § II.3 workspace_ai_usage_cap).

Out of scope for now: grant-credit APIs; coupon codes (see § Future: Coupon system).

## II.3 Data model — Balance

### Table: `workspace_balance`

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR(36) PK | UUID |
| workspace_id | VARCHAR(36) | FK workspaces(id), UNIQUE, NOT NULL |
| balance_usd | DECIMAL(12,6) | Current balance USD, default 0 |
| currency | VARCHAR(3) | e.g. "USD" |
| updated_at | DATETIME(6) | Last change |
| updated_by | VARCHAR(36) | Optional |

**Index:** `workspace_id` (unique). Enables row-level locking when updating balance.

### Table: `workspace_balance_transaction`

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR(36) PK | UUID |
| workspace_id | VARCHAR(36) | FK workspaces(id), NOT NULL |
| type | VARCHAR(30) | TOP_UP, API_DEDUCTION, ADJUSTMENT, FREE_CREDIT (future: COUPON_REDEMPTION) |
| amount_usd | DECIMAL(12,6) | Positive for credit, negative for deduction |
| balance_before | DECIMAL(12,6) | Balance before this tx |
| balance_after | DECIMAL(12,6) | Balance after this tx |
| reference_type | VARCHAR(50) | e.g. "API_USAGE", "MANUAL", "COUPON" |
| reference_id | VARCHAR(36) | e.g. api_usage.id for deductions |
| user_id | VARCHAR(36) | User who triggered or made the call |
| description | VARCHAR(500) | Optional note |
| actual_cost_usd | DECIMAL(12,6) | **For API_DEDUCTION only.** Actual API cost (e.g. $1.00). Enables super admin to see split. |
| cap_fee_usd | DECIMAL(12,6) | **For API_DEDUCTION only.** Surcharge (e.g. 10% of actual = $0.10). Total deducted = actual_cost_usd + cap_fee_usd = amount_usd (absolute). |
| created_at | DATETIME(6) | NOT NULL |

**Indexes:** `workspace_id`, `created_at`, `(workspace_id, created_at)`.

**For API_DEDUCTION:** amount_usd = -(actual_cost_usd + cap_fee_usd). Org users see only |amount_usd| (e.g. $1.10) in all reports; super admin sees actual_cost_usd ($1.00) and cap_fee_usd ($0.10) split.

**Idempotency (API deductions):** Enforce at DB level so the same API usage is never deducted twice. Use a **partial unique index** (PostgreSQL, SQL Server):

```sql
UNIQUE (reference_type, reference_id) WHERE reference_type = 'API_USAGE'
```

(Or equivalent: `CREATE UNIQUE INDEX uq_balance_tx_api_usage ON workspace_balance_transaction (reference_type, reference_id) WHERE reference_type = 'API_USAGE';`)

This prevents double deduction on retries and race conditions. *MySQL:* MySQL does not support partial unique indexes; enforce idempotency in application code (e.g. check for an existing row with `reference_type = 'API_USAGE'` and same `reference_id` before insert), or use a non-partial unique index on `(reference_type, reference_id)` and ensure `reference_id` is non-null only for API_USAGE rows.

**Conventions:** TOP_UP / FREE_CREDIT: amount_usd > 0. API_DEDUCTION: amount_usd < 0, reference_type = 'API_USAGE', reference_id = api_usage.id.

### Table: `feature_balance_threshold`

Stores **OK | WARN | BLOCK** threshold amounts (in USD) per feature. Used for feature-specific low-credit warnings and blocking.

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR(36) PK | UUID |
| feature | VARCHAR(80) | Feature code (same as `ApiUsageFeature` / `api_usage.feature`): e.g. `COURSE_GENERATION`, `TEXT_ENHANCEMENT`, `IMAGE_CREATION`, `VIDEO_GENERATION`. UNIQUE. |
| warn_below_usd | DECIMAL(10,4) | If balance ≤ this value → status **WARN** (show warning, allow call). Must be ≥ block_below_usd. |
| block_below_usd | DECIMAL(10,4) | If balance ≤ this value → status **BLOCK** (reject call with 402/403). |
| display_name | VARCHAR(120) | Optional. Display name for messages, e.g. "AI Image Creation". |
| is_active | BOOLEAN | Default true. Inactive rows are ignored (fallback: use app default or block). |
| created_at | DATETIME(6) | NOT NULL |
| updated_at | DATETIME(6) | NOT NULL |

**Index:** `feature` (unique), `is_active`.

**Status rules (evaluate in order):**

| Condition | Status | Behavior |
|-----------|--------|----------|
| balance_usd ≤ block_below_usd | **BLOCK** | Return 402/403; do not call AI. |
| balance_usd ≤ warn_below_usd | **WARN** | Allow call; return lowCreditStatus so UI can show warning. |
| balance_usd > warn_below_usd | **OK** | Proceed normally. |

**Example rows (production-oriented defaults):**

| feature | warn_below_usd | block_below_usd | display_name |
|---------|----------------|-----------------|--------------|
| TEXT_ENHANCEMENT | 1.50 | 0.25 | AI Text Enhancement |
| COURSE_GENERATION | 8.00 | 2.00 | AI Course Generation |
| IMAGE_CREATION | 4.00 | 1.00 | AI Image Creation |
| VIDEO_GENERATION | 15.00 | 5.00 | AI Video Generation |

**Production rationale (ideal numbers for real users):**

| Feature | Typical cost per use | block_below_usd | warn_below_usd | Rationale |
|---------|----------------------|-----------------|---------------|-----------|
| **TEXT_ENHANCEMENT** | ~$0.002–0.01 per call (gpt-4o-mini) | 0.25 | 1.50 | Block only when balance can't cover ~25–50 calls; warn when ~$1.50 left so user can top up before hitting block. |
| **COURSE_GENERATION** | ~$0.10–0.40 per full run (multiple LLM calls) | 2.00 | 8.00 | Block so user can't start if they can't complete one run; warn at $8 to allow time to top up. |
| **IMAGE_CREATION** | ~$0.02–0.04 per image (DALL·E 2/3) | 1.00 | 4.00 | Block when balance can't cover ~25–50 images; warn at $4. |
| **VIDEO_GENERATION** | ~$0.10–0.50 per second (Sora) | 5.00 | 15.00 | Don't start if balance &lt; $5 (short clip); warn at $15 so user tops up before expensive runs. |

Adjust per your actual usage (e.g. average tokens per course run, image size) and business policy. These values are safe starting points for production.

**Resolution:** Load threshold by `feature` where `is_active = true`.

**Feature fallback rule (explicit):** If `feature_balance_threshold` row is **missing or inactive** for a feature → fallback to **application default** (e.g. from config) **OR** treat as **BLOCK below 0**. Document the chosen behavior so implementers do not guess. Optional future: per-workspace-type or per-workspace override table (e.g. `workspace_type_balance_threshold`) for different thresholds by plan.

**Migration:** e.g. `Vxx__Create_feature_balance_threshold_table.sql`; seed initial rows for each feature.

### Free credits (no new table)

Free credits use the same `workspace_balance` and `workspace_balance_transaction`; type = FREE_CREDIT, amount_usd > 0, granted only on workspace creation (e.g. from WorkspaceTypeConfig.initial_credit_usd or app property).

### AI usage cap — deduction surcharge (organization-level, configurable %)

**Goal:** A **surcharge % on each deduction** (e.g. 10% cap). User has $5, uses $1 for API → **deduct $1 + 10% = $1.10**. **Super admin** sees the **split** (actual $1.00 + cap $0.10); **org users** see only **$1.10 consumed** in all reports and data.

#### Table: `workspace_ai_usage_cap`

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR(36) PK | UUID |
| workspace_id | VARCHAR(36) | FK workspaces(id), UNIQUE, NOT NULL |
| cap_percent | DECIMAL(5,2) | e.g. 10.00 for 10% surcharge on each deduction. NULL or row absent = no surcharge. |
| is_active | BOOLEAN | Default true. Inactive = no surcharge applied. |
| created_at | DATETIME(6) | NOT NULL |
| updated_at | DATETIME(6) | NOT NULL |

**Index:** `workspace_id` (unique), `is_active`.

**Deduction formula:** For each API usage: **actual_cost** = api_usage.cost (e.g. $1.00). **cap_fee** = actual_cost × (cap_percent / 100) if workspace has active cap (e.g. 10% → $0.10). **Total deducted** = actual_cost + cap_fee = $1.10. Balance is reduced by $1.10.

**Visibility (who sees what):**

| Role | What they see |
|------|----------------|
| **SUPER_ADMIN** | **Split:** In every report (consumption summary, transaction history, balance transactions), sees **actual_cost_usd** (e.g. $1.00) and **cap_fee_usd** (e.g. $0.10) and total deducted ($1.10). Aggregates: total actual, total cap fee, total consumed. |
| **Organization-level users** | **Total only:** In all reports and data they see only **total consumed** = total deducted (e.g. $1.10 per row). No split; they do not see actual vs cap. |

**Storage of split:** In `workspace_balance_transaction` for type API_DEDUCTION: **amount_usd** = -total_deducted (e.g. -1.10); **actual_cost_usd** = 1.00; **cap_fee_usd** = 0.10. Org users are shown only |amount_usd| (1.10); super admin is shown actual_cost_usd and cap_fee_usd from the same row.

**APIs:**

- **Consumption summary / transaction history / balance transactions:** Backend varies response by caller role. **Org users:** each row and totals use only **amount_usd** (total deducted, e.g. 1.10). **Super admin:** each row and totals include **actual_cost_usd**, **cap_fee_usd**, and total (amount_usd).
- **Manage cap (super admin only):** GET/PUT `/api/billing/workspace/{workspaceId}/usage-cap` — get or set **cap_percent** (e.g. 10) for that workspace. List: GET `/api/billing/usage-caps`.

**Migration:** Create `workspace_ai_usage_cap` (workspace_id, cap_percent, is_active, created_at, updated_at only). When creating `workspace_balance_transaction`, include **actual_cost_usd** and **cap_fee_usd** (nullable, for API_DEDUCTION).

## II.4 Components (balance)

| Component | Purpose |
|-----------|---------|
| **WorkspaceBalance** (entity) | Maps to `workspace_balance` |
| **WorkspaceBalanceTransaction** (entity) | Maps to `workspace_balance_transaction` |
| **WorkspaceBalanceRepository** | Find by workspaceId; optional lock for update (e.g. FOR UPDATE) |
| **WorkspaceBalanceTransactionRepository** | Save; find by workspaceId, date range, pagination |
| **FeatureBalanceThreshold** (entity) | Maps to `feature_balance_threshold` |
| **FeatureBalanceThresholdRepository** | Find by feature (and is_active); optional: list all for admin UI. |
| **WorkspaceAiUsageCap** (entity) | Maps to `workspace_ai_usage_cap` (cap_percent only). |
| **WorkspaceAiUsageCapRepository** | Find by workspaceId; list all for SUPER_ADMIN. |
| **BillingBalanceService** | getBalance(workspaceId), topUp(...), **deduct(workspaceId, actualCost, ...)** — computes cap_fee from workspace cap_percent, deducts actual_cost + cap_fee, stores actual_cost_usd and cap_fee_usd in transaction; getLowCreditStatus(workspaceId, feature); addFreeCredit(...). Uses locking and writes both balance and transaction. |
| **Integration in ApiUsageService** | After saving an `api_usage` row, call BillingBalanceService.deduct(workspaceId, api_usage.cost, ...). Before AI call: optionally getLowCreditStatus(workspaceId, feature); if BLOCK return 402/403. |
| **Integration on workspace creation** | If config entitles free credits (e.g. by type), call addFreeCredit after creating workspace. |
| **BillingBalanceController** | GET balance, POST top-up, GET transactions; GET low-credit-status?feature= (or extended balance with ?feature=). |
| **DB migrations** | Create `workspace_balance`, `workspace_balance_transaction`, **`feature_balance_threshold`**, **`workspace_ai_usage_cap`**; optional backfill. |

## II.5 Balance flows

### Get remaining balance

- Input: workspaceId (from JWT or path). Load `workspace_balance`; if missing, treat as 0 (or create row with 0).
- Output: balance_usd, optionally currency, updated_at.
- Who: Workspace user or BILLING_ADMIN/SUPER_ADMIN for any workspace.

### Top-up

- Input: workspaceId, amount_usd > 0, optional description.
- Logic: Lock workspace_balance row; **balance_after = balance_before + amount_usd**; update balance; insert transaction type TOP_UP; commit.
- **When balance is negative:** The same formula applies. The top-up **repays the negative first** — e.g. balance -$5, top-up +$10 → balance becomes $5. So the user effectively "pays off" the negative with the next top-up; no separate deduction step.
- Who: BILLING_ADMIN, SUPER_ADMIN (optionally workspace billing role later).

### Deduction on API use

- When: Immediately after persisting `api_usage` (same or next transaction).
- Logic: If workspace_id null, skip. **Actual cost** = api_usage.cost (e.g. $1.00). **Cap fee** = if workspace has active `workspace_ai_usage_cap` (cap_percent, e.g. 10), then actual_cost × (cap_percent / 100); else 0. **Total deducted** = actual_cost + cap_fee (e.g. $1.00 + $0.10 = $1.10). Lock workspace_balance; **allow balance to go negative** — balance_after = balance_before - total_deducted (may be negative). Update balance; insert API_DEDUCTION with amount_usd = -total_deducted, reference_type='API_USAGE', reference_id=api_usage.id, **actual_cost_usd** = actual cost, **cap_fee_usd** = cap fee; commit. **User sees negative balance** in UI; it is **repaid from the next top-up** (see § Managing negative balance).
- **Transaction boundaries:** Deduction and balance update must occur in the **same DB transaction** (or with row-level locking) to prevent race conditions.
- Idempotency: Use api_usage.id as reference_id so the same usage is never deducted twice.

### Check balance before AI call (required) — warn and block

- **Where:** Before calling OpenAIProvider (or other AI). This check is **required** so users are warned when balance is low and blocked when insufficient.
- **Logic:**
  1. Resolve current user's workspace_id and the **feature** for this request (e.g. COURSE_GENERATION, IMAGE_CREATION).
  2. Call **BillingBalanceService.getLowCreditStatus(workspaceId, feature)** → returns status **OK** \| **WARN** \| **BLOCK**, plus balanceUsd, warnBelowUsd, blockBelowUsd, displayName.
  3. **If status = BLOCK:** Return 402/403 with feature-specific message (e.g. "Insufficient balance for AI Image Creation. Minimum required: $2.00. Please top up."). Do **not** call the provider or record usage.
  4. **If status = WARN:** Allow the call; include **lowCreditStatus: "WARN"** (and balanceUsd, warnBelowUsd, blockBelowUsd, displayName) in the response so the **UI can warn the user** (e.g. banner: "Your balance is low. Consider topping up soon.").
  5. **If status = OK:** Proceed normally.
- **Optional:** Before step 2, if you support estimated cost for the request, check balance >= estimated cost; if not, return 402 immediately (so the request cannot complete).
- **Result:** User is **warned** when balance is in WARN range and **blocked** when in BLOCK range (e.g. balance ≤ block_below_usd). Balance **may go negative** if allowed by thresholds (e.g. block_below_usd is negative); user sees negative balance and it is **deducted from the next top-up** (see § Managing negative balance).

### Feature-specific low-credit warnings

- Goal: Allow **feature-specific thresholds** so cheaper features (e.g. text) can still work with low balance, while more expensive ones (e.g. images, video) warn or block earlier.
- **Source of thresholds:** Database table `feature_balance_threshold` (see § II.3). One row per feature with `warn_below_usd` and `block_below_usd`. Optional fallback: application config if no row exists.
- Backend check (when a feature flow starts; before calling AI):
  - Inputs: `workspaceId`, `feature` (e.g. `COURSE_GENERATION`), current `balance_usd`.
  - Load threshold: `SELECT warn_below_usd, block_below_usd, display_name FROM feature_balance_threshold WHERE feature = ? AND is_active = true`. If no row, use config defaults or treat as BLOCK when balance ≤ 0.
  - Evaluate (same rules as table above):
    - If `balance_usd <= block_below_usd` → **BLOCK**: return 402/403 with feature-specific message, e.g. `"Insufficient balance for AI Image Creation. Minimum required: $2.00"` (use `display_name` and `block_below_usd` from row).
    - Else if `balance_usd <= warn_below_usd` → **WARN**: allow the call; return `lowCreditStatus: "WARN"`, `feature`, `balanceUsd`, `warnBelowUsd`, `blockBelowUsd` so the UI can show a non-blocking warning.
    - Else → **OK**: proceed normally.
- Where to expose this:
  - Option A: Extend `GET /api/billing/balance` with optional query param `feature`; response includes `lowCredit: { status, warnBelowUsd, blockBelowUsd }` when feature is passed.
  - Option B: Dedicated endpoint `GET /api/billing/low-credit-status?feature=COURSE_GENERATION` returning `{ balanceUsd, feature, status: "OK"|"WARN"|"BLOCK", warnBelowUsd, blockBelowUsd, displayName }`.
  - Front-end calls this when a feature screen loads and before expensive actions, to show **feature-specific** low-credit warnings.

### Free credits (workspace creation only)

- When: In workspace creation flow (e.g. WorkspaceService or ClientAdminService after persisting workspace).
- Logic: If workspace type (or config) entitles free credits (e.g. TRIAL $5), ensure workspace_balance row exists, then add amount and insert transaction type FREE_CREDIT.
- Config: e.g. WorkspaceTypeConfig.initial_credit_usd or application property (e.g. billing.free-credits.trial=5).

## II.6 Balance API endpoints

| Method | Endpoint | Description | Who |
|--------|----------|-------------|-----|
| GET | `/api/billing/balance` | Current workspace balance (optional ?feature= for lowCredit in response) | Author, **TENANT** (same workspace balance) |
| GET | `/api/billing/balance/workspace/{workspaceId}` | Balance for a workspace | **TENANT** (org admin), SUPER_ADMIN |
| GET | `/api/billing/low-credit-status?feature=` | OK \| WARN \| BLOCK for feature; returns balanceUsd, status, warnBelowUsd, blockBelowUsd, displayName | Author, **TENANT** |
| POST | `/api/billing/top-up` | Add credit (body: amountUsd, optional workspaceId) | **TENANT** (org admin), SUPER_ADMIN |
| GET | `/api/billing/transactions` | Paginated balance transactions for current workspace | **Author** (own rows only) or **TENANT** (all users) — see Visibility below |
| GET | `/api/billing/transactions/workspace/{workspaceId}` | All balance transactions for workspace | **TENANT** (org admin), SUPER_ADMIN |
| GET | `/api/billing/workspace/{workspaceId}/usage-cap` | Get deduction surcharge % (cap_percent) for workspace | SUPER_ADMIN |
| PUT | `/api/billing/workspace/{workspaceId}/usage-cap` | Set deduction surcharge % (cap_percent, e.g. 10) for workspace | SUPER_ADMIN |
| GET | `/api/billing/usage-caps` | List workspaces with cap config | SUPER_ADMIN |

Optional (admin): GET/PUT `/api/billing/thresholds` or per-feature CRUD to manage `feature_balance_threshold` rows. No grant-credit or coupon endpoints in current scope.

**Visibility by role (balance):**

| Role | What they see |
|------|----------------|
| **Author** | **Only own data:** Workspace balance (shared); **balance transactions** filtered by **user_id = current user** (only rows they triggered, e.g. their deductions). Usage/consumption: own only (see Part I). |
| **TENANT (org admin)** | **All users in workspace:** Workspace balance; **all** balance transactions for the workspace (all users); consumption summary and transaction history for **all users** in workspace. Can top-up; can call balance/workspace/{id} for their workspace(s). *Currently org admin = TENANT role only.* |
| **SUPER_ADMIN** | All workspaces; balance and transactions for any workspaceId; usage-cap config; top-up. |

**Implementation:** For `/api/billing/transactions` (current workspace), if caller is **author** → filter by `user_id = current user`. If caller is **TENANT** (org admin) or SUPER_ADMIN → return all transactions for the workspace.

## II.7 Edge cases (balance)

- **New workspace:** Create workspace_balance row with 0 on creation or on first top-up; document choice.
- **Concurrent updates:** Lock workspace_balance row (e.g. SELECT ... FOR UPDATE) when updating.
- **Refunds/adjustments:** Transaction type ADJUSTMENT; admin-only flow; same pattern (lock, update balance, insert transaction).
- **Free credits:** On workspace creation only; ensure balance row exists before adding credit.

### Managing negative balance

- **Allow negative balance:** Balance **may go negative** when usage is deducted and balance is low or zero. **User sees negative balance** in the UI (e.g. "Balance: -$5.00").
- **Repaid from next top-up:** When the user (or admin) tops up, **balance_after = balance_before + amount_usd**. So the negative is **repaid first** — e.g. balance -$5, top-up +$10 → balance becomes $5. No separate "deduct from top-up" step; the same top-up flow applies. The next top-up effectively **deducts** the negative from the added amount.
- **Check balance before AI call:** Still apply WARN (warn user when balance is low) and BLOCK when balance ≤ block_below_usd (feature-specific). If block_below_usd is set to allow some negative (e.g. -$10), then AI is allowed until balance < -$10.
- **Optional:** Enforce a **max negative limit** (e.g. -$50) per workspace; when balance &lt; that limit, block new AI calls until top-up. Config: e.g. `billing.max-negative-balance-usd`.
- **Admin:** SUPER_ADMIN / BILLING_ADMIN can **adjust** balance via ADJUSTMENT transaction (e.g. write-off or correction).

---

# Part III: End-to-end flow (usage + balance)

## III.1 How the two systems work together

1. **Single source of truth for cost and workspace** — API Usage Tracking records each AI call in `api_usage` with `workspace_id`, `user_id`, and `cost` (from ApiPricingService). Organization Balance does not recalculate cost; it uses `api_usage.cost` and `api_usage.workspace_id` for deduction.

2. **Request flow (with balance check and warn/block):**
   - Resolve current user's workspace_id and **feature** for this request.
   - **Check balance:** Call BillingBalanceService.getLowCreditStatus(workspaceId, feature). If status = **BLOCK**, return 402/403 and do not call provider. If status = **WARN**, allow but include lowCreditStatus in response so UI can **warn** the user.
   - Call AI provider (e.g. OpenAIProvider).
   - Provider returns; extract usage and compute cost (or use existing ApiPricingService).
   - Save `api_usage` row (ApiUsageService) with workspace_id, user_id, cost, tokens.
   - Call BillingBalanceService.deduct(workspaceId, cost, userId, "API_USAGE", apiUsage.id).
   - Return response to client (including lowCreditStatus when WARN).

3. **Managing negative balance:** Balance may go negative; user sees it. Next top-up repays it (balance_after = balance_before + top_up). See § II.7 Managing negative balance.

## III.2 Integration points

- **Where usage is recorded:** ApiUsageService.recordUsage / recordUsageForCurrentUser (and any caller that writes api_usage). After saving the row, call BillingBalanceService.deduct(workspaceId, cost, userId, "API_USAGE", apiUsageId).
- **Where to check balance (optional):** Before invoking the AI provider (e.g. in the controller or service that calls OpenAIProvider). Use BillingBalanceService.getBalance(workspaceId).

---

# Combined API reference

All endpoints in one place. All require JWT unless stated otherwise.

| Area | Method | Endpoint | Description | Who |
|------|--------|----------|-------------|-----|
| **Usage** | GET | `/api/usage/stats` | Current user usage stats (own only) | **Author** |
| **Usage** | GET | `/api/usage/history` | Paginated usage history (own only) | **Author** |
| **Usage** | GET | `/api/usage/stats/range` | User stats by date range (own only) | **Author** |
| **Usage** | GET | `/api/usage/stats/user/{userId}` | User stats | SUPER_ADMIN or that user |
| **Usage** | GET | `/api/usage/stats/workspace/{workspaceId}` | Workspace usage stats (all users) | **TENANT** (org admin), SUPER_ADMIN |
| **Balance** | GET | `/api/billing/balance` | Current workspace balance | Author, **TENANT** |
| **Balance** | GET | `/api/billing/balance/workspace/{workspaceId}` | Balance for workspace | **TENANT** (org admin), SUPER_ADMIN |
| **Balance** | POST | `/api/billing/top-up` | Add credit to workspace | **TENANT** (org admin), SUPER_ADMIN |
| **Balance** | GET | `/api/billing/transactions` | Balance transactions (own only vs all by role) | **Author** (own) / **TENANT** (all) |
| **Balance** | GET | `/api/billing/transactions/workspace/{workspaceId}` | All balance transactions for workspace | **TENANT** (org admin), SUPER_ADMIN |
| **Balance** | GET / PUT | `/api/billing/workspace/{workspaceId}/usage-cap` | Get / set deduction surcharge % (cap_percent) for workspace | SUPER_ADMIN |
| **Balance** | GET | `/api/billing/usage-caps` | List workspaces with cap config | SUPER_ADMIN |

---

# Combined implementation checklist

## API Usage Tracking (Part I)

- [ ] Table `api_usage` (and entity, repository).
- [ ] ApiPricingService with model pricing; calculateTextCost, calculateImageCost.
- [ ] ApiUsageService: recordUsage, recordUsageForCurrentUser; getUserUsageStats, getCurrentUserUsageStats, getWorkspaceUsageStats, getUserUsageHistory, getUserUsageStatsByDateRange.
- [ ] OpenAIProvider (and other providers): after each call, extract usage and call ApiUsageService.recordUsageForCurrentUser (or equivalent).
- [ ] ApiUsageController: GET /api/usage/stats, history, stats/range, stats/user/{id}, stats/workspace/{id}.
- [ ] Migration for api_usage.

## Organization Balance (Part II)

- [ ] Table `workspace_balance` (and entity, repository).
- [ ] Table `workspace_balance_transaction` (and entity, repository).
- [ ] BillingBalanceService: getBalance, topUp, deduct (using cost from api_usage), addFreeCredit for workspace-creation only (with locking and transaction log).
- [ ] Integration: after saving api_usage, call deduct(workspaceId, cost, userId, "API_USAGE", apiUsageId).
- [ ] **Check balance before AI call (required):** getLowCreditStatus(workspaceId, feature); if BLOCK return 402/403; if WARN allow but return lowCreditStatus so UI can **warn** the user.
- [ ] **Managing negative balance:** Allow negative balance; user sees it in UI; next top-up repays it (balance_after = balance_before + amount_usd); optional max negative limit; admin adjust. See § II.7 Managing negative balance.
- [ ] Optional: Free credits on workspace creation (e.g. WorkspaceTypeConfig.initial_credit_usd or app property).
- [ ] BillingBalanceController: GET balance, POST top-up, GET transactions only.
- [ ] **Authorization and visibility:** **Author** sees only **own** usage, consumption summary, transaction history, and own balance transaction rows; **TENANT (org admin)** sees **all users'** data in the workspace. *Currently org admin = TENANT role only.* Balance read for workspace members; top-up and admin read for TENANT / SUPER_ADMIN.
- [ ] Migrations for workspace_balance, workspace_balance_transaction, and **feature_balance_threshold** (with seed rows for each feature); optional backfill.
- [ ] Low-credit: load thresholds from feature_balance_threshold; getLowCreditStatus(workspaceId, feature); block/warn per status; optional admin CRUD for thresholds.
- [ ] **AI usage cap (deduction surcharge):** Table `workspace_ai_usage_cap` (workspace_id, cap_percent, is_active only). On deduct: total_deducted = actual_cost + (actual_cost × cap_percent/100); store actual_cost_usd and cap_fee_usd in workspace_balance_transaction for API_DEDUCTION. **SUPER_ADMIN** sees split (actual + cap) in consumption summary, transaction history, balance transactions; **org users** see only total consumed (amount_usd) in all reports. GET/PUT usage-cap (SUPER_ADMIN only). Migration for workspace_ai_usage_cap; workspace_balance_transaction includes actual_cost_usd, cap_fee_usd.

---

# Future: Coupon system

Planned for a later phase. Summary:

- **Tables:** `coupon` (code, credit_amount_usd, max_redemptions, redemption_count, max_per_workspace, valid_from, valid_until, is_active, etc.), `coupon_redemption` (coupon_id, workspace_id, user_id, credit_amount_usd, redeemed_at).
- **Transaction type:** COUPON_REDEMPTION; reference_type = 'COUPON', reference_id = coupon_redemption.id.
- **Flow:** User submits code → validate (active, in date, under limits, per-workspace limit) → add credit to workspace balance → insert balance transaction + coupon_redemption → increment coupon.redemption_count.
- **Endpoints (future):** POST /api/billing/coupon/redeem (body: code); optional admin CRUD for coupons.
- **Edge cases:** Code normalized (e.g. uppercase); unique code; expiry; max_per_workspace (e.g. 1 = one redemption per workspace).

---

# Configuration & troubleshooting

## Configuration

- **OpenAI / AI:** Use existing config (e.g. ai.text.openai.api.url, api.key, model). ApiPricingService can use same model names for pricing.
- **Pricing updates:** Adjust ApiPricingService (e.g. initializePricing() or configurable properties).
- **Balance:** Optional app properties e.g. billing.free-credits.trial=5; WorkspaceTypeConfig.initial_credit_usd if added.

## Troubleshooting

### Usage not recorded

- Verify ApiUsageService is injected and called after provider response.
- Check OpenAI response includes `usage` object.
- Ensure user/workspace context (e.g. AuthUtil) is available when recording.

### Incorrect costs

- Verify model name matches ApiPricingService pricing keys.
- Confirm token counts (or image count) passed correctly to pricing methods.

### Balance not deducting

- Ensure deduct() is called after api_usage is saved and that workspace_id and cost are passed correctly.
- Check workspace_balance row exists (create with 0 if needed).
- Verify reference_id = api_usage.id to avoid double deduction.

### Balance endpoint errors

- Confirm user has access to workspace (or is BILLING_ADMIN/SUPER_ADMIN).
- Check workspace_balance and workspace_balance_transaction tables and migrations.

---

**Last updated:** February 2026  
**Version:** 1.0 (combined)

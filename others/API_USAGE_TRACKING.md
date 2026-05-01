# API Usage Tracking System Documentation

## Overview

The API Usage Tracking System automatically monitors and records OpenAI API usage (tokens and costs) for each user in the application. This system provides comprehensive tracking of API consumption, cost calculation, and usage statistics to help manage API expenses and provide insights into user behavior.

## Features

- **Automatic Tracking**: Token usage is automatically recorded when OpenAI API calls are made
- **Cost Calculation**: Real-time cost calculation based on OpenAI's official pricing
- **User-Level Tracking**: Individual usage tracking per user
- **Workspace-Level Tracking**: Aggregate usage statistics per workspace
- **Model Breakdown**: Detailed statistics grouped by AI model
- **Date Range Queries**: Filter usage by specific date ranges
- **Pagination Support**: Efficient retrieval of usage history with pagination

## Architecture

### Components

#### 1. **ApiUsage Entity** (`entity/ApiUsage.java`)
The core entity that stores API usage records in the database.

**Fields:**
- `id`: Unique identifier (UUID)
- `userId`: User who made the API call
- `workspaceId`: Workspace context (optional)
- `provider`: AI provider name (e.g., "openai", "gemini")
- `model`: Model name (e.g., "gpt-4o-mini", "dall-e-2")
- `apiType`: Type of API call (e.g., "text", "image")
- `promptTokens`: Number of tokens in the prompt
- `completionTokens`: Number of tokens in the completion
- `totalTokens`: Total tokens used
- `cost`: Calculated cost in USD
- `requestId`: Optional OpenAI request identifier
- `endpoint`: API endpoint used
- `createdAt`: Timestamp when usage was recorded

**Database Table:** `api_usage`
- Indexed on: `user_id`, `created_at`, `provider+model`, `workspace_id`

#### 2. **ApiUsageRepository** (`repository/ApiUsageRepository.java`)
Repository interface for database operations.

**Key Methods:**
- `findByUserIdOrderByCreatedAtDesc()`: Get user's usage history
- `findByWorkspaceIdOrderByCreatedAtDesc()`: Get workspace usage history
- `getTotalTokensByUserId()`: Calculate total tokens for a user
- `getTotalCostByUserId()`: Calculate total cost for a user
- `getUsageStatsByModelForUser()`: Get statistics grouped by model
- Date range query methods for filtering by time periods

#### 3. **ApiPricingService** (`service/ApiPricingService.java`)
Service for calculating API costs based on model pricing.

**Supported Models:**
- **Text Models:**
  - `gpt-4o-mini`: $0.150 per 1M input tokens, $0.600 per 1M output tokens
  - `gpt-4o`: $2.500 per 1M input tokens, $10.000 per 1M output tokens
  - `gpt-4-turbo`: $10.000 per 1M input tokens, $30.000 per 1M output tokens
  - `gpt-3.5-turbo`: $0.500 per 1M input tokens, $1.500 per 1M output tokens

- **Image Models:**
  - `dall-e-2`: $0.020 per image (1024x1024)
  - `dall-e-3`: $0.040 per image (1024x1024 standard)

**Key Methods:**
- `calculateTextCost()`: Calculate cost for text generation based on tokens
- `calculateImageCost()`: Calculate cost for image generation
- `getModelPricing()`: Get pricing information for a specific model
- `getAllModelPricing()`: Get pricing for all supported models

#### 4. **ApiUsageService** (`service/ApiUsageService.java`)
Service for recording and retrieving usage statistics.

**Key Methods:**
- `recordUsage()`: Record API usage for a specific user
- `recordUsageForCurrentUser()`: Record usage for currently authenticated user
- `getUserUsageStats()`: Get statistics for a user
- `getCurrentUserUsageStats()`: Get statistics for current user
- `getWorkspaceUsageStats()`: Get statistics for a workspace
- `getUserUsageHistory()`: Get paginated usage history
- `getUserUsageStatsByDateRange()`: Get statistics for a date range

#### 5. **OpenAIProvider Integration** (`service/OpenAIProvider.java`)
The OpenAI provider automatically extracts token usage from API responses and records it.

**Integration Points:**
- `processOpenAIResponse()`: Extracts usage from Chat Completions API responses
- `processOpenAIResponseRaw()`: Extracts usage from raw Chat Completions responses
- `parseResponsesApiResponse()`: Extracts usage from Responses API responses
- `parseResponsesApiResponseRaw()`: Extracts usage from raw Responses API responses

**How It Works:**
1. After each OpenAI API call, the response JSON is parsed
2. Token usage information is extracted from the `usage` object
3. `ApiUsageService.recordUsageForCurrentUser()` is called automatically
4. Usage is recorded in the database with calculated costs

#### 6. **ApiUsageController** (`controller/ApiUsageController.java`)
REST API endpoints for retrieving usage statistics.

## API Endpoints

### 1. Get Current User Statistics
**Endpoint:** `GET /api/usage/stats`

**Description:** Returns total tokens, costs, and breakdown by model for the authenticated user.

**Authentication:** Required (JWT token)

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Usage statistics retrieved successfully",
  "data": {
    "totalTokens": 15000,
    "totalCost": 0.002250,
    "modelBreakdown": [
      {
        "model": "gpt-4o-mini",
        "totalTokens": 15000,
        "totalCost": 0.002250,
        "requestCount": 25
      }
    ]
  },
  "timestamp": "2026-01-25T15:00:00"
}
```

### 2. Get Usage History
**Endpoint:** `GET /api/usage/history`

**Description:** Returns paginated list of API usage records for the authenticated user.

**Authentication:** Required (JWT token)

**Query Parameters:**
- `page` (default: 0): Page number (0-indexed)
- `size` (default: 20): Number of records per page
- `sortBy` (default: "createdAt"): Field to sort by
- `sortDir` (default: "DESC"): Sort direction (ASC or DESC)

**Example:** `GET /api/usage/history?page=0&size=20&sortBy=createdAt&sortDir=DESC`

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Usage history retrieved successfully",
  "data": [
    {
      "id": "uuid-123",
      "userId": "user-456",
      "workspaceId": "workspace-789",
      "provider": "openai",
      "model": "gpt-4o-mini",
      "apiType": "text",
      "promptTokens": 100,
      "completionTokens": 200,
      "totalTokens": 300,
      "cost": 0.000045,
      "requestId": "req-abc123",
      "endpoint": "https://api.openai.com/v1/chat/completions",
      "createdAt": "2026-01-25T14:30:00"
    }
  ],
  "count": 150
}
```

### 3. Get Statistics by Date Range
**Endpoint:** `GET /api/usage/stats/range`

**Description:** Returns total tokens and costs for the authenticated user within a specified date range.

**Authentication:** Required (JWT token)

**Query Parameters:**
- `startDate`: Start date (ISO 8601 format)
- `endDate`: End date (ISO 8601 format)

**Example:** `GET /api/usage/stats/range?startDate=2026-01-01T00:00:00&endDate=2026-01-31T23:59:59`

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Usage statistics retrieved successfully",
  "data": {
    "totalTokens": 5000,
    "totalCost": 0.000750,
    "startDate": "2026-01-01T00:00:00",
    "endDate": "2026-01-31T23:59:59"
  }
}
```

### 4. Get User Statistics (Admin Only)
**Endpoint:** `GET /api/usage/stats/user/{userId}`

**Description:** Returns statistics for a specific user. Only accessible by SUPER_ADMIN or the user themselves.

**Authentication:** Required (JWT token)

**Authorization:** SUPER_ADMIN or own user ID

**Response:** Same structure as "Get Current User Statistics"

### 5. Get Workspace Statistics (Admin Only)
**Endpoint:** `GET /api/usage/stats/workspace/{workspaceId}`

**Description:** Returns statistics for a workspace. Only accessible by SUPER_ADMIN or workspace members.

**Authentication:** Required (JWT token)

**Authorization:** SUPER_ADMIN or workspace member

**Response:** Same structure as "Get Current User Statistics"

## Database Schema

### Table: `api_usage`

```sql
CREATE TABLE api_usage (
    id VARCHAR(36) NOT NULL PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    workspace_id VARCHAR(36),
    provider VARCHAR(50) NOT NULL,
    model VARCHAR(100) NOT NULL,
    api_type VARCHAR(50) NOT NULL,
    prompt_tokens INT,
    completion_tokens INT,
    total_tokens INT,
    cost DECIMAL(10, 6),
    request_id VARCHAR(100),
    endpoint VARCHAR(200),
    created_at DATETIME(6) NOT NULL,
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at),
    INDEX idx_provider_model (provider, model),
    INDEX idx_workspace_id (workspace_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Migration File:** `V42__Create_api_usage_table.sql`

## How It Works

### Automatic Tracking Flow

1. **User makes an API call** → OpenAI API is invoked (via `OpenAIProvider`)
2. **Response received** → OpenAI returns JSON with usage information
3. **Usage extraction** → Token counts extracted from response `usage` object:
   ```json
   {
     "usage": {
       "prompt_tokens": 100,
       "completion_tokens": 200,
       "total_tokens": 300
     }
   }
   ```
4. **Cost calculation** → `ApiPricingService` calculates cost based on model and tokens
5. **Database recording** → `ApiUsageService` saves the record with:
   - User ID (from authentication context)
   - Workspace ID (from authentication context)
   - Token counts
   - Calculated cost
   - Model and provider information
   - Timestamp

### Cost Calculation Example

For a text generation request using `gpt-4o-mini`:
- Prompt tokens: 1,000
- Completion tokens: 500
- Total tokens: 1,500

**Calculation:**
- Input cost: (1,000 / 1,000,000) × $0.150 = $0.00015
- Output cost: (500 / 1,000,000) × $0.600 = $0.0003
- **Total cost: $0.00045**

## Configuration

### Application Properties

No additional configuration is required. The system uses existing OpenAI configuration:

```properties
# OpenAI Configuration
ai.text.openai.api.url=https://api.openai.com/v1/chat/completions
ai.text.openai.api.key=your-api-key
ai.text.openai.model=gpt-4o-mini
```

### Pricing Updates

To update pricing for models, modify the `initializePricing()` method in `ApiPricingService.java`. Prices are stored per 1K tokens for easy calculation.

## Usage Examples

### Example 1: Get Your Usage Statistics

```bash
curl -X GET "http://localhost:8080/api/usage/stats" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Example 2: Get Usage History with Pagination

```bash
curl -X GET "http://localhost:8080/api/usage/history?page=0&size=10" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Example 3: Get Usage for January 2026

```bash
curl -X GET "http://localhost:8080/api/usage/stats/range?startDate=2026-01-01T00:00:00&endDate=2026-01-31T23:59:59" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Example 4: Admin - Get User Statistics

```bash
curl -X GET "http://localhost:8080/api/usage/stats/user/user-123" \
  -H "Authorization: Bearer ADMIN_JWT_TOKEN"
```

## Statistics Breakdown

### User Statistics Include:
- **Total Tokens**: Sum of all tokens used by the user
- **Total Cost**: Sum of all costs incurred
- **Model Breakdown**: Statistics grouped by model:
  - Model name
  - Total tokens for that model
  - Total cost for that model
  - Number of requests using that model

### Workspace Statistics Include:
- Same structure as user statistics, but aggregated across all users in the workspace

## Error Handling

The system is designed to be non-intrusive:
- If `ApiUsageService` is not available, tracking is skipped (logged as debug)
- If usage information is missing from API response, tracking is skipped
- Errors in usage tracking do not affect the main API call flow
- All errors are logged but do not throw exceptions

## Performance Considerations

- **Indexes**: Database indexes on `user_id`, `created_at`, and `provider+model` ensure fast queries
- **Async Recording**: Usage recording happens synchronously but is lightweight
- **Pagination**: History endpoints support pagination to handle large datasets
- **Aggregation**: Statistics use SQL aggregation functions for efficient calculation

## Future Enhancements

Potential improvements:
1. **Async Recording**: Move usage recording to async queue for better performance
2. **Caching**: Cache frequently accessed statistics
3. **Alerts**: Set up alerts for high usage or costs
4. **Export**: Export usage data to CSV/Excel
5. **Graphs/Charts**: Visual representation of usage trends
6. **Budget Limits**: Set usage/cost limits per user or workspace
7. **Rate Limiting**: Integrate with rate limiting based on usage

## Troubleshooting

### Issue: Usage not being recorded

**Check:**
1. Verify `ApiUsageService` is properly injected (check logs for "ApiUsageService not available")
2. Check if OpenAI API responses include `usage` object
3. Verify user authentication is working (userId must be available)
4. Check database connection and table exists

### Issue: Incorrect costs

**Check:**
1. Verify model name matches pricing configuration
2. Check if pricing has been updated in `ApiPricingService`
3. Verify token counts are being extracted correctly

### Issue: Statistics endpoint errors

**Check:**
1. Verify user is authenticated
2. Check database queries are working
3. Verify date formats for date range queries (ISO 8601)

## Support

For issues or questions:
1. Check application logs for detailed error messages
2. Verify database migration `V42__Create_api_usage_table.sql` has run
3. Ensure all dependencies are properly configured

---

**Last Updated:** January 2026  
**Version:** 1.0


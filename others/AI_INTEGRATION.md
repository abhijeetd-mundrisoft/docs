# AI Text Generation Integration

This document describes the AI text generation integration in the Course Forge backend.

## Overview

The AI integration provides text generation capabilities using Google's Gemini API. The implementation is designed to be extensible for future media generation features.

## API Endpoints

### POST /api/ai/text

Generate text content using AI based on a prompt.

**Request:**
```json
{
  "prompt": "Create a course outline for machine learning basics",
  "context": "This is for a beginner-level course"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "content": "<h1>Machine Learning Basics Course Outline</h1><p>...</p>",
    "contentType": "text/html",
    "requestId": "req-123456",
    "metadata": {
      "promptLength": 45,
      "hasContext": true,
      "promptTokenCount": 150,
      "totalTokenCount": 200
    },
    "generatedAt": "2024-01-15T10:30:00"
  },
  "message": "Text content generated successfully"
}
```

## Authentication

All AI endpoints require JWT authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

## Configuration

The following properties can be configured in `application.properties`:

```properties
# AI Text Generation Configuration
ai.text.gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
ai.text.gemini.api.key=${GEMINI_API_KEY:your-api-key-here}
ai.text.gemini.timeout=30000
ai.text.max-prompt-length=2000
ai.text.rate-limit.per-minute=10
ai.text.rate-limit.per-hour=100
```

## Future Extensions

The architecture is designed to support additional AI capabilities:

- `/api/ai/image` - Image generation
- `/api/ai/video` - Video generation  
- `/api/ai/audio` - Audio generation

## Error Handling

The API returns appropriate HTTP status codes:

- `200` - Success
- `400` - Bad request (invalid prompt)
- `401` - Unauthorized (invalid token)
- `429` - Rate limit exceeded
- `500` - Internal server error

## Rate Limiting

Rate limits are applied per user:
- 10 requests per minute
- 100 requests per hour

## Dependencies

The following dependencies were added:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-all</artifactId>
    <version>0.64.8</version>
</dependency>
```

## Usage Example

```bash
curl -X POST http://localhost:8080/api/ai/text \
  -H "Authorization: Bearer <your-jwt-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a course outline for machine learning basics",
    "context": "This is for a beginner-level course"
  }'
```

**Note:** The `context` field is optional and can be `null` or blank. If provided, it will be included as additional context for the AI prompt.

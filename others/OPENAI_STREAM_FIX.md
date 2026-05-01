# OpenAI Stream Parameter Fix

## Issue
The OpenAI API was returning an error because the `stream` parameter was being sent as a string `"false"` instead of a boolean `false`.

## Error Message
```
"Invalid type for 'stream': expected a boolean, but got a string instead."
```

## Root Cause
In the `OpenAIProvider.java`, the `createOpenAIRequest()` method was passing a string value for the stream parameter:

```java
// Before (INCORRECT)
return new OpenAIRequest(
    model,
    List.of(new OpenAIRequest.OpenAIMessage("user", prompt)),
    2000,  // max_tokens
    0.7,   // temperature
    "false" // stream - STRING (incorrect)
);
```

## Fix Applied

### 1. **OpenAIRequest.java**
Changed the `stream` field type from `String` to `Boolean`:

```java
// Before
private String stream;

// After  
private Boolean stream;
```

### 2. **OpenAIProvider.java**
Updated the `createOpenAIRequest()` method to pass a boolean value:

```java
// After (CORRECT)
return new OpenAIRequest(
    model,
    List.of(new OpenAIRequest.OpenAIMessage("user", prompt)),
    2000,  // max_tokens
    0.7,   // temperature
    false  // stream - BOOLEAN (correct)
);
```

## Result
- ✅ **OpenAI API calls now work correctly**
- ✅ **Stream parameter is properly typed as boolean**
- ✅ **No more "invalid type" errors**
- ✅ **Text generation should work as expected**

## Test the Fix
```bash
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a course outline for machine learning",
    "provider": "openai"
  }'
```

The OpenAI integration should now work properly without the stream parameter type error.

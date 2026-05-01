# Simplified API Request Structure

## Overview

Removed the `parameters` field from the TextPromptRequest DTO to simplify the API request structure and eliminate unnecessary complexity.

## Changes Made

### **TextPromptRequest.java**
- **Removed** `parameters` field and its Map import
- **Simplified** request structure to essential fields only
- **Maintained** all core functionality

### **Before**
```java
@Data
public class TextPromptRequest {
    @NotBlank(message = "Prompt cannot be empty")
    @Size(max = 2000, message = "Prompt cannot exceed 2000 characters")
    private String prompt;
    
    private String context;
    
    @Pattern(regexp = "openai|gemini", message = "Provider must be 'openai' or 'gemini'")
    private String provider; // Optional - defaults to OpenAI if null
    
    private Map<String, Object> parameters; // Provider-specific settings
}
```

### **After**
```java
@Data
public class TextPromptRequest {
    @NotBlank(message = "Prompt cannot be empty")
    @Size(max = 2000, message = "Prompt cannot exceed 2000 characters")
    private String prompt;
    
    private String context;
    
    @Pattern(regexp = "openai|gemini", message = "Provider must be 'openai' or 'gemini'")
    private String provider; // Optional - defaults to OpenAI if null
}
```

## Simplified API Request Structure

### **Current Request Format**
```json
{
  "prompt": "Create a course outline for machine learning",
  "context": "This is for a beginner-level course",
  "provider": "openai"
}
```

### **Minimal Request (uses defaults)**
```json
{
  "prompt": "Create a course outline for machine learning"
}
```

## Benefits

1. **Cleaner API**: Removed unnecessary complexity
2. **Simpler Requests**: No need for empty parameters object
3. **Better UX**: Fewer fields to manage
4. **Maintained Functionality**: All core features still work
5. **Backward Compatible**: Existing requests still work

## API Usage Examples

### **Basic Request (uses OpenAI default)**
```bash
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a course outline for machine learning"
  }'
```

### **Request with Context**
```bash
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a course outline for machine learning",
    "context": "This is for a beginner-level course"
  }'
```

### **Request with Provider Selection**
```bash
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a course outline for machine learning",
    "context": "This is for a beginner-level course",
    "provider": "gemini"
  }'
```

## Response Format (Unchanged)

```json
{
  "success": true,
  "data": {
    "content": "<h1>Machine Learning Course Outline</h1><p>...</p>",
    "contentType": "text/html",
    "requestId": "req-123456",
    "metadata": {
      "provider": "openai",
      "promptLength": 45,
      "hasContext": true,
      "capabilities": {
        "provider": "OpenAI",
        "model": "gpt-4o-mini",
        "supportsContext": true,
        "supportsMarkdown": true,
        "maxTokens": 4096
      },
      "providerAvailable": true
    },
    "generatedAt": "2024-01-15T10:30:00"
  },
  "message": "Text content generated successfully"
}
```

## Migration Notes

- **No Breaking Changes**: Existing API calls continue to work
- **Simplified Requests**: Remove any `parameters` field from existing requests
- **Cleaner Code**: Removed unused Map import and field
- **Same Functionality**: All provider selection and text generation features remain intact

The API is now cleaner and more focused on the essential functionality while maintaining full backward compatibility.

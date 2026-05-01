# AI Multi-Provider Implementation

## Overview

Successfully implemented a multi-provider AI architecture with OpenAI as the default provider and Gemini as fallback. The implementation uses the Strategy Pattern to support multiple AI providers seamlessly.

## Architecture Components

### 1. **AI Provider Interface**
- **File**: `AIProvider.java`
- **Purpose**: Defines contract for all AI providers
- **Methods**: `generateText()`, `getProviderName()`, `isAvailable()`, `getCapabilities()`

### 2. **Provider Implementations**

#### **OpenAI Provider** (Default)
- **File**: `OpenAIProvider.java`
- **Service Name**: `openaiProvider`
- **Model**: `gpt-4o-mini`
- **Features**: Markdown to HTML conversion, context support

#### **Gemini Provider** (Fallback)
- **File**: `GeminiAIProvider.java`
- **Service Name**: `geminiProvider`
- **Model**: `gemini-2.0-flash`
- **Features**: Markdown to HTML conversion, context support

### 3. **Provider Factory**
- **File**: `AIProviderFactory.java`
- **Purpose**: Manages provider selection and fallback logic
- **Features**: Default provider selection, fallback mechanisms, status checking

### 4. **Updated Service Layer**
- **File**: `TextPromptService.java`
- **Changes**: Refactored to use provider factory instead of direct Gemini calls
- **Features**: Provider-agnostic text generation

### 5. **Enhanced DTOs**

#### **TextPromptRequest**
- Added `provider` field (defaults to "openai")
- Added `parameters` field for provider-specific settings
- Validation for provider selection

#### **ProviderInfo**
- New DTO for provider information
- Includes capabilities and availability status

### 6. **API Endpoints**

#### **Existing Endpoint**
- `POST /api/ai/text` - Enhanced to support provider selection

#### **New Endpoints**
- `GET /api/ai/providers` - Get available providers with capabilities
- `GET /api/ai/providers/status` - Get provider availability status

## Configuration

### **application.properties**
```properties
# Default provider is OpenAI
ai.text.default.provider=openai
ai.text.fallback.provider=gemini

# OpenAI Configuration (Default)
ai.text.openai.api.url=https://api.openai.com/v1/chat/completions
ai.text.openai.api.key=your key
ai.text.openai.model=gpt-4o-mini
ai.text.openai.timeout=30000

# Gemini Configuration (Fallback)
ai.text.gemini.api.url=https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
ai.text.gemini.api.key=your key
ai.text.gemini.timeout=30000
```

## Usage Examples

### **Text Generation with Provider Selection**
```json
POST /api/ai/text
{
  "prompt": "Create a course outline for machine learning",
  "context": "This is for a beginner-level course",
  "provider": "openai"
}
```

### **Get Available Providers**
```json
GET /api/ai/providers
Response:
{
  "success": true,
  "data": [
    {
      "name": "openai",
      "displayName": "Openai",
      "available": true,
      "capabilities": {
        "provider": "OpenAI",
        "model": "gpt-4o-mini",
        "supportsContext": true,
        "supportsMarkdown": true,
        "maxTokens": 4096
      }
    }
  ]
}
```

## Key Features

### **1. Provider Selection**
- Users can specify provider in request
- Defaults to OpenAI if not specified
- Validation ensures only supported providers

### **2. Fallback Mechanism**
- Automatic fallback to Gemini if OpenAI fails
- Configurable fallback provider
- Graceful error handling

### **3. Provider Capabilities**
- Each provider exposes its capabilities
- Model information and limits
- Feature support indicators

### **4. Status Monitoring**
- Real-time provider availability checking
- Status endpoint for monitoring
- Detailed error logging

## Benefits

1. **Extensibility**: Easy to add new AI providers
2. **Reliability**: Fallback mechanisms ensure service availability
3. **Flexibility**: Users can choose preferred provider
4. **Monitoring**: Built-in status and capability reporting
5. **Backward Compatibility**: Existing API calls work unchanged

## Migration Notes

- **Default Provider**: Changed from Gemini to OpenAI
- **API Compatibility**: Existing requests work with default provider
- **New Features**: Provider selection and status endpoints available
- **Configuration**: OpenAI API key added to properties

## Testing

### **Test Provider Selection**
```bash
# Test OpenAI (default)
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -d '{"prompt": "Hello world", "provider": "openai"}'

# Test Gemini
curl -X POST /api/ai/text \
  -H "Authorization: Bearer <token>" \
  -d '{"prompt": "Hello world", "provider": "gemini"}'
```

### **Test Provider Status**
```bash
# Get available providers
curl -X GET /api/ai/providers

# Get provider status
curl -X GET /api/ai/providers/status
```

## Future Enhancements

1. **Cost Tracking**: Per-provider usage and cost monitoring
2. **Load Balancing**: Distribute requests across providers
3. **A/B Testing**: Compare provider performance
4. **Custom Models**: Support for fine-tuned models
5. **Rate Limiting**: Provider-specific rate limiting

This implementation provides a robust, extensible foundation for multi-provider AI text generation while maintaining backward compatibility and adding powerful new capabilities.

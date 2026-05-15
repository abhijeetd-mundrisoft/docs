# ⚙️ Technical Context: Multiple Model Settings from App Config

## 1. Code Change Specification (Logical Flow)

### A. Data Model (Spring Configuration Properties)
- **New Class**: Create a structured `@ConfigurationProperties` class to map the application properties safely into a POJO.
- **Implementation**:
  ```java
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.context.annotation.Configuration;
  import java.util.Map;
  
  @Configuration
  @ConfigurationProperties(prefix = "ai")
  public class AiModelProperties {
      private Map<String, ModelConfig> models;
      
      // Getters and Setters
      
      public static class ModelConfig {
          private String apiUrl;
          private String apiKey;
          private Integer timeout;
          private Integer maxTokens;
          // Getters and Setters
      }
  }
  ```

### B. Service Layer
- **`AiModelService` & `AiProviderConfigService`**: 
  - Inject the new `AiModelProperties` bean via constructor injection.
  - Modify resolving logic (e.g., `resolveUrl`, `getTimeout`) to first inspect `AiModelProperties.getModels().get(modelName)`. If a configuration is present for the specific model, return it. Otherwise, fall back to the existing database-driven or provider-level application properties.
  - Ensure proper `import` statements are used (e.g., `import com.mundrisoft.courseforge.config.AiModelProperties;`). Do not rely on direct package directory paths.

## 2. Consistency Rules
- **Use Existing Code First**: Continue utilizing the `AiProviderConfigService` as the central authority for resolving configurations. Enhance its existing methods (`resolveUrl`, etc.) rather than creating redundant resolution flows.
- **Maintain Fallback Logic**: The application should maintain a clear priority hierarchy:
  1. App Settings (Model Specific)
  2. Database Config (Model Specific)
  3. Database Config (Provider Level)
  4. App Settings (Provider Level Fallback)

## 3. Data Representation (JSON Structure)
While configured via `application.properties` or `application.yml`, the conceptual structure mapped to the backend properties represents the following JSON schema:

```json
{
  "ai": {
    "models": {
      "gemini-2.5-pro": {
        "apiUrl": "https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent",
        "apiKey": "custom-override-key",
        "timeout": 60000,
        "maxTokens": 8192
      },
      "gpt-4o-mini": {
        "timeout": 15000
      }
    }
  }
}
```

*In `application.properties` format:*
```properties
ai.models.gemini-2.5-pro.api-url=https://generativelanguage.googleapis.com/v1/models/gemini-2.5-pro:generateContent
ai.models.gemini-2.5-pro.timeout=60000
ai.models.gpt-4o-mini.timeout=15000
```

## 4. CRUD Operation Handling
- **Create / Update / Delete**: This functionality is dictated entirely by environment variables and property files. These settings are structurally "read-only" at application runtime. Any updates require restarting the Spring Boot application or invoking a Spring Cloud Config Actuator refresh (`/actuator/refresh`) if dynamically bound.
- **Read**: Settings are ingested at startup and held in singleton memory via Spring Context, offering near-instant read access.

## 5. Implementation Plan

1. **Development**: 
   - Define the `AiModelProperties` class.
   - Refactor `AiProviderConfigService` to consume these properties.
   - Verify proper imports and clean code integration.
2. **Unit Testing**: 
   - Write `@SpringBootTest` context tests that mock different `application.properties` variations. 
   - Ensure the hierarchy prioritizes model-specific app config over provider-level fallbacks.
3. **Integration Testing**: 
   - Perform end-to-end course generation requests with overridden model settings to verify properties are active.
4. **Deployment**: 
   - Deploy backend utilizing environment variables in the infrastructure configuration (e.g., Kubernetes ConfigMaps or Docker env vars).
5. **Rollback Strategy**: 
   - To rollback, simply remove the `ai.models.*` properties from the configuration. The application will seamlessly revert to existing database/provider-level logic without requiring code rollbacks.

## 6. API Impact Analysis
- **Impacted Endpoints**:
  - `POST /api/v1/courses/generate`
  - `POST /api/v1/blocks/generate`
- **Required Code Modifications**: No direct modifications to the Controller or HTTP layer logic are necessary. The impact is strictly confined to how the downstream Service Layer resolves configurations during execution of these endpoints.

## 7. Documentation Reference
Refer to the `AI_MULTI_PROVIDER_IMPLEMENTATION.md` document in the repository root. Ensure the Multi-Provider implementation patterns are maintained and that adding model-specific properties acts as an extension, not a replacement, to the robust Provider Interfaces.

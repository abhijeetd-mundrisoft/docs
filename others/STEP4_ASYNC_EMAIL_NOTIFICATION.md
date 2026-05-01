# Step 4: Asynchronous Course Generation with Email Notification

## Overview

Step 4 implements an asynchronous course generation workflow where:
1. **Client sends request** → Server immediately returns **202 Accepted**
2. **Background processing** generates the full course structure
3. **Course is persisted** to the database (appears in user's course list)
4. **Email notification** is sent to the user when generation completes
5. **User receives email** and can access the course from their course list

---

## Architecture Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  Client Request                                                 │
│  POST /course/full-course                                       │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│  Controller                                                      │
│  - Validate request                                              │
│  - Return 202 Accepted immediately                               │
│  - Start async processing                                        │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│  CompletableFuture Pipeline (Background)                        │
│                                                                  │
│  Stage 1: Generate Course Structure                             │
│  ├─ Build prompt with block schema                              │
│  ├─ Resolve OpenAI file IDs                                     │
│  └─ Call AI service (30-60s)                                    │
│                                                                  │
│  Stage 2: Generate Images (if enabled)                          │
│  ├─ Extract image placeholders                                  │
│  ├─ Generate images in parallel (5-10s each)                    │
│  └─ Update blocks with file IDs                                 │
│                                                                  │
│  Stage 3: Persist Course to Database                            │
│  ├─ Create Course entity                                        │
│  ├─ Create Modules                                              │
│  ├─ Create Lessons                                              │
│  └─ Create Blocks (including quiz questions)                    │
│                                                                  │
│  Stage 4: Send Email Notification                                │
│  ├─ Prepare email content                                       │
│  └─ Send email via email service                                │
│                                                                  │
│  Stage 5: Cleanup                                                │
│  └─ Remove temporary AI artifacts                               │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│  Result                                                          │
│  - Course saved in database                                      │
│  - Course appears in user's course list                          │
│  - User receives email notification                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## API Design

### Request Endpoint

**POST** `/course/full-course`

**Request Body:**
```json
{
  "metadata": {
    "title": "Leadership Essentials for New Managers",
    "description": "A foundational course...",
    "audience": "New managers...",
    "goal": "Develop leadership capability.",
    "fileIds": ["file-abc123", "file-xyz789"]
  },
  "outline": {
    "lessons": [
      {
        "title": "Lesson 1: Understanding Leadership Roles",
        "description": "Introduction to leadership responsibilities."
      }
    ]
  },
  "questionsPerLesson": 3,
  "generateImages": true
}
```

**Response (202 Accepted):**
```json
{
  "success": true,
  "message": "Course generation started. You will receive an email notification when it's ready.",
  "data": {
    "status": "IN_PROGRESS",
    "message": "Course generation started. You will receive an email notification when it's ready."
  }
}
```

**Note:** 
- No `courseId` in response (will be available after generation)
- User will receive email with course details and link when ready

**Response Headers:**
- `HTTP/1.1 202 Accepted`
- `Content-Type: application/json`

---

## Implementation Details

### 1. Controller Layer

**Responsibilities:**
- Validate request (metadata, outline required)
- Authenticate user
- Return 202 Accepted immediately
- Start async processing (fire and forget)

**Code Structure:**
```java
@PostMapping("/course/full-course")
public ResponseEntity<ApiResponseDto<CourseGenerationResponseDto>> generateFullCourse(
        @RequestBody CourseFullCourseGenerationRequest request,
        HttpServletRequest httpRequest) {
    
    // Validate & authenticate
    String token = extractToken(httpRequest);
    String workspaceId = jwtUtil.getWorkspaceIdFromToken(token);
    String userId = jwtUtil.getUserIdFromToken(token);
    
    // Validate request
    if (request.getMetadata() == null || request.getOutline() == null) {
        return ResponseEntity.badRequest()
            .body(ApiResponseDto.error("Metadata and outline are required", 400));
    }
    
    // Start async processing (fire and forget)
    courseFullCourseService.generateFullCourseAsync(request, workspaceId, userId)
        .exceptionally(throwable -> {
            // Log error, send failure email
            log.error("Course generation failed for user: {}", userId, throwable);
            emailService.sendCourseGenerationFailureEmail(userId, throwable);
            return null;
        });
    
    // Return immediately
    return ResponseEntity.status(HttpStatus.ACCEPTED)
        .body(ApiResponseDto.success(
            CourseGenerationResponseDto.builder()
                .status("IN_PROGRESS")
                .message("Course generation started. You will receive an email when ready.")
                .build(),
            "Course generation started"
        ));
}
```

### 2. Service Layer - CompletableFuture Pipeline

**Pipeline Stages:**

```java
CompletableFuture<Course> pipeline = 

    // Stage 1: Generate Course Structure
    CompletableFuture.supplyAsync(() -> 
        generateCourseStructure(request, workspaceId), executor)
    
    // Stage 2: Generate Images (if enabled)
    .thenCompose(course -> 
        generateImagesIfEnabled(course, request, userId, workspaceId))
    
    // Stage 3: Persist Course to Database
    .thenCompose(course -> 
        persistCourseToDatabase(course, userId, workspaceId))
    
    // Stage 4: Send Email Notification (async, non-blocking)
    .thenAccept(savedCourse -> {
        // Send email asynchronously (don't block pipeline)
        CompletableFuture.runAsync(() -> 
            emailService.sendCourseGenerationSuccessEmail(userId, savedCourse), executor)
            .exceptionally(emailError -> {
                log.error("Failed to send success email", emailError);
                return null; // Don't fail pipeline if email fails
            });
    })
    
    // Stage 5: Cleanup
    .thenRun(() -> 
        cleanupSourceMaterials(request.getFileIds()))
    
    // Error Handling
    .exceptionally(throwable -> {
        log.error("Pipeline failed", throwable);
        // Send failure email
        CompletableFuture.runAsync(() -> 
            emailService.sendCourseGenerationFailureEmail(userId, throwable), executor);
        return null;
    });
```

### 3. Course Persistence Service

**Responsibilities:**
- Create Course entity with metadata (using `CourseService.createCourse()`)
- Create Modules (using `ModuleService.createModule()`)
- Create Lessons (using `LessonService.createLesson()`)
- Create Blocks (using `BlockService.createBlock()`)
- Serialize block content as JSON string
- **Note:** Quiz questions are stored in block content JSON (not separate table)

**Database Structure:**
```
Course
  └─ Modules[]
      └─ Lessons[]
          └─ Blocks[]
              └─ QuizQuestions[] (if block type is QUIZ)
```

**Implementation Notes:**
- Use `@Transactional` for data consistency
- Handle large course structures efficiently (batch inserts if needed)
- Serialize block content as JSON string using ObjectMapper
- Maintain proper order indices (1, 2, 3, ...)
- Set `created_by` = `userId` and `updated_by` = `userId`
- Set course `status` = `DRAFT` (not ACTIVE) for AI-generated courses
- Set module `status` = `ACTIVE` for AI-generated modules (default is DRAFT)
- Set lesson `status` = `ACTIVE` for AI-generated lessons (default is DRAFT)
- Set block `status` = `ACTIVE` for AI-generated blocks (default is DRAFT)
- Use `CourseService.createCourse()` which handles:
  - UUID generation for course ID
  - Workspace limitations increment
  - Timestamp setting
- Use existing `ModuleService.createModule()`, `LessonService.createLesson()`, `BlockService.createBlock()` for creation
- For blocks: Serialize content Object to JSON string using ObjectMapper before saving
- For QUIZ blocks: Content is already an array, serialize as-is
- Block content validation is handled automatically by `BlockContentValidator` in `BlockService`
- Use batch operations (`saveAll()`) for better performance when creating multiple entities

### 4. Email Notification Service

**Email Content:**
- **Subject**: "Your Course '[Course Title]' is Ready!"
- **Body**: 
  - Greeting
  - Course title and description
  - Link to view course
  - Course overview (modules, lessons count)
  - Thank you message

**Email Template (HTML):**
```html
Subject: Your Course '[Course Title]' is Ready!

<html>
<body>
  <h2>Hello [User First Name Last Name]!</h2>
  <p>Great news! Your course <strong>"[Course Title]"</strong> has been successfully generated 
     and is now available in your course list.</p>
  
  <h3>Course Overview:</h3>
  <ul>
    <li><strong>[X]</strong> Module(s)</li>
    <li><strong>[Y]</strong> Lesson(s)</li>
  </ul>
  
  <p>You can now:</p>
  <ul>
    <li>View and edit your course</li>
    <li>Add additional content</li>
    <li>Publish when ready</li>
  </ul>
  
  <p>
    <a href="[Course URL]" style="background-color: #4CAF50; color: white; 
       padding: 10px 20px; text-decoration: none; border-radius: 5px;">
      View Course
    </a>
  </p>
  
  <p>Thank you for using Course Forge!</p>
  <p>Best regards,<br>Course Forge Team</p>
</body>
</html>
```

**Failure Email:**
```html
Subject: Course Generation Failed

<html>
<body>
  <h2>Hello [User First Name Last Name]!</h2>
  <p>We encountered an issue while generating your course.</p>
  <p><strong>Error:</strong> [Error Message]</p>
  <p>Please try again or contact support if the issue persists.</p>
  <p>Best regards,<br>Course Forge Team</p>
</body>
</html>
```

### 5. Error Handling

**Error Scenarios:**

1. **Course Generation Fails**
   - Log error
   - Send failure email to user
   - Don't persist partial data
   - Keep source materials for retry

2. **Image Generation Fails (Partial)**
   - Continue with successful images
   - Log failures
   - Persist course with available images
   - Send success email (course is usable)

3. **Persistence Fails**
   - Log error
   - Send failure email
   - Don't send success email
   - Retry mechanism (optional)

4. **Email Sending Fails**
   - Log error
   - Course is still saved
   - User can access course from list
   - Consider retry mechanism

---

## Database Schema

### Course Entity
```sql
CREATE TABLE courses (
    id VARCHAR(36) PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    workspace_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    created_by VARCHAR(36),
    status ENUM('DRAFT', 'ACTIVE', 'ARCHIVED') DEFAULT 'DRAFT',
    profile_image_file_id VARCHAR(36),
    language VARCHAR(10),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    ...
);
```

**Note:** 
- Course status should be set to `DRAFT` for AI-generated courses (not `ACTIVE`)
- `user_id` is the owner of the course (set to `userId` from request)
- `created_by` is the user who created it (set to `userId` for AI generation)
- Use `CourseService.createCourse()` which:
  - Generates UUID for course ID
  - Sets timestamps automatically
  - Increments workspace course count
  - Sets default status to `ACTIVE` (override to `DRAFT` for AI-generated)

### Module Entity
```sql
CREATE TABLE modules (
    id VARCHAR(36) PRIMARY KEY,
    course_id VARCHAR(36) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    order_index INT,
    status ENUM('DRAFT', 'ACTIVE', 'ARCHIVED') DEFAULT 'DRAFT',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    updated_by VARCHAR(36),
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);
```

**Note:**
- Status should be explicitly set to `ACTIVE` for AI-generated modules (default is `DRAFT`)
- `order_index` should be set based on module position (1, 2, 3, ...)
- Use `ModuleService.createModule()` which:
  - Generates UUID for module ID
  - Sets timestamps automatically
  - Handles order index if not provided

### Lesson Entity
```sql
CREATE TABLE lessons (
    id VARCHAR(36) PRIMARY KEY,
    module_id VARCHAR(36) NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    order_index INT,
    status ENUM('DRAFT', 'ACTIVE', 'ARCHIVED') DEFAULT 'DRAFT',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    updated_by VARCHAR(36),
    FOREIGN KEY (module_id) REFERENCES modules(id) ON DELETE CASCADE
);
```

**Note:**
- Status should be explicitly set to `ACTIVE` for AI-generated lessons (default is `DRAFT`)
- `order_index` should be set based on lesson position within module
- Use `LessonService.createLesson()` which:
  - Generates UUID for lesson ID
  - Sets timestamps automatically
  - Handles order index if not provided

### Block Entity
```sql
CREATE TABLE blocks (
    id VARCHAR(36) PRIMARY KEY,
    lesson_id VARCHAR(36) NOT NULL,
    type ENUM('TITLE', 'TEXT', 'IMAGE', 'VIDEO', 'QUIZ', 'SUMMARY', 
              'HEADING_TEXT', 'HEADING_ONLY', 'QUOTE', 'QUOTE_BOXED',
              'TWO_COLUMN_TEXT', 'THREE_COLUMN_TEXT',
              'TWO_COLUMN_IMAGE', 'THREE_COLUMN_IMAGE',
              'IMAGE_TEXT_BOTTOM', 'TEXT_IMAGE_BOTTOM',
              'IMAGE_TEXT_RIGHT', 'TEXT_IMAGE_LEFT',
              'TWO_COLUMN_VIDEO', 'THREE_COLUMN_VIDEO',
              'VIDEO_TEXT_BOTTOM', 'TEXT_VIDEO_BOTTOM',
              'VIDEO_TEXT_RIGHT', 'TEXT_VIDEO_LEFT') NOT NULL,
    content JSON NOT NULL,
    order_index INT NOT NULL,
    file_id VARCHAR(36),
    status ENUM('DRAFT', 'ACTIVE', 'ARCHIVED') DEFAULT 'DRAFT',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by VARCHAR(36),
    updated_by VARCHAR(36),
    FOREIGN KEY (lesson_id) REFERENCES lessons(id) ON DELETE CASCADE
);
```

**Note:**
- `content` is stored as JSON string (serialized from Object using ObjectMapper)
- Status should be explicitly set to `ACTIVE` for AI-generated blocks (default is `DRAFT`)
- `order_index` should be set based on block position within lesson
- For QUIZ blocks, content is an array of question objects (stored in JSON)
- Block ID is auto-generated by `@GeneratedValue(strategy = GenerationType.UUID)`
- Use `BlockService.createBlock()` which:
  - Validates content using `BlockContentValidator`
  - Handles order index if not provided
  - Sets timestamps

### Quiz Questions Storage

**Important:** Quiz questions are stored **directly in the block content JSON**, not in a separate table.

**Block Content Structure for QUIZ blocks:**
```json
[
  {
    "quizType": "single-choice",
    "question": "What is the primary role of a leader?",
    "options": ["To guide the team", "To ignore the team"],
    "correctAnswer": "To guide the team"
  },
  {
    "quizType": "true-false",
    "question": "Leadership involves inspiring people.",
    "correctAnswer": "true"
  }
]
```

**Note:**
- The `quiz_questions` table exists in the database schema but is **not used** in the current implementation
- All quiz data is stored in the block's `content` JSON field
- Each question object in the array follows the structure defined in BlockSchemaUtil

---

## Data Flow

### 1. Request Processing
```
Request → Validation → Authentication → 202 Accepted
```

### 2. Background Processing
```
Course Structure Generation
    ↓
Image Generation (parallel)
    ↓
Course Persistence
    ├─ Course
    ├─ Modules
    ├─ Lessons
    ├─ Blocks
    └─ Quiz Questions
    ↓
Email Notification
    ↓
Cleanup
```

### 3. User Experience
```
User submits request
    ↓
Immediate response: "Course generation started"
    ↓
User continues working
    ↓
Email received: "Your course is ready!"
    ↓
User opens course list
    ↓
Course appears in list
    ↓
User clicks to view/edit
```

---

## Configuration

### Application Properties

```properties
# Course Generation Configuration
course.generation.thread-pool.core-size=5
course.generation.thread-pool.max-size=10
course.generation.thread-pool.queue-capacity=100
course.generation.image.thread-pool.core-size=3
course.generation.image.thread-pool.max-size=5
course.generation.image.thread-pool.queue-capacity=50
course.generation.max-images-per-course=20
course.generation.auto-generate-images=true

# Email Configuration
course.generation.email.enabled=true
course.generation.email.template.success=course-generation-success
course.generation.email.template.failure=course-generation-failure
```

---

## Email Service Integration

### Email Service Integration

**Existing Email Service:**
- `EmailSenderSMTP` has `sendCustomEmail(String toEmail, String subject, String content)` method
- Can be used directly for course generation emails

**Email Service Implementation:**

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class CourseGenerationEmailService {
    
    private final EmailSenderSMTP emailSender;
    private final UserRepository userRepository;
    private final ModuleRepository moduleRepository;
    private final LessonRepository lessonRepository;
    private final ObjectMapper objectMapper;
    
    /**
     * Send success email when course generation completes.
     */
    public void sendCourseGenerationSuccessEmail(String userId, Course course) {
        try {
            User user = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("User not found: " + userId));
            
            // Count modules and lessons
            List<Module> modules = moduleRepository.findByCourseIdOrderByOrderIndex(course.getId());
            int totalLessons = 0;
            for (Module module : modules) {
                totalLessons += lessonRepository.findByModuleIdOrderByOrderIndex(module.getId()).size();
            }
            
            String subject = "Your Course '" + course.getTitle() + "' is Ready!";
            String body = buildSuccessEmailBody(course, user, modules.size(), totalLessons);
            
            emailSender.sendCustomEmail(user.getEmail(), subject, body);
            
            log.info("Success email sent to user: {} for course: {}", userId, course.getId());
            
        } catch (Exception e) {
            log.error("Failed to send success email to user: {} for course: {}", 
                userId, course.getId(), e);
            // Don't fail the pipeline if email fails
        }
    }
    
    /**
     * Send failure email when course generation fails.
     */
    public void sendCourseGenerationFailureEmail(String userId, Throwable error) {
        try {
            User user = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("User not found: " + userId));
            
            String subject = "Course Generation Failed";
            String body = buildFailureEmailBody(user, error);
            
            emailSender.sendCustomEmail(user.getEmail(), subject, body);
            
            log.info("Failure email sent to user: {}", userId);
            
        } catch (Exception e) {
            log.error("Failed to send failure email to user: {}", userId, e);
        }
    }
    
    private String buildSuccessEmailBody(Course course, User user, int moduleCount, int lessonCount) {
        // Get user name
        String userName = getUserDisplayName(user);
        
        StringBuilder html = new StringBuilder();
        html.append("<html><body>");
        html.append("<h2>Hello ").append(userName).append("!</h2>");
        html.append("<p>Great news! Your course <strong>").append(course.getTitle())
            .append("</strong> has been successfully generated and is now available in your course list.</p>");
        html.append("<h3>Course Overview:</h3>");
        html.append("<ul>");
        html.append("<li><strong>").append(moduleCount).append("</strong> Module(s)</li>");
        html.append("<li><strong>").append(lessonCount).append("</strong> Lesson(s)</li>");
        html.append("</ul>");
        html.append("<p>You can now:</p>");
        html.append("<ul>");
        html.append("<li>View and edit your course</li>");
        html.append("<li>Add additional content</li>");
        html.append("<li>Publish when ready</li>");
        html.append("</ul>");
        html.append("<p><a href=\"").append(buildCourseUrl(course.getId()))
            .append("\" style=\"background-color: #4CAF50; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;\">View Course</a></p>");
        html.append("<p>Thank you for using Course Forge!</p>");
        html.append("<p>Best regards,<br>Course Forge Team</p>");
        html.append("</body></html>");
        return html.toString();
    }
    
    private String buildFailureEmailBody(User user, Throwable error) {
        // Get user name
        String userName = getUserDisplayName(user);
        
        StringBuilder html = new StringBuilder();
        html.append("<html><body>");
        html.append("<h2>Hello ").append(userName).append("!</h2>");
        html.append("<p>We encountered an issue while generating your course.</p>");
        html.append("<p><strong>Error:</strong> ").append(error.getMessage()).append("</p>");
        html.append("<p>Please try again or contact support if the issue persists.</p>");
        html.append("<p>Best regards,<br>Course Forge Team</p>");
        html.append("</body></html>");
        return html.toString();
    }
    
    private String getUserDisplayName(User user) {
        String firstName = user.getFirstName();
        String lastName = user.getLastName();
        
        if (firstName == null && lastName == null) {
            return "User";
        } else if (firstName == null) {
            return lastName;
        } else if (lastName == null) {
            return firstName;
        } else {
            return firstName + " " + lastName;
        }
    }
    
    private String buildCourseUrl(String courseId) {
        // Build frontend URL to course detail page
        // Example: https://yourdomain.com/courses/{courseId}
        // This should be configurable via application.properties
        return "https://yourdomain.com/courses/" + courseId;
    }
}
```

---

## Error Recovery

### Retry Strategy

**For Transient Failures:**
- Retry course generation (max 3 attempts)
- Exponential backoff
- Log retry attempts

**For Permanent Failures:**
- Send failure email immediately
- Don't retry
- Keep source materials for manual retry

### Partial Success Handling

**If Images Fail:**
- Course is still saved
- Blocks with failed images keep placeholders
- Success email sent (course is usable)
- User can manually add images later

**If Some Blocks Fail:**
- Save successful blocks
- Log failed blocks
- Success email sent
- User can manually add missing blocks

---

## Performance Considerations

### Timing Estimates

**Course Generation:**
- Prompt building: < 1s
- AI API call: 30-60s
- Response parsing: < 1s
- **Total: ~35-65s**

**Image Generation (10 images):**
- Parallel execution: ~10-15s
- **Total: ~10-15s** (vs 100s sequential)

**Course Persistence:**
- Course creation: < 1s
- Modules (5 modules): < 1s
- Lessons (20 lessons): 1-2s
- Blocks (100 blocks): 2-5s
- **Total: ~4-8s**

**Note:** Quiz questions are stored in block content JSON, so no separate persistence step needed.

**Email Sending:**
- Template rendering: < 1s
- Email delivery: 1-3s
- **Total: ~2-4s**

**Overall Pipeline:**
- **Total: ~51-92s (0.85-1.5 minutes)**

**Breakdown:**
- Course Generation: 35-65s
- Image Generation (10 images, parallel): 10-15s
- Course Persistence: 4-8s
- Email Sending: 2-4s

### Optimization Strategies

1. **Parallel Image Generation**: Already implemented
2. **Batch Database Inserts**: 
   - Use `saveAll()` for modules, lessons, blocks
   - Consider batch size of 50-100 for large courses
3. **Async Email Sending**: 
   - Email sending should not block the pipeline
   - Use `thenRunAsync()` for email sending
4. **Connection Pooling**: Already handled by Spring Data JPA
5. **Caching**: Cache user data for email (optional)
6. **Transaction Management**: 
   - Use `@Transactional` for course persistence
   - Consider breaking into smaller transactions if needed
7. **Content Serialization**: 
   - Serialize block content once per block
   - Use ObjectMapper efficiently

---

## Monitoring & Logging

### Key Metrics

- **Generation Success Rate**: % of successful generations
- **Average Generation Time**: P50, P95, P99
- **Email Delivery Rate**: % of emails successfully sent
- **Error Rate by Stage**: Track failures per stage
- **Image Generation Success Rate**: % of images successfully generated

### Logging Strategy

```java
// Start
log.info("Course generation started - User: {}, Workspace: {}", userId, workspaceId);

// Each Stage
log.info("Stage 1 completed - Course structure generated - Duration: {}ms", duration);
log.info("Stage 2 completed - Images generated: {} success, {} failed", success, failed);
log.info("Stage 3 completed - Course persisted - Course ID: {}", courseId);
log.info("Stage 4 completed - Email sent - User: {}", userId);

// Completion
log.info("Course generation completed - Course ID: {}, Total Duration: {}ms", courseId, totalDuration);

// Errors
log.error("Course generation failed - User: {}, Error: {}", userId, error.getMessage(), error);
```

---

## Security Considerations

### Access Control

- **Workspace Isolation**: Users can only access courses in their workspace
- **User Verification**: Verify user owns the workspace
- **Rate Limiting**: Limit concurrent generations per user/workspace

### Data Privacy

- **Email Content**: Only include course title, not sensitive content
- **Error Messages**: Don't expose internal errors in emails
- **User Data**: Use minimal user data in emails

---

## Testing Strategy

### Unit Tests

- Course structure generation
- Image generation processor
- Course persistence service
- Email service

### Integration Tests

- Full pipeline end-to-end
- Error scenarios
- Email delivery
- Database persistence

### Performance Tests

- Large course structures (100+ blocks)
- Many images (20+ images)
- Concurrent requests
- Database load

---

## User Experience

### Immediate Response

**User sees:**
```
"Course generation started. You will receive an email notification when it's ready."
```

**User can:**
- Continue working
- Check email later
- Access course from list when ready

### Email Notification

**Email includes:**
- Course title
- Course overview
- Direct link to course
- Next steps

**User clicks link:**
- Redirected to course detail page
- Can view/edit course
- Course is fully functional

---

## Implementation Checklist

### Phase 1: Core Pipeline
- [ ] Create CourseFullCourseService with CompletableFuture
- [ ] Implement course structure generation
- [ ] Implement image generation (parallel)
- [ ] Add error handling

### Phase 2: Persistence
- [ ] Create CoursePersistenceService
- [ ] Implement course entity creation:
  - Use `CourseService.createCourse()`
  - Set status to `DRAFT` (override default ACTIVE)
  - Set `userId` and `createdBy` = `userId`
  - Set `workspaceId`
- [ ] Implement module creation:
  - Use `ModuleService.createModule()` in batch with `saveAll()`
  - Set status to `ACTIVE`
  - Set `orderIndex` based on position
  - Set `createdBy` = `userId`
- [ ] Implement lesson creation:
  - Use `LessonService.createLesson()` in batch with `saveAll()`
  - Set status to `ACTIVE`
  - Set `orderIndex` based on position
  - Set `createdBy` = `userId`
- [ ] Implement block creation:
  - Serialize block content Object to JSON string using ObjectMapper
  - Use `BlockService.createBlock()` in batch with `saveAll()`
  - Set status to `ACTIVE`
  - Set `orderIndex` based on position
  - Set `createdBy` = `userId`
  - Set `fileId` if image/video block has generated file
- [ ] Add `@Transactional` for data consistency
- [ ] Handle large courses efficiently (batch processing)

### Phase 3: Email Integration
- [ ] Create CourseGenerationEmailService
- [ ] Inject `EmailSenderSMTP` (existing service)
- [ ] Inject `UserRepository` to get user email and name
- [ ] Implement `getUserDisplayName()` helper (firstName + lastName)
- [ ] Implement success email:
  - Get user by userId
  - Count modules and lessons
  - Build HTML email body
  - Call `emailSender.sendCustomEmail()`
- [ ] Implement failure email:
  - Get user by userId
  - Build HTML email body with error message
  - Call `emailSender.sendCustomEmail()`
- [ ] Add error handling (don't fail pipeline if email fails)
- [ ] Test email delivery
- [ ] Configure course URL in application.properties

### Phase 4: Controller & API
- [ ] Create controller endpoint
- [ ] Implement 202 Accepted response
- [ ] Add request validation
- [ ] Add authentication

### Phase 5: Testing & Optimization
- [ ] Unit tests
- [ ] Integration tests
- [ ] Performance testing
- [ ] Error scenario testing
- [ ] Email delivery testing

---

## Summary

This architecture provides:

1. **Non-Blocking**: Immediate response (202 Accepted)
2. **User-Friendly**: Email notification when ready
3. **Persistent**: Course saved to database automatically
4. **Efficient**: Parallel processing, optimized for large data
5. **Resilient**: Error handling, partial success support
6. **Scalable**: Thread pools, async processing

The user experience is seamless:
- Submit request → Get immediate confirmation
- Continue working → No waiting
- Receive email → Course is ready
- Access course → Appears in course list

---

## Key Implementation Notes (Based on Codebase Review)

### Entity Status Values
- **Course**: Set to `DRAFT` (override default `ACTIVE` from `CourseService.createCourse()`)
- **Module**: Set to `ACTIVE` (default is `DRAFT`)
- **Lesson**: Set to `ACTIVE` (default is `DRAFT`)
- **Block**: Set to `ACTIVE` (default is `DRAFT`)

### Service Usage
- Use `CourseService.createCourse()` - handles UUID, timestamps, workspace limits
- Use `ModuleService.createModule()` - handles UUID, timestamps, order index
- Use `LessonService.createLesson()` - handles UUID, timestamps, order index
- Use `BlockService.createBlock()` - handles UUID, timestamps, order index, content validation

### Content Serialization
- Block content must be serialized to JSON string using `ObjectMapper.writeValueAsString()`
- Quiz questions are stored in block content JSON (array of question objects)
- No separate `quiz_questions` table is used in current implementation

### Email Service
- Use existing `EmailSenderSMTP.sendCustomEmail()` method
- User has `firstName` and `lastName` fields (not `name`)
- Build user display name: `firstName + " " + lastName` (handle nulls)
- Email should be sent asynchronously to not block pipeline

### Repository Methods
- `ModuleRepository.findByCourseIdOrderByOrderIndex(courseId)` - get modules
- `LessonRepository.findByModuleIdOrderByOrderIndex(moduleId)` - get lessons
- `UserRepository.findById(userId)` - get user for email

### Batch Operations
- Use `saveAll()` for creating multiple modules, lessons, blocks
- Improves performance for large courses

### Transaction Management
- Use `@Transactional` on persistence service methods
- Ensures data consistency across course/module/lesson/block creation


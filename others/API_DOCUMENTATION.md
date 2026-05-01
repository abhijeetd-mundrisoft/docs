# Course Forge Backend API Documentation

## Base URL
```
http://localhost:8080/api
```

## Authentication
**No authentication required** - All endpoints are open for simple course creation and management.

## Course Management API

### 1. Create a New Course
**POST** `/courses`

**Request Body:**
```json
{
  "title": "Introduction to Java Programming",
  "description": "Learn the basics of Java programming language",
  "language": "English",
  "profileImage": "https://example.com/java-course.jpg",
  "createdBy": "John Doe",
  "updatedBy": "John Doe",
  "tenantId": "tenant-123"
}
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Introduction to Java Programming",
  "description": "Learn the basics of Java programming language",
  "status": "ACTIVE",
  "language": "English",
  "profileImage": "https://example.com/java-course.jpg",
  "createdBy": "John Doe",
  "updatedBy": "John Doe",
  "tenantId": "tenant-123",
  "createdAt": "2024-01-15T10:30:00",
  "updatedAt": "2024-01-15T10:30:00",
  "modules": []
}
```

### 2. Get All Courses
**GET** `/courses`

**Response:**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "title": "Introduction to Java Programming",
    "description": "Learn the basics of Java programming language",
    "status": "ACTIVE",
    "language": "English",
    "profileImage": "https://example.com/java-course.jpg",
    "createdBy": "John Doe",
    "updatedBy": "John Doe",
    "tenantId": "tenant-123",
    "createdAt": "2024-01-15T10:30:00",
    "updatedAt": "2024-01-15T10:30:00",
    "modules": []
  }
]
```

### 3. Get Course by ID
**GET** `/courses/{id}`

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Introduction to Java Programming",
  "description": "Learn the basics of Java programming language",
  "status": "ACTIVE",
  "language": "English",
  "profileImage": "https://example.com/java-course.jpg",
  "createdBy": "John Doe",
  "updatedBy": "John Doe",
  "tenantId": "tenant-123",
  "createdAt": "2024-01-15T10:30:00",
  "updatedAt": "2024-01-15T10:30:00",
  "modules": []
}
```

### 4. Update Course
**PUT** `/courses/{id}`

**Request Body:**
```json
{
  "title": "Advanced Java Programming",
  "description": "Advanced concepts in Java programming",
  "language": "English",
  "profileImage": "https://example.com/advanced-java.jpg",
  "updatedBy": "Jane Doe"
}
```

### 5. Delete Course
**DELETE** `/courses/{id}`

**Response:** `204 No Content`

### 6. Clone Course
**POST** `/courses/{id}/clone`

**Response:**
```json
{
  "id": "660e8400-e29b-41d4-a716-446655440001",
  "title": "COPY - Introduction to Java Programming",
  "description": "Learn the basics of Java programming language",
  "status": "ACTIVE",
  "language": "English",
  "profileImage": "https://example.com/java-course.jpg",
  "createdBy": "John Doe",
  "updatedBy": "John Doe",
  "tenantId": "tenant-123",
  "createdAt": "2024-01-15T11:00:00",
  "updatedAt": "2024-01-15T11:00:00",
  "modules": []
}
```

### 7. Get Courses by Status
**GET** `/courses/status/{status}`

Where `{status}` can be `ACTIVE` or `ARCHIVED`

### 8. Search Courses by Title
**GET** `/courses/search?title={searchTerm}`

### 9. Get Courses by Tenant
**GET** `/courses/tenant/{tenantId}`

### 10. Get Courses by Tenant and Status
**GET** `/courses/tenant/{tenantId}/status/{status}`

## Frontend Integration

### Course Creation Flow
1. **Frontend Form Submission**: The frontend sends a POST request to `/courses` with course details
2. **Backend Processing**: The backend creates the course with auto-generated ID and timestamps
3. **Response**: The backend returns the created course with all fields populated

### Course Listing Flow
1. **Frontend Request**: The frontend sends a GET request to `/courses`
2. **Backend Processing**: The backend retrieves all courses from the database
3. **Response**: The backend returns an array of courses with all necessary fields

### Course Update Flow
1. **Frontend Request**: The frontend sends a PUT request to `/courses/{id}` with updated course data
2. **Backend Processing**: The backend updates the course and sets the `updatedAt` timestamp
3. **Response**: The backend returns the updated course

### Course Deletion Flow
1. **Frontend Request**: The frontend sends a DELETE request to `/courses/{id}`
2. **Backend Processing**: The backend deletes the course and all associated modules, lessons, and blocks
3. **Response**: The backend returns a 204 No Content status

## Error Handling

### 400 Bad Request
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "path": "/api/courses"
}
```

### 404 Not Found
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Course not found",
  "path": "/api/courses/550e8400-e29b-41d4-a716-446655440000"
}
```

### 500 Internal Server Error
```json
{
  "timestamp": "2024-01-15T10:30:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "An unexpected error occurred",
  "path": "/api/courses"
}
```

## Testing the API

### Using curl
```bash
# Create a course
curl -X POST http://localhost:8080/api/courses \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test Course",
    "description": "A test course",
    "language": "English",
    "createdBy": "Test User",
    "updatedBy": "Test User",
    "tenantId": "test-tenant"
  }'

# Get all courses
curl -X GET http://localhost:8080/api/courses

# Get course by ID
curl -X GET http://localhost:8080/api/courses/{id}

# Update course
curl -X PUT http://localhost:8080/api/courses/{id} \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Course",
    "description": "Updated description"
  }'

# Delete course
curl -X DELETE http://localhost:8080/api/courses/{id}

# Clone course
curl -X POST http://localhost:8080/api/courses/{id}/clone
```

### Using Postman
1. Import the API collection
2. Set the base URL to `http://localhost:8080/api`
3. Test each endpoint with appropriate request bodies
4. Verify responses match the expected format

## Database Schema

The courses table includes the following fields:
- `id`: VARCHAR(36) PRIMARY KEY
- `title`: VARCHAR(255) NOT NULL
- `description`: TEXT
- `status`: ENUM('ACTIVE', 'ARCHIVED') NOT NULL
- `tenant_id`: VARCHAR(36)
- `profile_image`: VARCHAR(500)
- `language`: VARCHAR(10)
- `created_at`: TIMESTAMP
- `updated_at`: TIMESTAMP
- `created_by`: VARCHAR(100)
- `updated_by`: VARCHAR(100)

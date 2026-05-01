# Course Profile Image Upload Guide

## 🚀 **Course Profile Image System**

### **Features:**
- ✅ **Integrated Course Creation** - Upload image directly with course creation
- ✅ **File Entity Management** - Complete file metadata tracking
- ✅ **Organized Storage** - Files stored in course-specific directory
- ✅ **Persistent Storage** - Files stored outside JAR (won't be deleted on redeploy)
- ✅ **Unique Filenames** - UUID-based naming to prevent conflicts
- ✅ **File Type Detection** - Automatic content-type detection
- ✅ **Size Limits** - 10MB max file size
- ✅ **File Relationships** - Files linked to courses with entity tracking

## 📁 **File Storage Structure**

```
your-deployment-folder/
├── course-forge-backend-1.0.0.jar
├── uploads/                          ← Files stored here (persistent)
│   └── course/                       ← Course profile images
│       ├── abc123-def456.jpg
│       └── xyz789-ghi012.png
└── application-prod.properties
```

## 🔧 **API Endpoints**

### **1. Create Course (Unified Endpoint)**
```http
POST /api/courses
Content-Type: multipart/form-data

file: [your-image-file] (optional)
title: My Course
description: Course description (optional)
status: ACTIVE
tenantId: tenant-123 (optional)
language: en (optional)
createdBy: admin (optional)
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Course created successfully",
  "data": {
    "id": "course-123",
    "title": "My Course",
    "profileImage": "/api/files/file-123-abc-def",
    "description": "Course description",
    "status": "ACTIVE",
    "createdAt": "2025-01-25T10:30:00"
  },
  "timestamp": "2025-01-25T10:30:00"
}
```

### **2. Create Course without Image (Same Endpoint)**
```http
POST /api/courses
Content-Type: multipart/form-data

title: My Course
description: Course description
status: ACTIVE
tenantId: tenant-123
language: en
createdBy: admin
```

**Response:**
```json
{
  "success": true,
  "status": 201,
  "message": "Course created successfully",
  "data": {
    "id": "course-123",
    "title": "My Course",
    "profileImage": null,
    "description": "Course description",
    "status": "ACTIVE",
    "createdAt": "2025-01-25T10:30:00"
  },
  "timestamp": "2025-01-25T10:30:00"
}
```

### **3. Download File**
```http
GET /api/files/{fileId}
```

**Example:**
```http
GET /api/files/file-123-abc-def
```

**Response:** File content with appropriate content-type

### **4. Delete Course (with all files)**
```http
DELETE /api/courses/{courseId}
```

**Response:**
```json
{
  "success": true,
  "status": 200,
  "message": "Course deleted successfully",
  "data": null,
  "timestamp": "2025-01-25T10:30:00"
}
```

**Note:** Deleting a course automatically deletes all associated files.


## 🎯 **How to Use Course Profile Images**

### **Frontend Flow:**
1. **Create course** with optional image in single request
2. **File automatically uploaded** and stored (if provided)
3. **Course created** with profile image URL (if file provided)
4. **File persists** even after JAR redeployment

### **Example Frontend Code:**
```javascript
// Create course with image (unified approach)
const formData = new FormData();
formData.append('file', selectedFile); // Optional
formData.append('title', 'My Course');
formData.append('description', 'Course description');
formData.append('status', 'ACTIVE');
formData.append('tenantId', 'tenant-123');
formData.append('language', 'en');
formData.append('createdBy', 'admin');

const courseResponse = await fetch('/api/courses', {
  method: 'POST',
  body: formData
});

const { data: course } = await courseResponse.json();
// course.profileImage contains the uploaded file URL (if file was provided)
```

### **Alternative: Create Course without Image (Same Endpoint)**
```javascript
// Create course without image (same endpoint, no file)
const formData = new FormData();
formData.append('title', 'My Course');
formData.append('description', 'Course description');
formData.append('status', 'ACTIVE');
formData.append('tenantId', 'tenant-123');
formData.append('language', 'en');
formData.append('createdBy', 'admin');

const courseResponse = await fetch('/api/courses', {
  method: 'POST',
  body: formData
});

const { data: course } = await courseResponse.json();
// course.profileImage will be null
```

### **Download Course Profile Image**
```javascript
// Download course profile image
const imageUrl = course.profileImage; // e.g., "/api/files/file-123-abc-def"
const imageResponse = await fetch(imageUrl);
const imageBlob = await imageResponse.blob();

// Display image
const imageElement = document.createElement('img');
imageElement.src = URL.createObjectURL(imageBlob);
document.body.appendChild(imageElement);
```

## ⚙️ **Configuration**

### **File Upload Settings:**
```properties
# File storage directory (relative to JAR location)
app.file.upload-dir=./uploads

# Maximum file size
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

### **Production Deployment:**
1. **Create uploads directory** in your deployment folder
2. **Set proper permissions** for file writing
3. **Files persist** across JAR redeployments

## 🔒 **Security Considerations**

- ✅ **File size limits** (10MB max)
- ✅ **Unique filenames** prevent conflicts
- ✅ **Content-type validation** for downloads
- ⚠️ **Add file type validation** if needed
- ⚠️ **Add authentication** for production use

## 🚀 **Testing the Implementation**

### **1. Start the application:**
```bash
java -Xmx512m -Xms256m -jar course-forge-backend-1.0.0.jar --spring.profiles.active=prod --server.port=8081
```

### **2. Test course creation with image:**
```bash
curl -X POST -F "file=@test-image.jpg" -F "title=Test Course" -F "description=Test Description" -F "status=ACTIVE" http://localhost:8081/api/courses
```

### **3. Test course creation without image:**
```bash
curl -X POST -F "title=Test Course" -F "description=Test Description" -F "status=ACTIVE" http://localhost:8081/api/courses
```

### **4. Test file download:**
```bash
curl http://localhost:8081/api/files/{fileId}
```

### **5. Check Swagger UI:**
Visit: `http://localhost:8081/api/swagger-ui.html`

## 📝 **Next Steps**

1. **Test the integrated course creation** functionality
2. **Integrate with frontend** course creation
3. **Add file type validation** if needed
4. **Consider cloud storage** for production scaling

The course profile image system is now ready for use! 🎉

# Logging Configuration Guide

## Overview
This application uses **Logback** for logging (included with Spring Boot). All errors and exceptions are automatically logged to files when the application is running.

## Log File Locations

When you run the JAR file, logs are written to the following locations (relative to where you run the JAR):

### Default Log Locations (Windows)
- **All Logs**: `.\logs\course-forge-backend.log`
- **Error Logs Only**: `.\logs\course-forge-backend-error.log`

### Example Paths
If you run the JAR from `D:\ABHI\OFFICE\Mundrisoft\Content Creator\course-forge-backend\`, the logs will be at:
- `D:\ABHI\OFFICE\Mundrisoft\Content Creator\course-forge-backend\logs\course-forge-backend.log`
- `D:\ABHI\OFFICE\Mundrisoft\Content Creator\course-forge-backend\logs\course-forge-backend-error.log`

## Customizing Log File Location

You can customize the log file location using environment variables:

### Windows PowerShell
```powershell
$env:LOG_FILE="D:\Logs\myapp.log"
$env:LOG_ERROR_FILE="D:\Logs\myapp-error.log"
java -jar course-forge-backend-1.0.0.jar --spring.profiles.active=prod
```

### Windows Command Prompt
```cmd
set LOG_FILE=D:\Logs\myapp.log
set LOG_ERROR_FILE=D:\Logs\myapp-error.log
java -jar course-forge-backend-1.0.0.jar --spring.profiles.active=prod
```

## Log File Rotation

- **Daily rotation**: Log files are rotated daily
- **Size limit**: Each log file is limited to 100MB before rotation
- **Retention**: 
  - All logs: 30 days
  - Error logs: 60 days
- **Compression**: Old log files are automatically compressed (.gz format)

## What Gets Logged

### Error Log File (`course-forge-backend-error.log`)
Contains only **WARN** and **ERROR** level messages, including:
- All exceptions caught by `GlobalExceptionHandler`
- Validation errors
- Database errors
- API errors
- File upload errors
- Authentication failures

### Main Log File (`course-forge-backend.log`)
Contains **all log levels** (INFO, WARN, ERROR):
- Application startup/shutdown
- API requests (INFO level)
- Business logic operations
- All errors and warnings

## Viewing Logs on Windows

### Using PowerShell
```powershell
# View latest errors
Get-Content .\logs\course-forge-backend-error.log -Tail 50

# View all logs
Get-Content .\logs\course-forge-backend.log -Tail 50

# Follow logs in real-time (like tail -f)
Get-Content .\logs\course-forge-backend-error.log -Wait -Tail 20
```

### Using Command Prompt
```cmd
# View last 50 lines of error log
powershell -Command "Get-Content .\logs\course-forge-backend-error.log -Tail 50"
```

### Using Notepad
Simply open the log files in Notepad or any text editor:
```
.\logs\course-forge-backend-error.log
.\logs\course-forge-backend.log
```

## Common Error Patterns to Look For

1. **Database Connection Errors**: Look for `SQLException`, `Connection refused`, `Access denied`
2. **File Upload Errors**: Look for `MaxUploadSizeExceededException`, `MultipartException`
3. **Authentication Errors**: Look for `Invalid or expired token`, `Unauthorized`
4. **API Errors**: Look for `OpenAI API error`, `Failed to generate`, `Error calling`

## Troubleshooting

### Logs Directory Not Created
If the `logs` directory doesn't exist, the application will create it automatically when it starts. If it fails, check:
- Write permissions in the directory where the JAR is located
- Disk space availability

### No Logs Appearing
1. Check if the application is actually running
2. Verify the JAR is running from the expected directory
3. Check Windows Event Viewer for Java errors
4. Ensure the `logs` directory is writable

### Finding Logs on Remote Server
If you're accessing the Windows server remotely:
1. Use Remote Desktop Connection
2. Navigate to the directory where the JAR is running
3. Check the `logs` subdirectory
4. Or use PowerShell remoting to access logs remotely

## Log Levels

- **ERROR**: Critical errors that need immediate attention
- **WARN**: Warning messages that may indicate problems
- **INFO**: General informational messages about application flow
- **DEBUG**: Detailed debugging information (not in production)

## Production Recommendations

1. **Monitor the error log file** regularly for issues
2. **Set up log rotation** if you need longer retention
3. **Use a log aggregation tool** (like ELK stack) for centralized logging
4. **Set up alerts** for ERROR level messages in production


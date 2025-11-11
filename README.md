# 🚀 WowMySQL Go SDK

Official Go SDK for [WowMySQL](https://wowmysql.com) - MySQL Backend-as-a-Service with S3 Storage.

[![Go Reference](https://pkg.go.dev/badge/github.com/wowmysql/wowmysql-go.svg)](https://pkg.go.dev/github.com/wowmysql/wowmysql-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/wowmysql/wowmysql-go)](https://goreportcard.com/report/github.com/wowmysql/wowmysql-go)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## ✨ Features

### Database Features
- 🗄️ Full CRUD operations (Create, Read, Update, Delete)
- 🔍 Advanced filtering (eq, neq, gt, gte, lt, lte, like, isNull)
- 📄 Pagination (limit, offset)
- 📊 Sorting (orderBy)
- 🎯 Fluent query builder API
- 🔒 Type-safe queries
- 📝 Raw SQL queries
- 📋 Table schema introspection
- 🛡️ Comprehensive error handling

### Storage Features
- 📦 S3-compatible storage for file management
- ⬆️ File upload with automatic quota validation
- ⬇️ File download (presigned URLs)
- 📂 File listing with metadata
- 🗑️ File deletion (single and batch)
- 📊 Storage quota management
- 🌍 Multi-region S3 support
- 🛡️ Client-side limit enforcement

## 📦 Installation

```bash
go get github.com/wowmysql/wowmysql-go/wowmysql
```

## 🚀 Quick Start

### Database Operations

```go
package main

import (
    "fmt"
    "log"
    
    "github.com/wowmysql/wowmysql-go/wowmysql"
)

func main() {
    // Initialize client
    client := wowmysql.NewClient(
        "https://your-project.wowmysql.com",
        "your-api-key",
    )

    // Query data
    users, err := client.Table("users").
        Select("id", "name", "email").
        Eq("status", "active").
        Limit(10).
        Execute()
    
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Found %d users\n", users.Count)
    for _, user := range users.Data {
        fmt.Printf("%s - %s\n", user["name"], user["email"])
    }
}
```

### Storage Operations

```go
package main

import (
    "fmt"
    "log"
    "os"
    
    "github.com/wowmysql/wowmysql-go/wowmysql"
)

func main() {
    // Initialize storage client
    storage := wowmysql.NewStorageClient(
        "https://your-project.wowmysql.com",
        "your-api-key",
    )

    // Upload file
    fileData, _ := os.ReadFile("document.pdf")
    result, err := storage.Upload(
        fileData,
        "uploads/document.pdf",
        "application/pdf",
        nil,
    )
    
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Uploaded: %s\n", result.URL)

    // Check quota
    quota, err := storage.GetQuota()
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Storage used: %.2fGB / %.2fGB\n", 
        quota.StorageUsedGB, 
        quota.StorageQuotaGB)
}
```

### Project Authentication

```go
package main

import (
    "fmt"
    "log"

    "github.com/wowmysql/wowmysql-go/wowmysql"
)

func main() {
    auth := wowmysql.NewAuthClient(wowmysql.AuthConfig{
        ProjectURL:   "https://your-project.wowmysql.com",
        PublicAPIKey: "public-api-key",
    })

    // Sign up an end user
    result, err := auth.SignUp("user@example.com", "SuperSecret123",
        wowmysql.WithFullName("End User"))
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("User ID:", result.User.ID)
    fmt.Println("Access token:", result.Session.AccessToken)

    // Fetch the same user via stored session token
    user, err := auth.GetUser(result.Session.AccessToken)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Email verified:", user.EmailVerified)

    // OAuth Authentication
    oauthResp, err := auth.GetOAuthAuthorizationURL("github", "https://app.example.com/auth/callback")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Send user to:", oauthResp.AuthorizationURL)

    // After callback, exchange code for tokens
    redirectURI := "https://app.example.com/auth/callback"
    oauthResult, err := auth.ExchangeOAuthCallback("github", "authorization_code", &redirectURI)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("OAuth user:", oauthResult.User.Email)

    // Password Reset
    forgotResult, err := auth.ForgotPassword("user@example.com")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(forgotResult["message"])

    resetResult, err := auth.ResetPassword("reset_token", "newSecurePassword123")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(resetResult["message"])
}
```

## 📚 Usage Examples

### Select Queries

```go
client := wowmysql.NewClient(
    "https://your-project.wowmysql.com",
    "your-api-key",
)

// Select all columns
all, err := client.Table("users").Select("*").Execute()

// Select specific columns
users, err := client.Table("users").
    Select("id", "name", "email").
    Execute()

// With filters
active, err := client.Table("users").
    Select("*").
    Eq("status", "active").
    Gt("age", 18).
    Execute()

// With ordering
recent, err := client.Table("users").
    Select("*").
    OrderBy("created_at", wowmysql.SortDesc).
    Limit(10).
    Execute()

// With pagination
page1, err := client.Table("users").
    Select("*").
    Limit(20).
    Offset(0).
    Execute()

// Pattern matching
gmailUsers, err := client.Table("users").
    Select("*").
    Like("email", "%@gmail.com").
    Execute()

// Get first result
user, err := client.Table("users").
    Eq("email", "john@example.com").
    First()

if user != nil {
    fmt.Printf("Found user: %s\n", user["name"])
}
```

### Insert Data

```go
// Insert single record
result, err := client.Table("users").Insert(map[string]interface{}{
    "name":   "John Doe",
    "email":  "john@example.com",
    "age":    30,
    "status": "active",
})

if err != nil {
    log.Fatal(err)
}

fmt.Printf("New user ID: %v\n", result.ID)
```

### Update Data

```go
// Update by ID
result, err := client.Table("users").UpdateByID(1, map[string]interface{}{
    "name": "Jane Smith",
    "age":  26,
})

fmt.Printf("Updated %d row(s)\n", result.AffectedRows)

// Update with conditions
updated, err := client.Table("users").
    Where().
    Eq("status", "inactive").
    Update(map[string]interface{}{
        "status": "active",
    })
```

### Delete Data

```go
// Delete by ID
result, err := client.Table("users").DeleteByID(1)

fmt.Printf("Deleted %d row(s)\n", result.AffectedRows)

// Delete with conditions
deleted, err := client.Table("users").
    Where().
    Eq("status", "deleted").
    Delete()
```

### Filter Operators

```go
// Equal
.Eq("status", "active")

// Not equal
.Neq("status", "deleted")

// Greater than
.Gt("age", 18)

// Greater than or equal
.Gte("age", 18)

// Less than
.Lt("age", 65)

// Less than or equal
.Lte("age", 65)

// Pattern matching (SQL LIKE)
.Like("email", "%@gmail.com")

// Is null
.IsNull("deleted_at")
```

### Storage Operations

```go
storage := wowmysql.NewStorageClient(
    "https://your-project.wowmysql.com",
    "your-api-key",
)

// Upload file from bytes
fileData, _ := os.ReadFile("document.pdf")
uploadResult, err := storage.Upload(
    fileData,
    "uploads/2024/document.pdf",
    "application/pdf",
    nil,
)
fmt.Printf("Uploaded: %s\n", uploadResult.URL)

// Check if file exists
exists, err := storage.FileExists("uploads/document.pdf")
if exists {
    fmt.Println("File exists!")
}

// Get file information
info, err := storage.GetFileInfo("uploads/document.pdf")
fmt.Printf("Size: %d bytes\n", info.Size)
fmt.Printf("Modified: %s\n", info.LastModified)

// List files with prefix
files, err := storage.ListFiles("uploads/2024/", 0)
for _, file := range files {
    fmt.Printf("%s: %d bytes\n", file.Key, file.Size)
}

// Download file (get presigned URL)
url, err := storage.Download("uploads/document.pdf", 3600)
fmt.Printf("Download URL: %s\n", url)
// URL is valid for 1 hour

// Delete single file
err = storage.DeleteFile("uploads/old-file.pdf")

// Delete multiple files
err = storage.DeleteFiles([]string{
    "uploads/file1.pdf",
    "uploads/file2.pdf",
    "uploads/file3.pdf",
})

// Check quota
quota, err := storage.GetQuota()
fmt.Printf("Used: %.2f GB\n", quota.StorageUsedGB)
fmt.Printf("Available: %.2f GB\n", quota.StorageAvailableGB)
fmt.Printf("Usage: %.1f%%\n", quota.UsagePercentage)

// Check if enough storage before upload
if quota.StorageAvailableBytes < int64(len(fileData)) {
    fmt.Println("Not enough storage!")
} else {
    storage.Upload(fileData, "uploads/large-file.zip", "", nil)
}
```

### Error Handling

```go
import (
    "errors"
    "github.com/wowmysql/wowmysql-go/wowmysql"
)

users, err := client.Table("users").Select("*").Execute()
if err != nil {
    var authErr *wowmysql.AuthenticationError
    var notFoundErr *wowmysql.NotFoundError
    var rateLimitErr *wowmysql.RateLimitError
    var networkErr *wowmysql.NetworkError
    
    switch {
    case errors.As(err, &authErr):
        fmt.Printf("Authentication error: %s\n", authErr.Message)
    case errors.As(err, &notFoundErr):
        fmt.Printf("Not found: %s\n", notFoundErr.Message)
    case errors.As(err, &rateLimitErr):
        fmt.Printf("Rate limit exceeded: %s\n", rateLimitErr.Message)
    case errors.As(err, &networkErr):
        fmt.Printf("Network error: %s\n", err)
    default:
        fmt.Printf("Error: %s\n", err)
    }
}

// Storage errors
_, err = storage.Upload(fileData, "uploads/file.pdf", "", nil)
if err != nil {
    var limitErr *wowmysql.StorageLimitExceededError
    var storageErr *wowmysql.StorageError
    
    switch {
    case errors.As(err, &limitErr):
        fmt.Printf("Storage full: %s\n", limitErr.Message)
        fmt.Printf("Required: %d bytes\n", limitErr.RequiredBytes)
        fmt.Printf("Available: %d bytes\n", limitErr.AvailableBytes)
    case errors.As(err, &storageErr):
        fmt.Printf("Storage error: %s\n", storageErr.Message)
    }
}
```

### Utility Methods

```go
// List all tables
tables, err := client.ListTables()
fmt.Printf("Tables: %v\n", tables)

// Get table schema
schema, err := client.GetTableSchema("users")
fmt.Printf("Columns: %d\n", len(schema.Columns))
for _, column := range schema.Columns {
    fmt.Printf("  - %s (%s)\n", column.Name, column.Type)
}

// Raw SQL query
results, err := client.Query("SELECT COUNT(*) as count FROM users WHERE age > 18")
if len(results) > 0 {
    fmt.Printf("Adult users: %v\n", results[0]["count"])
}

// Check API health
health, err := client.Health()
fmt.Printf("Status: %v\n", health["status"])
```

## 🔧 Configuration

### Custom Timeout

```go
import "time"

// Database client with custom timeout
client := wowmysql.NewClientWithTimeout(
    "https://your-project.wowmysql.com",
    "your-api-key",
    60 * time.Second, // 60 seconds
)

// Storage client with custom timeout
storage := wowmysql.NewStorageClientWithOptions(
    "https://your-project.wowmysql.com",
    "your-api-key",
    120 * time.Second, // 2 minutes for large files
    true, // auto check quota
)
```

### Auto Quota Check

```go
// Disable automatic quota checking
storage := wowmysql.NewStorageClientWithOptions(
    "https://your-project.wowmysql.com",
    "your-api-key",
    60 * time.Second,
    false, // disable auto-check
)

// Manually check quota
quota, err := storage.GetQuota()
if quota.StorageAvailableBytes > int64(len(fileData)) {
    checkQuota := false
    storage.Upload(fileData, "uploads/file.pdf", "", &checkQuota)
}
```

## 🔑 API Keys

WowMySQL uses **different API keys for different operations**. Understanding which key to use is crucial for proper authentication.

### Key Types Overview

| Operation Type | Recommended Key | Alternative Key | Used By |
|---------------|----------------|-----------------|---------|
| **Database Operations** (CRUD) | Service Role Key (`wowbase_service_...`) | Anonymous Key (`wowbase_anon_...`) | `WowMySQLClient` |
| **Authentication Operations** (OAuth, sign-in) | Public API Key (`wowbase_auth_...`) | Service Role Key (`wowbase_service_...`) | `ProjectAuthClient` |

### Where to Find Your Keys

All keys are found in: **WowMySQL Dashboard → Authentication → PROJECT KEYS**

1. **Service Role Key** (`wowbase_service_...`)
   - Location: "Service Role Key (keep secret)"
   - Used for: Database CRUD operations (recommended for server-side)
   - Can also be used for authentication operations (fallback)
   - **Important**: Click the eye icon to reveal this key

2. **Public API Key** (`wowbase_auth_...`)
   - Location: "Public API Key"
   - Used for: OAuth, sign-in, sign-up, user management
   - Recommended for client-side/public authentication flows

3. **Anonymous Key** (`wowbase_anon_...`)
   - Location: "Anonymous Key"
   - Used for: Public/client-side database operations with limited permissions
   - Optional: Use when exposing database access to frontend/client

### Database Operations

Use **Service Role Key** or **Anonymous Key** for database operations:

```go
package main

import "github.com/wowmysql/wowmysql-go/wowmysql"

// Using Service Role Key (recommended for server-side, full access)
client := wowmysql.NewClient(
    "https://your-project.wowmysql.com",
    "wowbase_service_your-service-key-here",  // Service Role Key
)

// Using Anonymous Key (for public/client-side access with limited permissions)
client := wowmysql.NewClient(
    "https://your-project.wowmysql.com",
    "wowbase_anon_your-anon-key-here",  // Anonymous Key
)

// Query data
users, err := client.Table("users").Execute()
```

### Authentication Operations

Use **Public API Key** or **Service Role Key** for authentication:

```go
package main

import "github.com/wowmysql/wowmysql-go/wowmysql"

// Using Public API Key (recommended for OAuth, sign-in, sign-up)
auth := wowmysql.NewAuthClient(wowmysql.AuthConfig{
    ProjectURL:   "https://your-project.wowmysql.com",
    PublicAPIKey: "wowbase_auth_your-public-key-here",  // Public API Key
})

// Using Service Role Key (can be used for auth operations too)
auth := wowmysql.NewAuthClient(wowmysql.AuthConfig{
    ProjectURL:   "https://your-project.wowmysql.com",
    PublicAPIKey: "wowbase_service_your-service-key-here",  // Service Role Key
})

// OAuth authentication
oauthResp, err := auth.GetOAuthAuthorizationURL("github", "https://app.example.com/auth/callback")
```

### Environment Variables

Best practice: Use environment variables for API keys:

```go
package main

import (
    "os"
    "github.com/wowmysql/wowmysql-go/wowmysql"
)

// Database operations - Service Role Key
dbClient := wowmysql.NewClient(
    os.Getenv("WOWMYSQL_PROJECT_URL"),
    os.Getenv("WOWMYSQL_SERVICE_ROLE_KEY"),  // or WOWMYSQL_ANON_KEY
)

// Authentication operations - Public API Key
authClient := wowmysql.NewAuthClient(wowmysql.AuthConfig{
    ProjectURL:   os.Getenv("WOWMYSQL_PROJECT_URL"),
    PublicAPIKey: os.Getenv("WOWMYSQL_PUBLIC_API_KEY"),
})
```

### Key Usage Summary

- **`WowMySQLClient`** → Uses **Service Role Key** or **Anonymous Key** for database operations
- **`ProjectAuthClient`** → Uses **Public API Key** or **Service Role Key** for authentication operations
- **Service Role Key** can be used for both database AND authentication operations
- **Public API Key** is specifically for authentication operations only
- **Anonymous Key** is optional and provides limited permissions for public database access

### Security Best Practices

1. **Never expose Service Role Key** in client-side code or public repositories
2. **Use Public API Key** for client-side authentication flows
3. **Use Anonymous Key** for public database access with limited permissions
4. **Store keys in environment variables**, never hardcode them
5. **Rotate keys regularly** if compromised

### Troubleshooting

**Error: "Invalid API key for project"**
- Ensure you're using the correct key type for the operation
- Database operations require Service Role Key or Anonymous Key
- Authentication operations require Public API Key or Service Role Key
- Verify the key is copied correctly (no extra spaces)

**Error: "Authentication failed"**
- Check that you're using Public API Key (not Anonymous Key) for auth operations
- Verify the project URL matches your dashboard
- Ensure the key hasn't been revoked or expired

## 📋 Requirements

- Go: `1.21+`

## 🔗 Links

- 📚 [Documentation](https://wowmysql.com/docs)
- 🌐 [Website](https://wowmysql.com)
- 💬 [Discord](https://discord.gg/wowmysql)
- 🐛 [Issues](https://github.com/wowmysql/wowmysql/issues)

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## 📞 Support

- Email: support@wowmysql.com
- Discord: https://discord.gg/wowmysql
- Documentation: https://wowmysql.com/docs

---

Made with ❤️ by the WowMySQL Team


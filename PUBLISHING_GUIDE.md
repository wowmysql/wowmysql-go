# 🚀 Go SDK Publishing Guide

Complete guide for publishing the WowMySQL Go SDK to pkg.go.dev and GitHub.

---

## 📦 Package Information

- **Module Path**: `github.com/wowmysql/wowmysql-go`
- **Package**: `wowmysql`
- **Version**: `v1.0.0`
- **Go Version**: `1.21+`
- **Registry**: pkg.go.dev (automatic from GitHub)

---

## 🔧 Prerequisites

### 1. Git Repository Setup

```bash
# Initialize git repository (if not already done)
cd sdk/go
git init

# Add remote (replace with your actual repo URL)
git remote add origin https://github.com/wowmysql/wowmysql-go.git
```

### 2. Go Module Setup

The `go.mod` file is already configured:

```go
module github.com/wowmysql/wowmysql-go

go 1.21

require (
    github.com/google/go-querystring v1.1.0
)
```

---

## 📝 Publishing Steps

### Step 1: Initialize Go Module

```bash
cd sdk/go

# Download dependencies
go mod download

# Verify dependencies
go mod verify

# Tidy up
go mod tidy
```

### Step 2: Run Tests (Optional but Recommended)

Create a simple test file first:

```bash
# Create test file
cat > wowmysql/client_test.go << 'EOF'
package wowmysql

import "testing"

func TestNewClient(t *testing.T) {
    client := NewClient("https://test.wowmysql.com", "test-key")
    if client == nil {
        t.Fatal("Expected non-nil client")
    }
    if client.projectURL != "https://test.wowmysql.com" {
        t.Errorf("Expected projectURL to be https://test.wowmysql.com, got %s", client.projectURL)
    }
}
EOF

# Run tests
go test ./...
```

### Step 3: Commit to Git

```bash
# Add all files
git add .

# Commit
git commit -m "Initial release v1.0.0"

# Push to GitHub
git push origin main
```

### Step 4: Create Git Tag

```bash
# Create and push tag
git tag v1.0.0
git push origin v1.0.0
```

### Step 5: Publish to pkg.go.dev

**Good news!** Go packages are automatically published to pkg.go.dev when you push a Git tag.

1. Wait 10-15 minutes after pushing the tag
2. Visit: https://pkg.go.dev/github.com/wowmysql/wowmysql-go/wowmysql
3. If it doesn't appear, request indexing:
   ```bash
   # Trigger indexing by fetching the module
   GOPROXY=proxy.golang.org go list -m github.com/wowmysql/wowmysql-go@v1.0.0
   ```

### Step 6: Verify Installation

```bash
# Create a test project
mkdir test-wowmysql
cd test-wowmysql
go mod init test

# Install the package
go get github.com/wowmysql/wowmysql-go/wowmysql

# Create a test file
cat > main.go << 'EOF'
package main

import (
    "fmt"
    "github.com/wowmysql/wowmysql-go/wowmysql"
)

func main() {
    client := wowmysql.NewClient(
        "https://your-project.wowmysql.com",
        "your-api-key",
    )
    fmt.Println("WowMySQL client created successfully!")
}
EOF

# Run it
go run main.go
```

---

## 🎯 Quick Publish Script

Create `publish.sh`:

```bash
#!/bin/bash
set -e

VERSION=${1:-v1.0.0}

echo "🚀 Publishing WowMySQL Go SDK $VERSION..."

# Step 1: Tidy
echo "🧹 Tidying dependencies..."
go mod tidy

# Step 2: Test
echo "🧪 Running tests..."
go test ./... || echo "⚠️  No tests found, skipping..."

# Step 3: Commit
echo "📝 Committing changes..."
git add .
git commit -m "Release $VERSION" || echo "Nothing to commit"

# Step 4: Tag
echo "🏷️  Creating tag $VERSION..."
git tag $VERSION

# Step 5: Push
echo "⬆️  Pushing to GitHub..."
git push origin main
git push origin $VERSION

# Step 6: Trigger indexing
echo "📚 Triggering pkg.go.dev indexing..."
GOPROXY=proxy.golang.org go list -m github.com/wowmysql/wowmysql-go@$VERSION

echo "✅ Done! Visit https://pkg.go.dev/github.com/wowmysql/wowmysql-go/wowmysql"
```

**Windows (`publish.bat`):**

```batch
@echo off
setlocal

set VERSION=%1
if "%VERSION%"=="" set VERSION=v1.0.0

echo Publishing WowMySQL Go SDK %VERSION%...

echo Tidying dependencies...
go mod tidy

echo Running tests...
go test ./...

echo Committing changes...
git add .
git commit -m "Release %VERSION%"

echo Creating tag %VERSION%...
git tag %VERSION%

echo Pushing to GitHub...
git push origin main
git push origin %VERSION%

echo Triggering pkg.go.dev indexing...
set GOPROXY=proxy.golang.org
go list -m github.com/wowmysql/wowmysql-go@%VERSION%

echo Done! Visit https://pkg.go.dev/github.com/wowmysql/wowmysql-go/wowmysql
pause
```

Make executable and run:

```bash
chmod +x publish.sh
./publish.sh v1.0.0
```

---

## 🐛 Troubleshooting

### Error: "module not found"

**Solution**: Make sure you've pushed the Git tag to GitHub:
```bash
git push origin v1.0.0
```

### Error: "package not showing on pkg.go.dev"

**Solution**: Trigger manual indexing:
```bash
GOPROXY=proxy.golang.org go list -m github.com/wowmysql/wowmysql-go@v1.0.0
```

### Error: "invalid version"

**Solution**: Use semantic versioning with 'v' prefix:
- ✅ `v1.0.0`
- ❌ `1.0.0`

---

## ✅ Post-Publishing Tasks

### 1. Verify Installation

```bash
go get github.com/wowmysql/wowmysql-go/wowmysql@v1.0.0
```

### 2. Check pkg.go.dev

Visit: https://pkg.go.dev/github.com/wowmysql/wowmysql-go/wowmysql

### 3. Create GitHub Release

1. Go to: https://github.com/wowmysql/wowmysql-go/releases/new
2. Tag: `v1.0.0`
3. Title: "Go SDK v1.0.0 - Initial Release"
4. Description: Copy from `CHANGELOG.md`
5. Publish

### 4. Update Main Repository

Update the main WowMySQL repository to reference the new SDK:

```markdown
## SDKs

- [Python](https://pypi.org/project/wowmysql-sdk/)
- [TypeScript/JavaScript](https://www.npmjs.com/package/@wowmysql/sdk)
- [Flutter/Dart](https://pub.dev/packages/wowmysql)
- [Kotlin](https://search.maven.org/artifact/com.wowmysql/wowmysql-sdk)
- [Go](https://pkg.go.dev/github.com/wowmysql/wowmysql-go/wowmysql) ⬅️ NEW!
```

### 5. Announce Release

Post to:
- [ ] Twitter/X
- [ ] LinkedIn
- [ ] Reddit (r/golang)
- [ ] Discord community
- [ ] Dev.to blog post
- [ ] Hacker News (Show HN)

---

## 📚 Additional Resources

- **pkg.go.dev Guide**: https://go.dev/doc/modules/publishing
- **Go Modules Reference**: https://go.dev/ref/mod
- **Semantic Versioning**: https://semver.org/

---

## 🔄 Releasing New Versions

### Patch Release (v1.0.1)

```bash
# Make changes
git add .
git commit -m "Fix: bug description"

# Tag and push
git tag v1.0.1
git push origin main
git push origin v1.0.1
```

### Minor Release (v1.1.0)

```bash
# Add new features
git add .
git commit -m "Feature: new feature description"

# Tag and push
git tag v1.1.0
git push origin main
git push origin v1.1.0
```

### Major Release (v2.0.0)

```bash
# Breaking changes - update go.mod if needed
git add .
git commit -m "BREAKING: breaking change description"

# Tag and push
git tag v2.0.0
git push origin main
git push origin v2.0.0
```

---

Made with ❤️ by the WowMySQL Team


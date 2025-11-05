# Minify Package

[![BSD-3-Clause](https://img.shields.io/badge/BSD--3--Clause-green?style=flat)](https://github.com/valksor/php-bundle/blob/master/LICENSE)
[![Coverage Status](https://coveralls.io/repos/github/valksor/go-minify/badge.svg?branch=master)](https://coveralls.io/github/valksor/go-minify?branch=master)

A Go package that provides JavaScript and CSS minification capabilities with content-based hashing for cache-busting. It supports both bundle-based and single-file workflows, with automatic versioning and cleanup of old files.

## Features

- **Bundle System**: Configure multiple bundles with different file patterns via JSON
- **Content-Based Hashing**: Automatic hash generation for cache-busting using xxhash
- **File Type Support**: JavaScript and CSS minification with optimized output
- **Glob Pattern Support**: Flexible file selection using standard glob patterns
- **Automatic Versioning**: Content-based versioning prevents cache issues
- **Cleanup Management**: Automatic removal of old bundle versions
- **Single File Processing**: Individual file minification outside of bundle system
- **Error Handling**: Comprehensive error handling with detailed messages
- **Performance Optimized**: Fast hashing and minification for large files

## Installation

```bash
go get github.com/valksor/go-minify
```

## Quick Start

### Basic Bundle Processing

```go
package main

import (
    "log"
    "github.com/valksor/go-minify"
)

func main() {
    config := minify.Config{
        BundlesFile: "bundles.json",
        OutputDir:   "./assets/static",
    }
    
    if err := minify.ProcessBundles(config); err != nil {
        log.Fatal(err)
    }
}
```

### Bundle Configuration File

Create a `bundles.json` file to define your bundles:

```json
{
    "bundles": [
        {
            "name": "base",
            "files": [
                "assets/js/utils.js",
                "assets/js/components/*.js",
                "assets/js/main.js"
            ]
        },
        {
            "name": "admin",
            "files": [
                "assets/js/admin/*.js"
            ]
        }
    ]
}
```

### Single File Processing

```go
package main

import (
    "fmt"
    "log"
    "github.com/valksor/go-minify"
)

func main() {
    // Minify a CSS file
    filename, err := minify.AndVersionCSS("assets/css/main.css", "public/css")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Minified CSS: %s\n", filename) // Output: "main.a1b2c3d4.css"
    
    // Minify a JavaScript file
    filename, err = minify.AndVersionFile("assets/js/app.js", "public/js", "js")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Minified JS: %s\n", filename) // Output: "app.a1b2c3d4.min.js"
}
```

## API Reference

### Core Types

#### Config

```go
type Config struct {
    BundlesFile string  // Path to bundles configuration JSON file
    OutputDir   string  // Directory where minified bundles will be written
}
```

#### Bundle

```go
type Bundle struct {
    Name  string   `json:"name"`   // Bundle name (used in output filename)
    Files []string `json:"files"`  // Array of file patterns to include
}
```

#### BundleConfig

```go
type BundleConfig struct {
    Bundles []Bundle `json:"bundles"`  // Array of bundle definitions
}
```

### Core Functions

#### ProcessBundles(config Config) error

Processes all bundles defined in the configuration file. This is the main function for bundle-based workflow.

**Parameters:**
- `config` - Configuration specifying bundles file and output directory

**Returns:**
- `error` - Any error that occurred during processing

**Example:**
```go
config := minify.Config{
    BundlesFile: "config/bundles.json",
    OutputDir:   "public/assets",
}

if err := minify.ProcessBundles(config); err != nil {
    log.Fatalf("Failed to process bundles: %v", err)
}
```

#### GetBundleHash(bundleName, bundlesFile string) (string, error)

Calculates the content-based hash for a specific bundle. Useful for generating cache-busting URLs.

**Parameters:**
- `bundleName` - Name of the bundle to hash
- `bundlesFile` - Path to the bundles configuration file

**Returns:**
- `string` - 8-character hash string in base36 format
- `error` - Any error that occurred

**Example:**
```go
hash, err := minify.GetBundleHash("base", "bundles.json")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Bundle hash: %s\n", hash) // Output: "a1b2c3d4"
```

#### GetBundleFilename(bundleName, bundlesFile string) (string, error)

Gets the complete filename including hash for a bundle.

**Parameters:**
- `bundleName` - Name of the bundle
- `bundlesFile` - Path to the bundles configuration file

**Returns:**
- `string` - Complete filename (e.g., "base.a1b2c3d4.min.js")
- `error` - Any error that occurred

**Example:**
```go
filename, err := minify.GetBundleFilename("base", "bundles.json")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Bundle filename: %s\n", filename) // Output: "base.a1b2c3d4.min.js"
```

#### BundleExists(bundleName, bundlesFile, outputDir string) (bool, error)

Checks if a bundle file already exists in the output directory.

**Parameters:**
- `bundleName` - Name of the bundle to check
- `bundlesFile` - Path to the bundles configuration file
- `outputDir` - Output directory to check

**Returns:**
- `bool` - Whether the bundle exists
- `error` - Any error that occurred

**Example:**
```go
exists, err := minify.BundleExists("base", "bundles.json", "./assets/static")
if err != nil {
    log.Fatal(err)
}
if !exists {
    // Bundle needs to be generated
    err = minify.ProcessBundles(config)
}
```

#### CleanOldBundles(bundleName, bundlesFile, outputDir string) error

Removes old versions of a bundle, keeping only the current version.

**Parameters:**
- `bundleName` - Name of the bundle to clean
- `bundlesFile` - Path to the bundles configuration file
- `outputDir` - Directory containing bundle files

**Returns:**
- `error` - Any error that occurred during cleanup

**Example:**
```go
err := minify.CleanOldBundles("base", "bundles.json", "./assets/static")
if err != nil {
    log.Printf("Warning: Failed to clean old bundles: %v", err)
}
```

#### AndVersionFile(inputPath, outputDir, fileType string) (string, error)

Minifies a single file and creates a versioned copy with content-based hashing.

**Parameters:**
- `inputPath` - Path to the input file to minify
- `outputDir` - Directory where the minified file should be written
- `fileType` - Type of file to minify ("css" or "js")

**Returns:**
- `string` - The filename of the created minified file
- `error` - Any error that occurred during processing

**Example:**
```go
filename, err := minify.AndVersionFile("assets/css/main.css", "public/css", "css")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Minified CSS: %s\n", filename) // Output: "main.a1b2c3d4.css"
```

#### AndVersionCSS(inputPath, outputDir string) (string, error)

Convenience function for minifying CSS files. Wrapper around `AndVersionFile` for CSS.

**Parameters:**
- `inputPath` - Path to the input CSS file
- `outputDir` - Directory where the minified CSS file should be written

**Returns:**
- `string` - The filename of the created minified CSS file
- `error` - Any error that occurred during processing

**Example:**
```go
filename, err := minify.AndVersionCSS("assets/css/main.css", "public/css")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Minified CSS: %s\n", filename) // Output: "main.a1b2c3d4.css"
```

## Usage Examples

### Example 1: Basic Bundle Processing

```go
package main

import (
    "fmt"
    "log"
    "github.com/valksor/go-minify"
)

func main() {
    config := minify.Config{
        BundlesFile: "config/bundles.json",
        OutputDir:   "public/assets",
    }
    
    if err := minify.ProcessBundles(config); err != nil {
        log.Fatalf("Failed to process bundles: %v", err)
    }
    
    fmt.Println("All bundles processed successfully!")
}
```

### Example 2: Conditional Bundle Generation

```go
package main

import (
    "fmt"
    "log"
    "github.com/valksor/go-minify"
)

func buildIfNeeded(bundleName string) error {
    config := minify.Config{
        BundlesFile: "bundles.json",
        OutputDir:   "./assets",
    }
    
    exists, err := minify.BundleExists(bundleName, config.BundlesFile, config.OutputDir)
    if err != nil {
        return err
    }
    
    if !exists {
        fmt.Printf("Bundle %s doesn't exist, generating...\n", bundleName)
        return minify.ProcessBundles(config)
    }
    
    fmt.Printf("Bundle %s already exists\n", bundleName)
    return nil
}

func main() {
    bundles := []string{"base", "admin", "vendor"}
    
    for _, bundle := range bundles {
        if err := buildIfNeeded(bundle); err != nil {
            log.Fatalf("Failed to build bundle %s: %v", bundle, err)
        }
    }
}
```

### Example 3: Template Integration

```go
package main

import (
    "fmt"
    "html/template"
    "log"
    "net/http"
    "github.com/valksor/go-minify"
)

type PageData struct {
    Title    string
    CSSFiles []string
    JSFiles  []string
}

func getAssetURL(bundleName, baseURL string) (string, error) {
    filename, err := minify.GetBundleFilename(bundleName, "bundles.json")
    if err != nil {
        return "", err
    }
    return baseURL + "/assets/" + filename, nil
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    cssURL, err := getAssetURL("base", "")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    jsURL, err := getAssetURL("base", "")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    data := PageData{
        Title:    "Home Page",
        CSSFiles: []string{cssURL},
        JSFiles:  []string{jsURL},
    }
    
    tmpl := template.Must(template.New("home").Parse(`
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
    {{range .CSSFiles}}
    <link rel="stylesheet" href="{{.}}">
    {{end}}
</head>
<body>
    <h1>{{.Title}}</h1>
    {{range .JSFiles}}
    <script src="{{.}}"></script>
    {{end}}
</body>
</html>
    `))
    
    tmpl.Execute(w, data)
}

func main() {
    // Process bundles first
    config := minify.Config{
        BundlesFile: "bundles.json",
        OutputDir:   "public/assets",
    }
    
    if err := minify.ProcessBundles(config); err != nil {
        log.Fatalf("Failed to process bundles: %v", err)
    }
    
    // Serve static files
    http.Handle("/assets/", http.StripPrefix("/assets/", http.FileServer(http.Dir("public/assets"))))
    
    // Serve home page
    http.HandleFunc("/", homeHandler)
    
    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Example 4: Build Script Integration

```go
package main

import (
    "fmt"
    "log"
    "os"
    "github.com/valksor/go-minify"
)

func main() {
    // Get configuration from environment or use defaults
    bundlesFile := os.Getenv("BUNDLES_FILE")
    if bundlesFile == "" {
        bundlesFile = "bundles.json"
    }
    
    outputDir := os.Getenv("OUTPUT_DIR")
    if outputDir == "" {
        outputDir = "public/assets"
    }
    
    config := minify.Config{
        BundlesFile: bundlesFile,
        OutputDir:   outputDir,
    }
    
    fmt.Printf("Processing bundles from %s to %s\n", config.BundlesFile, config.OutputDir)
    
    if err := minify.ProcessBundles(config); err != nil {
        log.Fatalf("Failed to process bundles: %v", err)
    }
    
    // Clean up old versions
    bundles := []string{"base", "admin", "vendor"}
    for _, bundle := range bundles {
        if err := minify.CleanOldBundles(bundle, config.BundlesFile, config.OutputDir); err != nil {
            log.Printf("Warning: Failed to clean old bundles for %s: %v", bundle, err)
        }
    }
    
    fmt.Println("Bundle processing completed successfully!")
}
```

### Example 5: Single File Processing

```go
package main

import (
    "fmt"
    "log"
    "path/filepath"
    "github.com/valksor/go-minify"
)

func processStaticAssets() error {
    // Process CSS files
    cssFiles := []string{
        "assets/css/main.css",
        "assets/css/admin.css",
        "assets/css/vendor.css",
    }
    
    for _, cssFile := range cssFiles {
        filename, err := minify.AndVersionCSS(cssFile, "public/css")
        if err != nil {
            return fmt.Errorf("failed to process CSS file %s: %w", cssFile, err)
        }
        fmt.Printf("Processed CSS: %s -> %s\n", cssFile, filename)
    }
    
    // Process JavaScript files
    jsFiles := []string{
        "assets/js/main.js",
        "assets/js/admin.js",
        "assets/js/vendor.js",
    }
    
    for _, jsFile := range jsFiles {
        filename, err := minify.AndVersionFile(jsFile, "public/js", "js")
        if err != nil {
            return fmt.Errorf("failed to process JS file %s: %w", jsFile, err)
        }
        fmt.Printf("Processed JS: %s -> %s\n", jsFile, filename)
    }
    
    return nil
}

func main() {
    if err := processStaticAssets(); err != nil {
        log.Fatalf("Failed to process static assets: %v", err)
    }
    
    fmt.Println("Static asset processing completed!")
}
```

## Configuration

### Bundle Configuration Format

The bundle configuration file is a JSON file that defines how files should be grouped and processed:

```json
{
    "bundles": [
        {
            "name": "base",
            "files": [
                "assets/js/utils.js",
                "assets/js/components/*.js",
                "assets/js/main.js"
            ]
        },
        {
            "name": "admin",
            "files": [
                "assets/js/admin/*.js",
                "assets/js/admin/components/*.js"
            ]
        },
        {
            "name": "vendor",
            "files": [
                "node_modules/jquery/dist/jquery.min.js",
                "node_modules/bootstrap/dist/js/bootstrap.min.js"
            ]
        }
    ]
}
```

### Glob Pattern Support

The package supports standard Go glob patterns:

- `*.js` - All JavaScript files in the current directory
- `**/*.js` - All JavaScript files in current and subdirectories (requires shell expansion)
- `assets/js/*.js` - All JavaScript files in assets/js directory
- `assets/js/components/*.js` - All JavaScript files in assets/js/components directory
- `assets/js/main.js` - Specific file

### File Naming Convention

Generated files follow a consistent naming convention:

- **Bundles**: `{bundle_name}.{8_char_hash}.min.js`
- **CSS Files**: `{base_name}.{8_char_hash}.css`
- **JS Files**: `{base_name}.{8_char_hash}.min.js`

Examples:
- `base.a1b2c3d4.min.js`
- `main.x9y8z7w6.css`
- `admin.m5n4o3p2.min.js`

## Error Handling

The package provides comprehensive error handling for common scenarios:

### Bundle Configuration Errors

```go
// File not found
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to read bundle config file" error
}

// Invalid JSON
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to unmarshal bundle config" error
}
```

### File Processing Errors

```go
// Glob pattern errors
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to glob pattern" error
}

// No files found
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "no files found for pattern" error
}

// File read errors
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to read file" error
}
```

### Minification Errors

```go
// Minification failures
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to minify bundle" error
}

// Unsupported file type
_, err := minify.AndVersionFile("file.txt", "output", "txt")
if err != nil {
    // Handle "unsupported file type" error
}
```

### File System Errors

```go
// Directory creation errors
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to create output directory" error
}

// File write errors
err := minify.ProcessBundles(config)
if err != nil {
    // Handle "failed to write minified file" error
}
```

## Performance Considerations

### Hashing Performance

- **xxhash**: Uses xxhash for fast content-based hashing
- **Base36 Encoding**: Compact hash representation (8 characters)
- **Single Pass**: Hash calculated once per bundle/file

### Memory Usage

- **Streaming**: Files are processed in memory for performance
- **Large Files**: Consider available memory for very large bundles
- **Concurrent Processing**: Safe for concurrent use with different bundles

### File I/O Optimization

- **Batch Processing**: Bundle processing minimizes individual file operations
- **Existence Checking**: Avoid regenerating unchanged files
- **Cleanup Strategy**: Regular cleanup prevents disk space issues

## Integration Examples

### Makefile Integration

```makefile
.PHONY: assets
assets:
	@echo "Building assets..."
	@go run scripts/build-assets.go

.PHONY: clean-assets
clean-assets:
	@echo "Cleaning old assets..."
	@go run scripts/clean-assets.go

.PHONY: build
build: assets
	@echo "Building application..."
	@go build -o bin/myapp ./cmd/myapp
```

### Docker Integration

```dockerfile
FROM golang:1.24-alpine AS builder

WORKDIR /app
COPY . .

# Install dependencies
RUN go mod download

# Build assets
RUN go run scripts/build-assets.go

# Build application
RUN go build -o bin/myapp ./cmd/myapp

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/

# Copy binary and assets
COPY --from=builder /app/bin/myapp .
COPY --from=builder /app/public ./public

CMD ["./myapp"]
```

### CI/CD Integration

```yaml
# GitHub Actions
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v3
      with:
        go-version: 1.24
    
    - name: Install dependencies
      run: go mod download
    
    - name: Build assets
      run: go run scripts/build-assets.go
    
    - name: Run tests
      run: go test -v ./...
    
    - name: Build application
      run: go build -o bin/myapp ./cmd/myapp
```

## Best Practices

1. **Bundle Organization**: Group related files into logical bundles
2. **File Ordering**: List files in dependency order within bundles
3. **Pattern Specificity**: Use specific glob patterns to avoid including unwanted files
4. **Regular Cleanup**: Implement regular cleanup of old bundle versions
5. **Error Handling**: Always check for errors when processing bundles
6. **Build Integration**: Integrate bundle processing into your build pipeline
7. **Performance**: Use `BundleExists` to avoid unnecessary regeneration
8. **Monitoring**: Log bundle processing for debugging and monitoring

## Thread Safety

The package functions are safe for concurrent use, but avoid processing the same bundle simultaneously from multiple goroutines to prevent file conflicts.

## Dependencies

- `github.com/cespare/xxhash/v2` - Fast hashing for content-based cache busting
- `github.com/tdewolff/minify/v2` - JavaScript and CSS minification
- Standard Go packages: `encoding/json`, `fmt`, `os`, `path/filepath`, `strconv`, `strings`

## License

BSD 3-Clause License - see [LICENSE](LICENSE) file for details.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and changes.

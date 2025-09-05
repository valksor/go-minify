# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of minify package
- JavaScript and CSS minification with content-based hashing
- Bundle configuration system via JSON files
- Automatic cache-busting with versioned filenames
- Glob pattern support for flexible file selection
- Bundle processing with `ProcessBundles()` function
- Hash generation with `GetBundleHash()` and `GetBundleFilename()`
- Bundle existence checking with `BundleExists()`
- Automatic cleanup of old bundle versions with `CleanOldBundles()`
- Single file minification with `AndVersionFile()`, `AndVersionCSS()`
- Support for both JavaScript and CSS minification
- Content-based hash generation using xxhash for performance
- Comprehensive error handling with detailed error messages
- Directory creation and file management utilities
- Configurable bundle system with JSON configuration
- Automatic old file cleanup to prevent disk space issues

### Technical Details
- Module name: `github.com/valksor/go-minify`
- Go version: 1.24+
- Package name: `minify`
- Dependencies: `github.com/cespare/xxhash/v2`, `github.com/tdewolff/minify/v2`
- Hash algorithm: xxhash (64-bit, base36 encoded, 8-character)
- Supported file types: JavaScript (.js) and CSS (.css)
- Bundle format: JSON configuration with name and file patterns
- Output format: `{bundle}.{hash}.min.{ext}`

### Features
- **Bundle System**: Define multiple bundles with different file patterns
- **Content Hashing**: Automatic hash generation based on minified content
- **Glob Patterns**: Support for complex file selection patterns
- **File Versioning**: Automatic versioning prevents cache issues
- **Cleanup Management**: Automatic removal of old bundle versions
- **Error Handling**: Comprehensive error handling with detailed messages
- **Performance**: Optimized minification and hashing for large files
- **Flexibility**: Support for both bundle and single-file workflows

### Documentation
- Complete API documentation with examples
- README with usage guide and configuration examples
- Comprehensive test suite demonstrating all functionality
- Integration examples for build systems and templates

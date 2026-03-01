# Posture v0.3.0 Release Notes

**Release Date:** March 2026

This release migrates the module path from the agentplexus organization to plexusone.

## Breaking Changes

### Module Path Changed

The Go module path has changed from `github.com/agentplexus/posture` to `github.com/plexusone/posture`.

**Before (v0.2.0):**
```go
import "github.com/agentplexus/posture/inspector"
```

**After (v0.3.0):**
```go
import "github.com/plexusone/posture/inspector"
```

## What's Changed

### Changed

- Module path migrated from `github.com/agentplexus/posture` to `github.com/plexusone/posture`
- All internal imports updated to use new module path
- Documentation URLs updated to plexusone organization
- CI workflows migrated to standard plexusone reusable workflows

### Fixed

- Use `os/user` instead of `USER` environment variable for fprintd on Linux

## Installation

### Pre-built Binaries

Download from the [Releases](https://github.com/plexusone/posture/releases) page:

- `posture` - CLI tool for manual security assessment
- `mcp-posture` - MCP server for AI assistants

### Build from Source

```bash
git clone https://github.com/plexusone/posture.git
cd posture

# Build CLI tool
go build -o posture ./cmd/posture/

# Build MCP server
go build -o mcp-posture ./cmd/mcp-posture/
```

## Migration Guide

1. Update your `go.mod` imports:
   - Change `github.com/agentplexus/posture` to `github.com/plexusone/posture`
2. Run `go mod tidy`
3. Update any import statements in your code

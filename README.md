# Posture

[![Build Status][build-status-svg]][build-status-url]
[![Lint Status][lint-status-svg]][lint-status-url]
[![Go Report Card][goreport-svg]][goreport-url]
[![Docs][docs-godoc-svg]][docs-godoc-url]
[![License][license-svg]][license-url]

A cross-platform security posture assessment tool with Model Context Protocol (MCP) server support. Posture provides unified security inspection across macOS, Windows, and Linux, enabling AI assistants to query hardware security modules, boot security, disk encryption, and biometric capabilities.

## Features

### Security Assessment
- **Platform Security Chip** - Secure Enclave (macOS) / TPM (Windows/Linux) detection and status
- **Secure Boot** - UEFI/Apple Secure Boot verification
- **Disk Encryption** - FileVault (macOS), BitLocker (Windows), LUKS (Linux)
- **Biometrics** - Touch ID, Face ID, Windows Hello, fprintd
- **Security Summary** - Unified security score with recommendations

### System Metrics
- **CPU Usage** - Overall and per-core monitoring
- **Memory Usage** - Total, used, free, available memory
- **Process List** - Running processes with resource usage

### Output Formats
- **JSON** (default) - Structured data for programmatic use
- **Table** - Rich ASCII tables with ANSI colors and UTF-8 icons

## Installation

### Pre-built Binary

Download the latest release for your platform from the [Releases](https://github.com/plexusone/posture/releases) page.

### Build from Source

Requires Go 1.23 or later.

```bash
git clone https://github.com/plexusone/posture.git
cd posture
go build -o posture ./cmd/posture/
```

## Usage

Posture can be used in three ways:

1. **CLI** - Command-line tool for interactive use
2. **MCP Server** - Model Context Protocol server for AI assistants
3. **Go Module** - Programmatic access in Go applications

## CLI Usage

```bash
# Show security summary with score
posture summary -f table

# Check platform security chip (Secure Enclave / TPM) status
posture security-chip -f table

# Check Secure Boot status
posture secureboot -f table

# Check disk encryption status
posture encryption -f table

# Check biometric capabilities
posture biometrics -f table

# System metrics
posture cpu -f table
posture memory -f table
posture processes -n 10 -f table
```

## MCP Server Usage

### Claude Desktop Configuration

Add to your Claude Desktop configuration file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "posture": {
      "command": "/path/to/posture",
      "args": ["serve"]
    }
  }
}
```

### MCP Tools

| Tool | Description |
|------|-------------|
| `get_platform_security_chip` | Secure Enclave (macOS) / TPM (Windows/Linux) status |
| `get_secure_boot_status` | UEFI Secure Boot verification |
| `get_encryption_status` | Disk encryption (FileVault/BitLocker/LUKS) |
| `get_biometric_capabilities` | Biometric authentication status |
| `get_security_summary` | Unified security posture with score |
| `get_cpu_usage` | CPU usage statistics |
| `get_memory` | Memory usage statistics |
| `list_processes` | Running process list |

## Go Module Usage

Import the `inspector` package for programmatic access to all security and system metrics.

### Installation

```bash
go get github.com/plexusone/posture
```

### Example: Security Summary

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"

	"github.com/plexusone/posture/inspector"
)

func main() {
	// Get unified security summary
	summary, err := inspector.GetSecuritySummary()
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Security Score: %d/100\n", summary.OverallScore)
	fmt.Printf("Status: %s\n", summary.OverallStatus)

	// Output as JSON
	data, _ := json.MarshalIndent(summary, "", "  ")
	fmt.Println(string(data))

	// Or use built-in table formatting
	fmt.Println(inspector.FormatSecuritySummaryTable(summary))
}
```

### Example: Individual Checks

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/plexusone/posture/inspector"
)

func main() {
	ctx := context.Background()

	// Platform Security Chip (Secure Enclave / TPM)
	if inspector.IsTPMSupported() {
		tpm, err := inspector.GetTPMStatus()
		if err == nil {
			fmt.Printf("Security Chip: %s (enabled: %v)\n", tpm.Type, tpm.Enabled)
		}
	}

	// Secure Boot
	if inspector.IsSecureBootSupported() {
		boot, err := inspector.GetSecureBootStatus()
		if err == nil {
			fmt.Printf("Secure Boot: %v (mode: %s)\n", boot.Enabled, boot.Mode)
		}
	}

	// Disk Encryption
	if inspector.IsEncryptionSupported() {
		enc, err := inspector.GetEncryptionStatus()
		if err == nil {
			fmt.Printf("Encryption: %s (status: %s)\n", enc.Type, enc.Status)
		}
	}

	// Biometrics
	if inspector.IsBiometricsSupported() {
		bio, err := inspector.GetBiometricCapabilities()
		if err == nil {
			fmt.Printf("Biometrics: %s (enrolled: %v)\n",
				bio.BiometryType, bio.TouchIDEnrolled || bio.FaceIDEnrolled)
		}
	}

	// System Metrics
	cpu, _ := inspector.GetCPUUsage(ctx)
	fmt.Printf("CPU Usage: %.1f%%\n", cpu.OverallPercent)

	mem, _ := inspector.GetMemory(ctx)
	fmt.Printf("Memory: %s / %s (%.1f%%)\n",
		inspector.FormatBytes(mem.Used),
		inspector.FormatBytes(mem.Total),
		mem.UsedPercent)
}
```

### Available Functions

| Function | Description |
|----------|-------------|
| `GetSecuritySummary()` | Unified security posture with score |
| `GetTPMStatus()` | Platform security chip status |
| `GetSecureBootStatus()` | Secure Boot configuration |
| `GetEncryptionStatus()` | Disk encryption status |
| `GetBiometricCapabilities()` | Biometric authentication status |
| `GetCPUUsage(ctx)` | CPU usage statistics |
| `GetMemory(ctx)` | Memory usage statistics |
| `ListProcesses(ctx, limit)` | Running process list |

Each function has a corresponding `IsXXXSupported()` function to check platform availability.

## Platform Support

| Feature | macOS | Windows | Linux |
|---------|-------|---------|-------|
| Platform Security Chip | ✅ Secure Enclave | ✅ TPM 1.2/2.0 | ✅ TPM 2.0 |
| Secure Boot | ✅ Apple Secure Boot | ✅ UEFI Secure Boot | ✅ UEFI Secure Boot |
| Disk Encryption | ✅ FileVault | ✅ BitLocker | ✅ LUKS/dm-crypt |
| Biometrics | ✅ Touch ID/Face ID | ✅ Windows Hello | ✅ fprintd/Howdy |
| CPU/Memory/Processes | ✅ | ✅ | ✅ |

## Example Output

### Security Summary (Table Format)

```
🛡️  Security Summary
────────────────────────────────────────────────────────────

Platform: 🍎 macOS

Security Score: 75/100
██████████████████████████████░░░░░░░░░░

Status: ✓ Good

Security Features:
┌──────────────────────────┬──────────────┬────────────────────┐
│ Feature                  │ Status       │ Details            │
├──────────────────────────┼──────────────┼────────────────────┤
│ 🛡️  Secure Enclave       │ ✓ Enabled    │ secure_enclave     │
│ 🔒 Secure Boot           │ ✓ Enabled    │ full               │
│ 🔒 FileVault             │ ✗ Disabled   │ disabled           │
│ 👆 Biometrics            │ ✓ Enabled    │ touch_id           │
└──────────────────────────┴──────────────┴────────────────────┘

⚠️  Recommendations:
──────────────────────────────────────────────────
  1. Enable FileVault to protect data at rest
```

### Security Summary (JSON Format)

```json
{
  "platform": "darwin",
  "overall_score": 75,
  "overall_status": "good",
  "tpm": {
    "present": true,
    "enabled": true,
    "type": "secure_enclave"
  },
  "secure_boot": {
    "enabled": true,
    "mode": "full"
  },
  "encryption": {
    "enabled": false,
    "type": "filevault",
    "status": "disabled"
  },
  "biometrics": {
    "available": true,
    "configured": true,
    "type": "touch_id"
  },
  "recommendations": [
    "Enable FileVault to protect data at rest"
  ]
}
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Claude Desktop                         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                    MCP Client                          │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ stdio (JSON-RPC)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       Posture                               │
│  ┌──────────────────┐  ┌──────────────────────────────────┐ │
│  │   MCP Server     │  │         Security Tools           │ │
│  │                  │  │  🛡️  get_platform_security_chip  │ │
│  │  - Tool registry │  │  🔒 get_secure_boot_status       │ │
│  │  - JSON-RPC      │  │  🔐 get_encryption_status        │ │
│  │  - stdio         │  │  👆 get_biometric_capabilities   │ │
│  │                  │  │  📊 get_security_summary         │ │
│  └──────────────────┘  └──────────────────────────────────┘ │
│                              │                              │
│  ┌───────────────────────────┴────────────────────────────┐ │
│  │                    Inspectors                          │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │ │
│  │  │ darwin  │ │ windows │ │  linux  │ │  common │       │ │
│  │  │ (cgo)   │ │ (WMI)   │ │ (sysfs) │ │(gopsutil│       │ │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Security Considerations

This tool is designed with security in mind:

- **Read-only operations** - No system modifications are possible
- **No secrets exposed** - Does not access keychain, passwords, or private keys
- **Non-invasive checks** - Only tests capability, never extracts keys
- **Process listing is informational** - Cannot terminate or modify processes

### What This Tool Does NOT Do

- Access or export any cryptographic keys
- Read keychain items or passwords
- Modify system settings
- Execute arbitrary commands
- Access file contents
- Make network requests

## Building for Different Platforms

```bash
# macOS (includes Secure Enclave)
GOOS=darwin GOARCH=arm64 go build -o posture-darwin-arm64 ./cmd/posture/
GOOS=darwin GOARCH=amd64 go build -o posture-darwin-amd64 ./cmd/posture/

# Linux (includes TPM, LUKS)
GOOS=linux GOARCH=amd64 go build -o posture-linux-amd64 ./cmd/posture/
GOOS=linux GOARCH=arm64 go build -o posture-linux-arm64 ./cmd/posture/

# Windows (includes TPM, BitLocker)
GOOS=windows GOARCH=amd64 go build -o posture-windows-amd64.exe ./cmd/posture/
```

Note: Cross-compiling for macOS from other platforms will not include Secure Enclave support due to cgo dependencies.

## Dependencies

- [modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk) - Official MCP Go SDK
- [shirou/gopsutil/v4](https://github.com/shirou/gopsutil) - Cross-platform system metrics
- [spf13/cobra](https://github.com/spf13/cobra) - CLI framework

## Related Projects

- [MCP Specification](https://modelcontextprotocol.io/)
- [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk)
- [Claude Desktop](https://claude.ai/download)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

 [build-status-svg]: https://github.com/plexusone/posture/actions/workflows/ci.yaml/badge.svg?branch=main
 [build-status-url]: https://github.com/plexusone/posture/actions/workflows/ci.yaml
 [lint-status-svg]: https://github.com/plexusone/posture/actions/workflows/lint.yaml/badge.svg?branch=main
 [lint-status-url]: https://github.com/plexusone/posture/actions/workflows/lint.yaml
 [goreport-svg]: https://goreportcard.com/badge/github.com/plexusone/posture
 [goreport-url]: https://goreportcard.com/report/github.com/plexusone/posture
 [docs-godoc-svg]: https://pkg.go.dev/badge/github.com/plexusone/posture
 [docs-godoc-url]: https://pkg.go.dev/github.com/plexusone/posture
 [license-svg]: https://img.shields.io/badge/license-MIT-blue.svg
 [license-url]: https://github.com/plexusone/posture/blob/master/LICENSE
 [used-by-svg]: https://sourcegraph.com/github.com/plexusone/posture/-/badge.svg
 [used-by-url]: https://sourcegraph.com/github.com/plexusone/posture?badge

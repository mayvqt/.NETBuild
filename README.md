# .NETBuild

A cross-platform build automation toolkit for .NET projects that simplifies the process of publishing .NET applications for multiple target platforms (Runtime Identifiers).

## Overview

This repository contains platform-specific build scripts that automate the compilation and publishing of .NET projects across different operating systems and architectures. The scripts handle common build scenarios including self-contained deployments, single-file executables, and trimmed applications.

## Files

### `build.bat` (Windows Build Script)
A Windows batch script that serves as the primary build automation tool for Windows environments (Command Prompt and PowerShell). This script provides comprehensive options for building and publishing .NET applications with support for:

- **Multi-platform targeting**: Builds for Windows (x64), Linux (x64/ARM64), and macOS (x64/ARM64)
- **Self-contained deployments**: Bundles the .NET runtime with your application
- **Single-file publishing**: Creates a single executable file (with caveats for reflection-heavy libraries)
- **Application trimming**: Reduces output size by removing unused code
- **Flexible project detection**: Automatically finds `.csproj` files in `./src` directory or current directory
- **Build artifact management**: Organizes output in structured `build/<project>/<config>/<rid>/publish` folders
- **Optional ZIP packaging**: Creates compressed archives of published applications

### `build.sh` (Linux/macOS Build Script)
A bash script that provides identical functionality to `build.bat` but for Unix-like systems (Linux and macOS). Features include:

- **Parallel builds**: Supports concurrent publishing for multiple target platforms
- **Cross-platform publishing**: Can build Windows, Linux, and macOS targets from any Unix system
- **Automatic cleanup**: Optional removal of previous build artifacts
- **CI/CD friendly**: Includes options for automated build environments

## Usage

### Windows (PowerShell/CMD)
```cmd
build.bat [options]
```

### Linux/macOS (Bash)
```bash
./build.sh [options]
```

## Command Line Options

Both scripts support the same set of options:

| Option | Description |
|--------|-------------|
| `--self-contained` | Produce self-contained publish (bundles .NET runtime) |
| `--single-file` | Produce a single-file executable |
| `--trim` | Enable publish trimming (smaller output; test carefully) |
| `--clean` | Remove previous build output before publishing |
| `--config <Debug\|Release>` | Build configuration (default: Release) |
| `--project <path>` | Path to .csproj or project directory |
| `--rids <comma-separated>` | Target runtime identifiers |
| `--zip` | Create ZIP archives of published applications |
| `--dry-run` | Show commands without executing them |
| `--parallel` | Build multiple targets concurrently (bash only) |
| `--ci` / `--no-pause` | CI mode (no interactive pauses) |

## Target Platforms

By default, both scripts build for the following Runtime Identifiers (RIDs):
- `win-x64` - Windows 64-bit
- `linux-x64` - Linux 64-bit 
- `linux-arm64` - Linux ARM64 (e.g., Raspberry Pi, Apple Silicon containers)
- `osx-x64` - macOS Intel 64-bit
- `osx-arm64` - macOS Apple Silicon (M1/M2)

## Project Structure

The scripts expect one of the following project structures:

```
your-project/
├── src/
│   └── YourApp.csproj    # Preferred: project in src subdirectory
└── build/                # Generated output directory
```

Or:

```
your-project/
├── YourApp.csproj        # Alternative: project in root
└── build/                # Generated output directory
```

## Output Structure

Published applications are organized in the following structure:

```
build/
└── <ProjectName>/
    └── <Configuration>/
        └── net8.0/
            ├── win-x64/
            │   └── publish/
            ├── linux-x64/
            │   └── publish/
            ├── linux-arm64/
            │   └── publish/
            ├── osx-x64/
            │   └── publish/
            └── osx-arm64/
                └── publish/
```

## Examples

### Basic Release Build
```bash
# Build for all default platforms in Release mode
./build.sh

# Windows equivalent
build.bat
```

### Self-Contained Single-File Application
```bash
# Create optimized, portable executables
./build.sh --self-contained --single-file --trim

# Windows equivalent  
build.bat --self-contained --single-file --trim
```

### Custom Project and Configuration
```bash
# Build specific project in Debug mode
./build.sh --project ./MyApp/MyApp.csproj --config Debug --clean

# Windows equivalent
build.bat --project .\MyApp\MyApp.csproj --config Debug --clean
```

### CI/CD Pipeline Usage
```bash
# Automated build for continuous integration
./build.sh --ci --clean --self-contained --zip

# Windows equivalent
build.bat --ci --clean --self-contained --zip
```

## Prerequisites

- [.NET 8.0 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or later
- For Linux/macOS: Bash shell
- For Windows: Command Prompt or PowerShell
- For ZIP functionality: PowerShell (Windows) or `zip` utility (Linux/macOS)

## Notes

- **Single-file publishing**: May not work with applications that heavily use reflection or load assemblies dynamically
- **Trimming**: Can significantly reduce application size but may break functionality in reflection-heavy applications
- **Self-contained**: Creates larger executables but eliminates the need for .NET runtime installation on target machines
- **Parallel builds**: Only available in the bash script (`build.sh`) due to shell limitations in batch files

## Use Cases

This toolkit is ideal for:
- Cross-platform .NET application distribution
- Creating portable executables for multiple operating systems  
- CI/CD pipelines requiring automated multi-target builds
- Development teams working across different platforms
- Applications requiring deployment without .NET runtime dependencies


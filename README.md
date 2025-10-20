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
A bash script that provides identical functionality to `build.bat` but for Unix-like systems (Linux and macOS). This script includes all the same features as the Windows version plus additional Unix-specific enhancements:

- **Feature parity**: All the same options and capabilities as `build.bat`
- **Parallel builds**: Supports concurrent publishing for multiple target platforms to speed up build times
- **Cross-platform publishing**: Can build Windows, Linux, and macOS targets from any Unix system
- **Robust error handling**: Proper exit codes and error propagation for CI/CD environments
- **Automatic cleanup**: Optional removal of previous build artifacts with improved error handling

## Getting Started

### Setup
1. **Install .NET SDK**: Download and install from [Microsoft .NET Downloads](https://dotnet.microsoft.com/download)
2. **Verify installation**:
   ```bash
   dotnet --version
   ```
3. **Clone or download** this repository to your machine
4. **Make scripts executable** (Linux/macOS only):
   ```bash
   chmod +x build.sh
   ```

### Basic Usage

#### Windows (Command Prompt or PowerShell)
```cmd
# Navigate to the build script directory
cd path\to\.NETBuild

# Run basic build (Release mode, all platforms)
build.bat

# Build with options
build.bat --self-contained --single-file --clean

# Build specific project
build.bat --project "C:\path\to\MyApp.csproj" --config Debug
```

#### Linux/macOS (Terminal/Bash)
```bash
# Navigate to the build script directory
cd /path/to/.NETBuild

# Run basic build (Release mode, all platforms)
./build.sh

# Build with options
./build.sh --self-contained --single-file --clean --parallel

# Build specific project
./build.sh --project "/path/to/MyApp.csproj" --config Debug
```

### Script Execution Permissions

#### Windows
No special permissions needed - batch files run directly in Command Prompt or PowerShell.

#### Linux/macOS
Make the script executable before first use:
```bash
chmod +x build.sh
```

## Command Line Options

Both scripts support identical options for maximum compatibility:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `--self-contained` | Flag | `false` | Produce self-contained publish (bundles .NET runtime) |
| `--single-file` | Flag | `false` | Produce a single-file executable |
| `--trim` | Flag | `false` | Enable publish trimming (smaller output; test carefully) |
| `--clean` | Flag | `true` | Remove previous build output before publishing |
| `--config <value>` | String | `Release` | Build configuration: `Debug` or `Release` |
| `--project <path>` | String | `./src` or `.` | Path to `.csproj` file or project directory |
| `--rids <list>` | String | See below | Comma-separated runtime identifiers |
| `--zip` | Flag | `false` | Create ZIP archives of published applications |
| `--dry-run` | Flag | `false` | Show commands without executing them |
| `--parallel` | Flag | `false` | Build multiple targets concurrently (bash only) |
| `--ci` / `--no-pause` | Flag | `false` | CI mode (no interactive pauses) |

### Default Runtime Identifiers (RIDs)
By default, both scripts build for these platforms:
- `win-x64` - Windows 64-bit (Intel/AMD)
- `linux-x64` - Linux 64-bit (Intel/AMD)
- `linux-arm64` - Linux ARM64 (Raspberry Pi 4+, AWS Graviton, etc.)
- `osx-x64` - macOS Intel 64-bit
- `osx-arm64` - macOS Apple Silicon (M1/M2/M3)

### Custom Runtime Identifiers
Override with `--rids` option:
```bash
# Build only for Windows and Linux x64
./build.sh --rids "win-x64,linux-x64"

# Build for specific ARM platforms
./build.sh --rids "linux-arm,linux-arm64,osx-arm64"

# Single platform build
./build.sh --rids "win-x64"
```

## Target Platforms

By default, both scripts build for the following Runtime Identifiers (RIDs):
- `win-x64` - Windows 64-bit
- `linux-x64` - Linux 64-bit 
- `linux-arm64` - Linux ARM64 (e.g., Raspberry Pi, Apple Silicon containers)
- `osx-x64` - macOS Intel 64-bit
- `osx-arm64` - macOS Apple Silicon (M1/M2)

## Project Structure

### Expected Directory Layout
The scripts automatically detect your project structure and work with these common layouts:

#### Option 1: Project in `src` subdirectory (Recommended)
```
your-solution/
├── .NETBuild/
│   ├── build.bat         # Place build scripts here
│   ├── build.sh
│   └── README.md
├── src/
│   ├── MyApp/
│   │   ├── MyApp.csproj  # Your .NET project
│   │   ├── Program.cs
│   │   └── ...
│   └── MyLibrary/
│       ├── MyLibrary.csproj
│       └── ...
└── build/                # Generated output (auto-created)
    └── MyApp/
        └── Release/
            └── net8.0/
                ├── win-x64/
                ├── linux-x64/
                └── ...
```

#### Option 2: Project in root directory
```
your-project/
├── MyApp.csproj          # Project file in root
├── Program.cs
├── build.bat             # Build scripts alongside project
├── build.sh
└── build/                # Generated output (auto-created)
```

#### Option 3: Custom project location
```
custom-structure/
├── tools/
│   ├── build.bat         # Build scripts in tools folder
│   └── build.sh
├── apps/
│   └── MyApp/
│       ├── MyApp.csproj  # Specify with --project parameter
│       └── ...
└── output/               # Use custom output with scripts
```

### Project Detection Logic
1. **Explicit path**: If `--project` is specified, use that exact path
2. **Auto-detection**: Look for `.csproj` files in this order:
   - `./src/` directory (if exists)
   - Current working directory
   - First `.csproj` found in the target directory

### Supported Project Types
- **.NET 8.0+** Console applications
- **.NET 8.0+** Web applications (ASP.NET Core)
- **.NET 8.0+** Worker services
- **.NET 8.0+** Class libraries (when used as executable)
- **Any .NET project** with a valid `.csproj` file

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

### 1. Basic Release Build
Build for all default platforms in Release configuration:

**Linux/macOS:**
```bash
./build.sh
```

**Windows:**
```cmd
build.bat
```

### 2. Self-Contained Single-File Application
Create optimized, portable executables that don't require .NET runtime:

**Linux/macOS:**
```bash
./build.sh --self-contained --single-file --trim
```

**Windows:**
```cmd
build.bat --self-contained --single-file --trim
```

### 3. Fast Parallel Development Build
Quick Debug build with parallel processing (Linux/macOS only):

```bash
./build.sh --config Debug --parallel --clean
```

### 4. Custom Project and Platforms
Build specific project for selected platforms:

**Linux/macOS:**
```bash
./build.sh --project ./src/MyApp/MyApp.csproj --rids "win-x64,linux-x64" --zip
```

**Windows:**
```cmd
build.bat --project .\src\MyApp\MyApp.csproj --rids "win-x64,linux-x64" --zip
```

### 5. CI/CD Pipeline Usage
Automated build for continuous integration:

**Linux/macOS:**
```bash
./build.sh --ci --clean --self-contained --zip --parallel
```

**Windows:**
```cmd
build.bat --ci --clean --self-contained --zip
```

### 6. Dry Run (Test Configuration)
Preview commands without executing:

**Linux/macOS:**
```bash
./build.sh --dry-run --self-contained --single-file
```

**Windows:**
```cmd
build.bat --dry-run --self-contained --single-file
```

### 7. Production Release Build
Complete production build with all optimizations:

**Linux/macOS:**
```bash
./build.sh --self-contained --single-file --trim --zip --clean --ci
```

**Windows:**
```cmd
build.bat --self-contained --single-file --trim --zip --clean --ci
```

## Prerequisites

### All Platforms
- [.NET 8.0 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) or later
- A .NET project with a `.csproj` file

### Windows (`build.bat`)
- **Windows 7/8/10/11** or **Windows Server 2012+**
- **Command Prompt** or **PowerShell** (any version)
- **PowerShell** (for ZIP functionality) - pre-installed on Windows 7+ and Windows Server 2012+

### Linux (`build.sh`) 
- **Bash 4.0+** (pre-installed on most distributions)
- **zip utility** - install if missing:
  ```bash
  # Ubuntu/Debian
  sudo apt-get install zip
  
  # CentOS/RHEL/Fedora
  sudo yum install zip    # or: sudo dnf install zip
  
  # Alpine Linux
  sudo apk add zip
  ```

### macOS (`build.sh`)
- **macOS 10.10+** (Yosemite or later)
- **Bash 4.0+** - macOS includes bash, but you may want to update:
  ```bash
  # Install newer bash via Homebrew (optional)
  brew install bash
  ```
- **zip utility** (pre-installed)
- **Command Line Tools for Xcode**:
  ```bash
  xcode-select --install
  ```

## Troubleshooting

### Common Issues

#### 1. "dotnet command not found"
**Solution**: Install .NET SDK and ensure it's in your PATH:
```bash
# Verify installation
dotnet --version

# Add to PATH if needed (Linux/macOS)
export PATH="$PATH:/usr/share/dotnet"

# Windows: Reinstall .NET SDK or restart terminal
```

#### 2. Permission denied (Linux/macOS)
**Solution**: Make script executable:
```bash
chmod +x build.sh
```

#### 3. "No .csproj found"
**Solution**: Specify project explicitly:
```bash
./build.sh --project "/full/path/to/YourApp.csproj"
```

#### 4. Build fails for specific platforms
**Solution**: Check platform compatibility and exclude problematic RIDs:
```bash
./build.sh --rids "win-x64,linux-x64"  # Skip ARM if not supported
```

#### 5. ZIP creation fails
- **Linux**: Install zip utility: `sudo apt-get install zip`
- **Windows**: Ensure PowerShell is available

### Performance Tips

#### Faster Builds
- Use `--parallel` on Linux/macOS for concurrent builds
- Use `--config Debug` for faster compilation during development
- Exclude unnecessary platforms with `--rids`

#### Smaller Outputs
- Use `--trim` to remove unused code (test thoroughly)
- Use `--single-file` to create single executable
- Avoid `--self-contained` unless runtime deployment is required

## Important Notes

### Build Options Impact
- **Single-file publishing**: May not work with applications that heavily use reflection or load assemblies dynamically at runtime
- **Trimming**: Can significantly reduce application size (50-90% smaller) but may break functionality in reflection-heavy applications or libraries that use dynamic loading
- **Self-contained**: Creates larger executables (80-150MB) but eliminates the need for .NET runtime installation on target machines
- **Framework-dependent**: Smaller executables (1-50MB) but requires .NET runtime on target machines

### Platform Considerations
- **Parallel builds**: Available in bash script (`build.sh`) for faster multi-target builds; Windows batch files run sequentially due to shell limitations
- **Cross-compilation**: Both scripts can build for any supported platform from any host OS (Windows can build Linux binaries, etc.)
- **Feature parity**: Both scripts support identical command-line options and functionality

### CI/CD Integration
- Use `--ci` flag to disable interactive prompts
- Use `--dry-run` to validate configuration before actual builds
- Both scripts return appropriate exit codes for automation
- Output is organized consistently across platforms for easy artifact collection

## Use Cases

This toolkit is ideal for:

### Development Scenarios
- **Cross-platform .NET application distribution** - Build once, deploy everywhere
- **Multi-target development** - Support Windows, Linux, and macOS from single codebase
- **Portable application deployment** - Create self-contained executables that run without installing .NET
- **Development team collaboration** - Consistent build process across Windows, Linux, and macOS developers

### Production Deployments  
- **Container deployments** - Build optimized, trimmed applications for Docker containers
- **Edge computing** - Deploy to ARM64 devices (Raspberry Pi, AWS Graviton, Apple Silicon)
- **Desktop application distribution** - Create single-file executables for end users
- **Server deployments** - Framework-dependent or self-contained server applications

### CI/CD Pipelines
- **Automated multi-target builds** in GitHub Actions, Azure DevOps, Jenkins, etc.
- **Release artifact generation** with automatic ZIP packaging
- **Build matrix testing** across multiple platforms and configurations
- **Deployment automation** with consistent output structure

### Specific Examples
- **Microservices**: Build lightweight, trimmed services for Kubernetes
- **CLI tools**: Create portable command-line utilities for system administrators  
- **Desktop apps**: Distribute .NET applications without requiring users to install .NET
- **IoT applications**: Deploy to ARM-based devices and embedded systems
- **Web applications**: Build ASP.NET Core apps for various hosting environments

## License

This project is provided as-is for educational and development purposes. Feel free to modify and distribute according to your needs.

## Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests to improve the build scripts and documentation.


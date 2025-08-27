# Bats-core Installation Guide

This guide covers all available methods to install Bats-core (Bash Automated Testing System) on various operating systems and environments.

## Table of Contents

- [Linux: Distribution Package Manager](#linux-distribution-package-manager)
- [macOS: Homebrew](#macos-homebrew)
- [Any OS: npm](#any-os-npm)
- [Any OS: Installing from Source](#any-os-installing-from-source)
- [Windows: Git Bash](#windows-git-bash)
- [Docker](#docker)
- [Verification](#verification)

## Linux: Distribution Package Manager

The following Linux distributions provide Bats via their package manager:

### Arch Linux
```bash
sudo pacman -S bats
```

### Alpine Linux
```bash
sudo apk add bats
```

### Debian/Ubuntu Linux
```bash
sudo apt-get update
sudo apt-get install bats
```

### Fedora Linux
```bash
sudo dnf install bats
```

### Gentoo Linux
```bash
sudo emerge dev-util/bats
```

### OpenSUSE Linux
```bash
sudo zypper install bats
```

**Note**: Bats versions pre-1.0 are from the original project. For the latest features and bug fixes, consider using one of the other installation methods below to get the most recent Bats release.

## macOS: Homebrew

On macOS, install [Homebrew](https://brew.sh/) if you haven't already, then run:

```bash
brew install bats-core
```

## Any OS: npm

You can install the [Bats npm package](https://www.npmjs.com/package/bats) via npm:

### Global Installation
```bash
npm install -g bats
```

### Project-local Installation
To install into your project and save it as one of the "devDependencies" in your `package.json`:

```bash
npm install --save-dev bats
```

After local installation, you can run tests using:
```bash
npx bats test/
```

## Any OS: Installing from Source

### Prerequisites
- Git
- Bash 3.2 or higher
- Standard Unix utilities (install, mkdir, etc.)

### Installation Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/bats-core/bats-core.git
   cd bats-core
   ```

2. **Install to a system directory (requires sudo):**
   ```bash
   sudo ./install.sh /usr/local
   ```

3. **Install to a custom directory:**
   ```bash
   ./install.sh /path/to/your/prefix
   ```

4. **Install with custom library directory:**
   ```bash
   ./install.sh /usr/local lib64
   ```

### Alternative: Add to PATH

Instead of installing system-wide, you can add the Bats `bin` directory to your `$PATH`:

```bash
git clone https://github.com/bats-core/bats-core.git
export PATH="$PWD/bats-core/bin:$PATH"
```

Add the export line to your shell's configuration file (`.bashrc`, `.zshrc`, etc.) to make it permanent.

## Windows: Git Bash

### Prerequisites
- Git for Windows (includes Git Bash)

### Installation Steps

1. **Open Git Bash**

2. **Clone and install to home directory:**
   ```bash
   git clone https://github.com/bats-core/bats-core.git
   cd bats-core
   ./install.sh $HOME
   ```

This will place the `bats` executable in `$HOME/bin`, which should already be in your `$PATH`.

## Docker

### Using the Official Image

There is an official Bats image available on Docker Hub:

```bash
# Check version
docker run -it bats/bats:latest --version

# Run tests from current directory
docker run -it -v "${PWD}:/code" bats/bats:latest test

# Run Bats' internal test suite
docker run -it bats/bats:latest /opt/bats/test
```

### Building a Custom Docker Image

1. **Clone the repository:**
   ```bash
   git clone https://github.com/bats-core/bats-core.git
   cd bats-core
   ```

2. **Build the image:**
   ```bash
   docker build --tag bats/bats:latest .
   ```

3. **Run tests with the custom image:**
   ```bash
   docker run -it -v "${PWD}:/code" bats/bats:latest test
   ```

### Docker Usage Examples

- **Run tests from a specific directory:**
  ```bash
  docker run -it -v "${PWD}/tests:/code" bats/bats:latest /code
  ```

- **Use as base image in Dockerfile:**
  ```dockerfile
  FROM bats/bats:latest
  # Add your additional tools here
  RUN apk add --no-cache curl openssl
  ```

## Verification

After installation, verify that Bats is working correctly:

### Check Version
```bash
bats --version
```

### Run Help
```bash
bats --help
```

### Create and Run a Simple Test

1. **Create a test file (`test.bats`):**
   ```bash
   #!/usr/bin/env bats
   
   @test "addition using bc" {
     result="$(echo 2+2 | bc)"
     [ "$result" -eq 4 ]
   }
   ```

2. **Make it executable:**
   ```bash
   chmod +x test.bats
   ```

3. **Run the test:**
   ```bash
   bats test.bats
   ```

You should see output like:
```
 ✓ addition using bc

1 test, 0 failures
```

## Troubleshooting

### Common Issues

**Permission denied during installation:**
- Use `sudo` when installing to system directories
- Or install to a user directory like `$HOME`

**Command not found after installation:**
- Check that the installation directory is in your `$PATH`
- For source installations, the default is `/usr/local/bin`

**Bash version too old:**
- Bats requires Bash 3.2 or higher
- Update your Bash installation or use a different installation method

**Tests not running in Docker:**
- Ensure you're mounting the test directory correctly
- Use absolute paths when mounting volumes
- Check that test files have the `.bats` extension

### Getting Help

- [GitHub Issues](https://github.com/bats-core/bats-core/issues)
- [Gitter Chat](https://gitter.im/bats-core/bats-core)
- [Documentation](https://bats-core.readthedocs.io)

## Uninstallation

If you installed from source, you can uninstall using:

```bash
# From the bats-core source directory
sudo ./uninstall.sh /usr/local
```

For package manager installations, use the appropriate removal command:
- **Homebrew:** `brew uninstall bats-core`
- **npm:** `npm uninstall -g bats` or `npm uninstall bats`
- **apt:** `sudo apt-get remove bats`
- **dnf:** `sudo dnf remove bats`
- **pacman:** `sudo pacman -R bats`

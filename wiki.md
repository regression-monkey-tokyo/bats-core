# Bats-core Usage Guide

Bats is a TAP-compliant testing framework for Bash 3.2 or above. It provides a simple way to verify that the UNIX programs you write behave as expected.

## Table of Contents

- [Basic Test Structure](#basic-test-structure)
- [Command Line Usage](#command-line-usage)
- [Writing Tests](#writing-tests)
- [Setup and Teardown](#setup-and-teardown)
- [Helper Libraries](#helper-libraries)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Basic Test Structure

A Bats test file is a Bash script with special syntax for defining test cases. Each test case is a function with a description.

### Simple Example

```bash
#!/usr/bin/env bats

@test "addition using bc" {
  result="$(echo 2+2 | bc)"
  [ "$result" -eq 4 ]
}

@test "addition using dc" {
  result="$(echo 2 2+p | dc)"
  [ "$result" -eq 4 ]
}
```

### Test File Requirements

- Files must have the `.bats` extension
- Start with `#!/usr/bin/env bats` shebang
- Each test uses the `@test "description" { ... }` syntax
- Tests consist of standard shell commands
- If every command exits with status 0, the test passes

## Command Line Usage

### Basic Syntax

```bash
bats [OPTIONS] <test-file-or-directory>...
```

### Common Options

```bash
# Run a single test file
bats test.bats

# Run all tests in a directory
bats test/

# Run tests recursively
bats -r test/

# Show help
bats --help

# Show version
bats --version
```

### Output Formats

```bash
# Default pretty format (when connected to terminal)
bats test.bats

# TAP format
bats --formatter tap test.bats

# JUnit XML format
bats --formatter junit test.bats

# Custom formatter
bats --formatter /path/to/custom-formatter test.bats
```

### Parallel Execution

```bash
# Run tests in parallel (requires GNU parallel)
bats --jobs 4 test/

# Parallel within files only
bats --jobs 4 --no-parallelize-across-files test/

# Sequential files, parallel within files
bats --jobs 4 --no-parallelize-within-files test/
```

### Report Generation

```bash
# Generate JUnit report
bats --report-formatter junit --output /tmp test.bats

# This creates /tmp/report.xml
```

## Writing Tests

### Using the `run` Command

The `run` command executes a command and captures its output and exit status:

```bash
@test "check script output" {
  run ./my-script.sh
  [ "$status" -eq 0 ]
  [ "$output" = "Hello, World!" ]
}
```

### Variables Available After `run`

- `$status`: Exit status of the command
- `$output`: Combined stdout and stderr
- `$lines`: Array of output lines

### Testing Exit Status

```bash
@test "script should fail with invalid input" {
  run ./my-script.sh --invalid-option
  [ "$status" -eq 1 ]
}
```

### Testing Output

```bash
@test "script produces expected output" {
  run echo "Hello, World!"
  [ "$output" = "Hello, World!" ]
  [ "${lines[0]}" = "Hello, World!" ]
}
```

### Skipping Tests

```bash
@test "this test requires special setup" {
  if [ ! -f /special/file ]; then
    skip "Special file not available"
  fi
  
  # Test code here
}
```

## Setup and Teardown

### Per-Test Setup and Teardown

```bash
setup() {
  # Run before each test
  export TEST_DIR=$(mktemp -d)
  cd "$TEST_DIR"
}

teardown() {
  # Run after each test (even if test fails)
  rm -rf "$TEST_DIR"
}

@test "creates temporary file" {
  touch test-file
  [ -f test-file ]
}
```

### Per-File Setup and Teardown

```bash
setup_file() {
  # Run once before all tests in the file
  export SHARED_RESOURCE="initialized"
}

teardown_file() {
  # Run once after all tests in the file
  cleanup_shared_resources
}
```

### Common Setup Pattern

```bash
setup() {
  # Get the directory containing this test file
  DIR="$( cd "$( dirname "$BATS_TEST_FILENAME" )" >/dev/null 2>&1 && pwd )"
  
  # Make project executables available
  PATH="$DIR/../src:$PATH"
}
```

## Helper Libraries

### Installing Helper Libraries

```bash
# Add as git submodules
git submodule add https://github.com/bats-core/bats-support.git test/test_helper/bats-support
git submodule add https://github.com/bats-core/bats-assert.git test/test_helper/bats-assert
git submodule add https://github.com/bats-core/bats-file.git test/test_helper/bats-file
```

### Using bats-support and bats-assert

```bash
setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'
}

@test "check output with assertions" {
  run echo "Hello, World!"
  assert_success
  assert_output "Hello, World!"
}

@test "check partial output" {
  run echo "Hello, World!"
  assert_output --partial "Hello"
}

@test "check error output" {
  run bash -c 'echo "error" >&2; exit 1'
  assert_failure
  assert_output --stderr "error"
}
```

### Common Assertions

```bash
# Status assertions
assert_success
assert_failure
assert_equal "$status" 2

# Output assertions
assert_output "expected output"
assert_output --partial "substring"
assert_output --regexp "pattern"
assert_line "expected line"
assert_line --index 0 "first line"

# Negated assertions
refute_output "unexpected output"
refute_line "unexpected line"
```

### Using bats-file

```bash
setup() {
  load 'test_helper/bats-file/load'
}

@test "file operations" {
  touch test-file
  assert_file_exists test-file
  assert_file_not_empty test-file
  
  mkdir test-dir
  assert_dir_exists test-dir
}
```

## Advanced Features

### Conditional Test Execution

```bash
@test "test requiring docker" {
  if ! command -v docker >/dev/null; then
    skip "Docker not available"
  fi
  
  run docker --version
  assert_success
}
```

### Testing Functions

```bash
# Source the script to test its functions
setup() {
  source "$BATS_TEST_DIRNAME/../src/my-functions.sh"
}

@test "my_function returns correct value" {
  run my_function "input"
  assert_success
  assert_output "expected output"
}
```

### Multi-file Test Organization

Create a common setup file:

```bash
# test/test_helper/common-setup.bash
_common_setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'
  
  # Project root directory
  PROJECT_ROOT="$( cd "$( dirname "$BATS_TEST_FILENAME" )/.." >/dev/null 2>&1 && pwd )"
  PATH="$PROJECT_ROOT/src:$PATH"
}
```

Use in test files:

```bash
setup() {
  load 'test_helper/common-setup'
  _common_setup
}
```

### Environment Variables

```bash
@test "test with environment variables" {
  export MY_VAR="test value"
  run my-script.sh
  assert_success
}
```

### Testing with Different Inputs

```bash
@test "script handles various inputs" {
  inputs=("input1" "input2" "input3")
  
  for input in "${inputs[@]}"; do
    run my-script.sh "$input"
    assert_success
  done
}
```

## Best Practices

### Test Organization

1. **Use descriptive test names:**
   ```bash
   @test "user can login with valid credentials"
   @test "user receives error message with invalid credentials"
   ```

2. **Group related tests in files:**
   ```
   test/
   ├── auth.bats          # Authentication tests
   ├── database.bats      # Database tests
   └── api.bats          # API tests
   ```

3. **Use setup/teardown for common operations:**
   ```bash
   setup() {
     create_test_environment
   }
   
   teardown() {
     cleanup_test_environment
   }
   ```

### Writing Reliable Tests

1. **Make tests independent:**
   - Each test should be able to run in isolation
   - Don't rely on test execution order
   - Clean up after each test

2. **Use absolute paths or proper PATH setup:**
   ```bash
   setup() {
     DIR="$( cd "$( dirname "$BATS_TEST_FILENAME" )" && pwd )"
     PATH="$DIR/../bin:$PATH"
   }
   ```

3. **Test both success and failure cases:**
   ```bash
   @test "command succeeds with valid input" {
     run my-command valid-input
     assert_success
   }
   
   @test "command fails with invalid input" {
     run my-command invalid-input
     assert_failure
   }
   ```

### Performance Tips

1. **Use parallel execution for large test suites:**
   ```bash
   bats --jobs $(nproc) test/
   ```

2. **Skip expensive tests when appropriate:**
   ```bash
   @test "integration test" {
     if [ "$SKIP_INTEGRATION" = "true" ]; then
       skip "Integration tests disabled"
     fi
     # Test code
   }
   ```

3. **Use setup_file for expensive one-time setup:**
   ```bash
   setup_file() {
     # Start test database, compile code, etc.
   }
   ```

## Troubleshooting

### Common Issues

**Tests fail with "command not found":**
- Check your PATH setup in the setup function
- Ensure executables are in the correct location
- Use absolute paths if necessary

**Tests pass individually but fail when run together:**
- Tests may have dependencies on each other
- Use proper setup/teardown to ensure test isolation
- Check for shared state or files

**Parallel tests fail:**
- Tests may be writing to the same files
- Use unique temporary directories for each test
- Disable parallelization for problematic files

### Debugging Tests

1. **Add debug output:**
   ```bash
   @test "debug example" {
     echo "Debug: variable value is $MY_VAR" >&3
     run my-command
     assert_success
   }
   ```

2. **Use `--verbose-run` flag:**
   ```bash
   bats --verbose-run test.bats
   ```

3. **Check intermediate values:**
   ```bash
   @test "check intermediate steps" {
     run step1
     echo "Step 1 output: $output" >&3
     assert_success
     
     run step2 "$output"
     echo "Step 2 output: $output" >&3
     assert_success
   }
   ```

### Getting Help

- **Documentation:** https://bats-core.readthedocs.io
- **GitHub Issues:** https://github.com/bats-core/bats-core/issues
- **Gitter Chat:** https://gitter.im/bats-core/bats-core
- **Wiki:** https://github.com/bats-core/bats-core/wiki

### Example Test Suite Structure

```
project/
├── src/
│   ├── main.sh
│   └── utils.sh
├── test/
│   ├── bats/                    # Bats submodule
│   ├── test_helper/
│   │   ├── bats-support/        # Helper library
│   │   ├── bats-assert/         # Assertion library
│   │   └── common-setup.bash    # Shared setup
│   ├── main.bats               # Main functionality tests
│   ├── utils.bats              # Utility function tests
│   └── integration.bats        # Integration tests
└── README.md
```

This structure provides a solid foundation for testing Bash projects with Bats.

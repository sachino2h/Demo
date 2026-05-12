# Advanced Shell Scripting Training Guide

A comprehensive guide for mastering advanced shell scripting techniques to write production-grade `.sh` files.

---

## Table of Contents

1. [Fundamentals Review](#fundamentals-review)
2. [Advanced Variables & Parameter Expansion](#advanced-variables--parameter-expansion)
3. [Functions & Scope](#functions--scope)
4. [Control Flow Structures](#control-flow-structures)
5. [Text Processing & Regular Expressions](#text-processing--regular-expressions)
6. [File Operations & I/O Redirection](#file-operations--io-redirection)
7. [Process Management](#process-management)
8. [Error Handling & Debugging](#error-handling--debugging)
9. [Advanced Scripting Patterns](#advanced-scripting-patterns)
10. [Performance Optimization](#performance-optimization)
11. [Security Best Practices](#security-best-practices)
12. [Real-World Examples](#real-world-examples)

---

## 1. Fundamentals Review

### Shebang & Script Initialization

```bash
#!/bin/bash
# Or use this for better portability:
#!/usr/bin/env bash

# Always set these options for robustness
set -euo pipefail  # exit on error, undefined vars, pipe failures
IFS=$'\n\t'        # safer Internal Field Separator
```

**Explanation:**
- `-e`: Exit immediately if any command exits with non-zero status
- `-u`: Treat unset variables as error
- `-o pipefail`: Return exit status of first pipe command that fails
- `IFS`: Prevents word splitting on whitespace

### Script Header Best Practices

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# Script metadata
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_VERSION="1.0.0"

# Global configuration
declare -r LOG_FILE="${LOG_FILE:-/var/log/myscript.log}"
declare -r DEBUG="${DEBUG:-false}"

# Utility functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $*" >&2
}

main() {
    log "Starting $SCRIPT_NAME v$SCRIPT_VERSION"
    # Main logic here
}

main "$@"
```

---

## 2. Advanced Variables & Parameter Expansion

### Variable Declaration & Types

```bash
# Read-only variables
readonly PI=3.14159
declare -r CONSTANT="immutable"

# Integer variables (arithmetic)
declare -i count=0
declare -i result=$((count + 5))

# Array (indexed)
declare -a fruits=("apple" "banana" "orange")
fruits+=("grape")  # Append to array

# Associative array (hash/map)
declare -A config=(
    [host]="localhost"
    [port]="8080"
    [debug]="true"
)
config[timeout]="30"

# Export variables to subshells
export PATH_TO_DATA="/usr/local/data"

# Type info
declare -p config          # Print declaration
declare -p | grep "fruits" # List specific
```

### Parameter Expansion Operators

```bash
var="hello world"

# Default values
echo "${var:-default}"       # Use default if var is unset/empty
echo "${var:=default}"       # Assign default if unset/empty
echo "${var:+replaced}"      # Use alternate if var is set

# String manipulation
echo "${var:0:5}"            # Substring (offset:length)
echo "${var#hello }"         # Remove prefix (greedy)
echo "${var##hello }"        # Remove prefix (non-greedy)
echo "${var% world}"         # Remove suffix
echo "${var%% world}"        # Remove suffix (greedy)
echo "${var/world/globe}"    # Replace first occurrence
echo "${var//world/globe}"   # Replace all occurrences
echo "${var^}"               # Convert to uppercase (first char)
echo "${var^^}"              # Convert to uppercase (all)
echo "${var,}"               # Convert to lowercase (first char)
echo "${var,,}"              # Convert to lowercase (all)
echo "${#var}"               # String length

# Array operations
array=(a b c d e)
echo "${array[@]}"           # All elements
echo "${array[*]}"           # All elements (as string)
echo "${#array[@]}"          # Array length
echo "${array[@]:1:2}"       # Slice from index 1, length 2
echo "${!array[@]}"          # All indices
echo "${array[@]#a}"         # Remove prefix from all

# Indirect variable expansion
var_name="target"
target="value"
echo "${!var_name}"          # Outputs: "value"
```

### Advanced Pattern Matching

```bash
# Glob patterns
for file in *.{txt,md,sh}; do
    [[ -e "$file" ]] && echo "Found: $file"
done

# Case-insensitive globbing (shopt)
shopt -s nocaseglob

# Extended glob patterns (shopt -s extglob)
shopt -s extglob

# ?(pattern) - Zero or one occurrence
if [[ "color" == @(colour|color) ]]; then
    echo "Matched!"
fi

# *(pattern) - Zero or more occurrences
# +(pattern) - One or more occurrences
# !(pattern) - Anything except pattern
```

---

## 3. Functions & Scope

### Function Definition & Parameters

```bash
# Basic function
my_function() {
    local var="local scope"
    echo "$var"
}

# Function with parameters
greet() {
    local name="${1:?Name is required}"
    local greeting="${2:-Hello}"
    echo "$greeting, $name!"
}

# Function with return values
calculate() {
    local -i num1=$1 num2=$2
    local -i result=$((num1 + num2))
    
    # Return via stdout (preferred)
    echo "$result"
    
    # Or return exit status (0-255)
    return 0
}

# Capture return value
sum=$(calculate 5 10)
echo "Sum: $sum"

# Variable scope
global_var="I'm global"

scoped_function() {
    local global_var="I'm local"  # Shadows global
    echo "$global_var"             # Outputs: I'm local
}

scoped_function
echo "$global_var"                 # Outputs: I'm global

# Function with variable arguments
process_files() {
    local -i count=0
    
    for file in "$@"; do
        echo "Processing: $file"
        ((count++))
    done
    
    echo "Processed $count files"
}

process_files file1.txt file2.txt file3.txt

# Recursive functions
fibonacci() {
    local -i n=$1
    
    if (( n <= 1 )); then
        echo "$n"
    else
        local -i prev=$(fibonacci $((n - 1)))
        local -i prevprev=$(fibonacci $((n - 2)))
        echo $((prev + prevprev))
    fi
}
```

### Function Best Practices

```bash
# Template for robust functions
# Usage: get_user_input <prompt> <variable_name>
get_user_input() {
    local prompt="$1"
    local var_ref="$2"
    
    if [[ -z "$var_ref" ]]; then
        error "Variable name required"
        return 1
    fi
    
    read -rp "$prompt" value
    
    # Assign to variable indirectly
    eval "$var_ref='$value'"
    
    return 0
}

# Function that sets global variables
configure_app() {
    CONFIG_HOST="localhost"
    CONFIG_PORT=8080
    CONFIG_DEBUG=true
    
    return 0
}

# Declare and use
configure_app
echo "Connecting to $CONFIG_HOST:$CONFIG_PORT"
```

---

## 4. Control Flow Structures

### Conditional Statements

```bash
# Test conditions ([ ] is equivalent to test)
if [[ -f "$file" ]]; then
    echo "File exists"
elif [[ -d "$file" ]]; then
    echo "Directory exists"
else
    echo "Neither file nor directory"
fi

# [[ ]] is preferred (bash-specific, more features)
if [[ $var == pattern* ]] && [[ $var != *bad* ]]; then
    echo "Matches pattern"
fi

# Ternary-like operator
result=$( [[ $age -ge 18 ]] && echo "adult" || echo "minor" )

# Pattern matching in [[ ]]
string="hello123world"

if [[ $string =~ ^[a-z]+[0-9]+[a-z]+$ ]]; then
    echo "Matches pattern"
fi

# Using ! for negation
if [[ ! -e "$file" ]]; then
    echo "File does not exist"
fi
```

### File & Directory Tests

```bash
# File type tests
[[ -f "$file" ]]      # Regular file
[[ -d "$dir" ]]       # Directory
[[ -L "$link" ]]      # Symbolic link
[[ -b "$dev" ]]       # Block device
[[ -c "$dev" ]]       # Character device
[[ -p "$pipe" ]]      # Named pipe
[[ -S "$socket" ]]    # Socket

# File properties
[[ -e "$file" ]]      # Exists (any type)
[[ -r "$file" ]]      # Readable
[[ -w "$file" ]]      # Writable
[[ -x "$file" ]]      # Executable
[[ -s "$file" ]]      # Not empty (has size)
[[ -t 0 ]]            # stdin is terminal

# File comparison
[[ "$file1" -nt "$file2" ]]   # file1 is newer
[[ "$file1" -ot "$file2" ]]   # file1 is older
[[ "$file1" -ef "$file2" ]]   # Same inode (hard link)
```

### Loops

```bash
# For loop - iterate over array
for item in "${array[@]}"; do
    echo "Item: $item"
done

# For loop - iterate over range
for i in {1..10}; do
    echo "Number: $i"
done

# For loop - C-style
for ((i=0; i<10; i++)); do
    echo "Index: $i"
done

# While loop
while [[ $counter -lt 10 ]]; do
    echo "Counter: $counter"
    ((counter++))
done

# Until loop (opposite of while)
until [[ $counter -eq 10 ]]; do
    echo "Counter: $counter"
    ((counter++))
done

# Break and continue
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        continue  # Skip this iteration
    fi
    if [[ $i -eq 8 ]]; then
        break     # Exit loop
    fi
    echo "$i"
done

# Iterating over files
for file in /path/to/files/*; do
    [[ -f "$file" ]] || continue
    
    process_file "$file"
done

# Reading file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < "$filename"

# Process substitution (avoiding subshell)
while IFS= read -r line; do
    echo "Line: $line"
done < <(command)
```

### Case Statements

```bash
case "$option" in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    status)
        echo "Status: running"
        ;;
    *)
        echo "Unknown option: $option"
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac

# Pattern matching in case
case "$filename" in
    *.txt)
        echo "Text file"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        ;;
    *)
        echo "Unknown type"
        ;;
esac

# Case with regex (using =~)
case "$input" in
    [0-9]*)
        echo "Starts with number"
        ;;
    [A-Z]*)
        echo "Starts with uppercase"
        ;;
    *)
        echo "Other"
        ;;
esac
```

---

## 5. Text Processing & Regular Expressions

### Grep Advanced Usage

```bash
# Basic grep
grep "pattern" file.txt

# Extended regex
grep -E "pattern[0-9]+" file.txt

# Perl regex (more powerful)
grep -P "(?<=[Qq]uote).*?(?=[Qq]uote)" file.txt

# Invert match (show lines NOT matching)
grep -v "pattern" file.txt

# Multiple patterns (OR)
grep -E "pattern1|pattern2" file.txt

# Case-insensitive
grep -i "Pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Show context (lines before/after/both)
grep -B 2 -A 2 "pattern" file.txt
grep -C 3 "pattern" file.txt

# Only show matched part
grep -o "pattern" file.txt

# Recursive search in directory
grep -r "pattern" /path/to/directory

# Exclude files/directories
grep -r --exclude="*.log" --exclude-dir="node_modules" "pattern" .

# Count matches
grep -c "pattern" file.txt

# File matching patterns
grep -l "pattern" *.txt  # Show only filenames
grep -L "pattern" *.txt  # Show files NOT matching
```

### Sed (Stream Editor)

```bash
# Substitution
sed 's/old/new/' file.txt           # Replace first on each line
sed 's/old/new/g' file.txt          # Replace all occurrences
sed 's/old/new/2' file.txt          # Replace second occurrence
sed 's|/path|/newpath|' file.txt    # Use | as delimiter
sed 's/old/new/gi' file.txt         # Case-insensitive

# Delete lines
sed '/pattern/d' file.txt           # Delete matching lines
sed '2,5d' file.txt                 # Delete lines 2-5
sed '2d' file.txt                   # Delete line 2
sed '$ d' file.txt                  # Delete last line

# Print specific lines
sed -n '5,10p' file.txt             # Print lines 5-10
sed -n '/pattern/p' file.txt        # Print matching lines

# Insert/append
sed '/pattern/i\Inserted text' file.txt      # Insert before
sed '/pattern/a\Appended text' file.txt      # Append after
sed '2i\New line' file.txt                   # Insert at line 2

# Hold buffer operations
sed -n '1h;1!H;$!d;x;p' file.txt   # Reverse lines

# In-place editing
sed -i.bak 's/old/new/g' file.txt  # Creates backup
sed -i 's/old/new/g' file.txt      # No backup

# Backreferences
sed 's/\([a-z]*\) \([a-z]*\)/\2 \1/' file.txt  # Swap words

# Address ranges
sed '1,10s/old/new/' file.txt      # Only in lines 1-10
sed '/START/,/END/s/old/new/' file.txt  # Between patterns
```

### Awk

```bash
# Basic syntax: awk 'pattern { action }' file

# Print specific columns
awk '{print $1, $3}' file.txt       # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd    # Specify delimiter
awk '{print NF}' file.txt           # Print number of fields

# Pattern matching
awk '/pattern/ {print}' file.txt
awk '$2 > 100 {print $1, $2}' data.txt  # Conditional

# Variables
awk '{sum += $1} END {print sum}' file.txt
awk '{count++} END {print count}' file.txt

# Field processing
awk '{gsub(/,/, ""); print}' file.txt          # Remove commas
awk '{$2=""; print}' file.txt                  # Remove field 2
awk 'NF {print $(NF-1)}' file.txt              # Print second-to-last

# Multi-line processing
awk 'NR % 2 == 0 {next} {print}' file.txt     # Print odd lines

# String functions
awk '{print toupper($0)}' file.txt             # Uppercase
awk '{print substr($0, 1, 5)}' file.txt       # Substring
awk '{print length($0)}' file.txt              # String length
awk '{print index($0, "text")}' file.txt      # Find position

# Formatted output
awk '{printf "%-20s %10d\n", $1, $2}' file.txt
```

### Regex Patterns

```bash
# Character classes
.       # Any character
\d      # Digit (in extended regex: [0-9])
\w      # Word character
\s      # Whitespace
[a-z]   # Range
[^a-z]  # Negated range

# Anchors
^       # Start of line/string
$       # End of line/string
\<      # Word boundary start
\>      # Word boundary end

# Quantifiers
*       # Zero or more
+       # One or more
?       # Zero or one
{n}     # Exactly n
{n,}    # n or more
{n,m}   # Between n and m

# Groups and alternation
(abc)   # Group
(a|b)   # Alternation
(?:abc) # Non-capturing group (PCRE)

# Examples
^[0-9]{3}-[0-9]{3}-[0-9]{4}$     # Phone number
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$  # Email
```

---

## 6. File Operations & I/O Redirection

### Input/Output Redirection

```bash
# Output redirection
echo "hello" > file.txt           # Overwrite
echo "hello" >> file.txt          # Append
echo "error" >&2                  # Redirect to stderr

# Input redirection
while read line; do
    echo "$line"
done < input.txt

# Here document
cat <<EOF
This is multiline text
with $variables expanded
EOF

# Here string
grep "pattern" <<< "text to search in"

# Combine stdout and stderr
command > output.txt 2>&1
command &> output.txt             # Alternative syntax

# Discard output
command > /dev/null 2>&1

# File descriptors
exec 3< input.txt                # Open for reading as FD 3
exec 4> output.txt               # Open for writing as FD 4
read var <&3                     # Read from FD 3
echo "data" >&4                  # Write to FD 4
exec 3<&-                        # Close FD 3
exec 4>&-                        # Close FD 4

# Swap stdout and stderr
command 3>&1 1>&2 2>&3 3>&-

# Process substitution
diff <(sort file1.txt) <(sort file2.txt)
while read line; do echo "Line: $line"; done < <(cat file.txt)
```

### Working with Files

```bash
# Check file properties
if [[ -f "$file" && -r "$file" ]]; then
    cat "$file"
fi

# Safe file reading
if [[ -f "$file" ]]; then
    mapfile -t lines < "$file"
    for line in "${lines[@]}"; do
        echo "Processing: $line"
    done
fi

# Read specific lines
sed -n '10p' file.txt             # Line 10
sed -n '10,20p' file.txt          # Lines 10-20
head -n 20 file.txt               # First 20 lines
tail -n 20 file.txt               # Last 20 lines
tail -f file.txt                  # Follow (for logs)

# Temporary files
temp_file=$(mktemp)               # Create temp file
trap "rm -f '$temp_file'" EXIT    # Cleanup on exit

temp_dir=$(mktemp -d)             # Create temp directory
```

### Advanced I/O

```bash
# Read with delimiter
IFS=$'\t' read -r -a fields < line_with_tabs.txt

# Read without trailing newline
read -r -n 100 data < file.txt

# Prompt for input
read -p "Enter name: " name

# Silent password input
read -s -p "Password: " password

# Timeout for input
read -t 10 -p "Respond in 10 seconds: " response
if [[ -z "$response" ]]; then
    echo "Timeout"
fi

# Array of lines
while IFS= read -r line; do
    array+=("$line")
done < file.txt

echo "Total lines: ${#array[@]}"
```

---

## 7. Process Management

### Command Execution & Substitution

```bash
# Command substitution
result=$(command)                # Preferred (allows nesting)
result=`command`                 # Older syntax (avoid)

# Arithmetic expansion
result=$((10 + 5))
result=$((10 * 2))
result=$((100 / 4))
result=$((10 % 3))

# Increment/Decrement
((counter++))
((--counter))
((counter += 5))

# Compound commands
(
    cd /tmp
    touch tempfile
) # Subshell executes in subshell

{ 
    cd /tmp
    touch tempfile
} # Curly braces execute in current shell

# Background execution
long_running_command &

# Wait for background process
pid=$!
wait $pid
echo "Process $pid finished with status: $?"

# Wait for all background processes
wait

# Process substitution
comm <(sort file1) <(sort file2)
```

### Job Control

```bash
# Run process in background
command &

# Jobs list
jobs
jobs -l                          # With PID
jobs -r                          # Running only
jobs -s                          # Stopped only

# Bring job to foreground
fg %1

# Continue stopped job in background
bg %1

# Process info
ps aux | grep process
ps -ef --forest                  # Tree view
pgrep process_name               # Get PID by name
pkill -f "pattern"               # Kill by pattern
```

### Signal Handling

```bash
# Trap signals
trap 'echo "Cleaning up..."; exit' SIGINT SIGTERM

# Multiple actions
trap 'cleanup' EXIT SIGINT SIGTERM

cleanup() {
    [[ -n "${temp_file:-}" ]] && rm -f "$temp_file"
    [[ -n "${temp_dir:-}" ]] && rm -rf "$temp_dir"
    exit 0
}

# Send signals
kill -SIGTERM $pid
kill -9 $pid                     # SIGKILL (force)

# Wait with cleanup
trap 'kill $bg_pid 2>/dev/null' EXIT
command &
bg_pid=$!
wait $bg_pid
```

---

## 8. Error Handling & Debugging

### Error Handling Best Practices

```bash
#!/bin/bash
set -euo pipefail

# Custom error handling
error_exit() {
    local exit_code=$?
    local line_number=$1
    echo "ERROR: Script failed at line $line_number with exit code $exit_code" >&2
    exit "$exit_code"
}

trap 'error_exit ${LINENO}' ERR

# Try-catch pattern
try() {
    [[ $- = *e* ]]; SAVED_OPT_E=$?
    set +e
}

catch() {
    export exception_code=$?
    (( SAVED_OPT_E )) && set +e
    return $exception_code
}

throw() {
    exit $1
}

# Usage
try
(
    # Commands that might fail
    false
)
catch || {
    case $exception_code in
        1)
            echo "Error 1"
            ;;
        *)
            echo "Unknown error: $exception_code"
            ;;
    esac
}

# Conditional execution
command && echo "Success" || echo "Failed"

# Error context
set -E  # Inherit ERR trap
command || {
    local exit_code=$?
    echo "Command failed with code $exit_code at $FUNCNAME:$LINENO" >&2
    return $exit_code
}
```

### Debugging Techniques

```bash
# Enable debugging
bash -x script.sh               # Trace all commands
bash -v script.sh               # Print commands before execution

# Inside script
set -x                          # Enable tracing
set +x                          # Disable tracing
set -v                          # Verbose mode

# Debug specific section
{
    set -x
    problematic_code
    set +x
} 2>&1 | tee debug.log

# Print PS4 for better debugging
PS4='+ (${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x

# Verify syntax
bash -n script.sh               # Check syntax

# Debug variables
declare -p                      # Print all variables
declare -p myvar                # Print specific variable
env | grep PATTERN              # Check environment

# Assertions
assert() {
    "$@" || {
        echo "Assertion failed: $*" >&2
        exit 1
    }
}

assert [[ -f "$required_file" ]]
```

### Logging

```bash
# Basic logging function
log() {
    local level="$1"
    shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

log_info() {
    log "INFO" "$@"
}

log_warn() {
    log "WARN" "$@"
}

log_error() {
    log "ERROR" "$@" >&2
}

log_debug() {
    [[ "${DEBUG:-false}" == "true" ]] && log "DEBUG" "$@"
}

# Usage
log_info "Script started"
log_warn "This is a warning"
log_error "An error occurred"
log_debug "Debug information"

# Rotating logs
rotate_log() {
    local log_file="$1"
    local max_size=1048576  # 1MB
    
    if [[ -f "$log_file" ]] && [[ $(stat -f%z "$log_file") -gt $max_size ]]; then
        mv "$log_file" "${log_file}.$(date +%s)"
        gzip "${log_file}".*  # Compress old logs
    fi
}
```

---

## 9. Advanced Scripting Patterns

### Configuration Management

```bash
# Source configuration file
readonly DEFAULT_CONFIG="/etc/myapp/config.conf"
CONFIG_FILE="${CONFIG_FILE:-$DEFAULT_CONFIG}"

if [[ ! -f "$CONFIG_FILE" ]]; then
    echo "Config file not found: $CONFIG_FILE" >&2
    exit 1
fi

# Source with safety checks
if source "$CONFIG_FILE"; then
    log_info "Configuration loaded from $CONFIG_FILE"
else
    log_error "Failed to load configuration"
    exit 1
fi

# Validate configuration
validate_config() {
    local required_vars=(
        "APP_NAME"
        "APP_PORT"
        "LOG_LEVEL"
    )
    
    for var in "${required_vars[@]}"; do
        if [[ -z "${!var:-}" ]]; then
            log_error "Required config variable not set: $var"
            return 1
        fi
    done
    
    return 0
}

validate_config || exit 1
```

### Option Parsing

```bash
# Getopts for POSIX compliance
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS]
Options:
    -h              Show this help message
    -v              Verbose output
    -c CONFIG       Configuration file
    -p PORT         Port number
EOF
}

verbose=false
config_file=""
port=""

while getopts "hvc:p:" opt; do
    case $opt in
        h)
            usage
            exit 0
            ;;
        v)
            verbose=true
            ;;
        c)
            config_file="$OPTARG"
            ;;
        p)
            port="$OPTARG"
            ;;
        *)
            usage >&2
            exit 1
            ;;
    esac
done

shift $((OPTIND - 1))

# Advanced: getopt for long options
OPTS=$(getopt -o hvc:p: --long help,verbose,config:,port:,timeout: -n "$SCRIPT_NAME" -- "$@")

eval set -- "$OPTS"

while true; do
    case "$1" in
        -h|--help)
            usage
            exit 0
            ;;
        -v|--verbose)
            verbose=true
            shift
            ;;
        -c|--config)
            config_file="$2"
            shift 2
            ;;
        -p|--port)
            port="$2"
            shift 2
            ;;
        --timeout)
            timeout="$2"
            shift 2
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "Internal error!" >&2
            exit 1
            ;;
    esac
done

# Remaining arguments
echo "Positional arguments: $@"
```

### Validation & Input Sanitization

```bash
# Validate email
validate_email() {
    local email="$1"
    if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    fi
    return 1
}

# Validate IP address
validate_ip() {
    local ip="$1"
    if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
        local IFS=.
        local -a octets=($ip)
        for octet in "${octets[@]}"; do
            if (( octet > 255 )); then
                return 1
            fi
        done
        return 0
    fi
    return 1
}

# Validate port
validate_port() {
    local port="$1"
    if [[ $port =~ ^[0-9]+$ ]] && (( port >= 0 && port <= 65535 )); then
        return 0
    fi
    return 1
}

# Sanitize filename (remove unsafe characters)
sanitize_filename() {
    local filename="$1"
    # Remove everything except alphanumeric, dash, underscore, dot
    echo "$filename" | sed 's/[^a-zA-Z0-9._-]/_/g'
}

# Escape for SQL (basic protection - use parameterized queries in real code)
escape_sql() {
    local input="$1"
    echo "$input" | sed "s/'/''/g"
}

# Sanitize for shell (prevent injection)
quote_arg() {
    printf '%q\n' "$1"
}
```

### Locking & Concurrency

```bash
# File-based lock
acquire_lock() {
    local lock_file="$1"
    local timeout="${2:-30}"
    local elapsed=0
    
    while [[ -f "$lock_file" ]] && [[ $elapsed -lt $timeout ]]; do
        sleep 1
        ((elapsed++))
    done
    
    if [[ -f "$lock_file" ]]; then
        return 1  # Failed to acquire
    fi
    
    touch "$lock_file"
    return 0
}

release_lock() {
    local lock_file="$1"
    rm -f "$lock_file"
}

# Usage
LOCK_FILE="/var/lock/myapp.lock"

acquire_lock "$LOCK_FILE" || {
    echo "Could not acquire lock" >&2
    exit 1
}

trap "release_lock '$LOCK_FILE'" EXIT

# Critical section
do_critical_work
```

---

## 10. Performance Optimization

### Efficient Loops & Operations

```bash
# Avoid unnecessary subshells
# SLOW: Uses subshell for each iteration
cat file.txt | while read line; do
    process "$line"
done

# FAST: Reads directly from file
while read line; do
    process "$line"
done < file.txt

# FASTER: Use process substitution if subshell unavailable
while read line; do
    process "$line"
done < <(cat file.txt)

# Builtin vs external commands
# SLOW: Spawns external process
for file in $(ls *.txt); do
    process_file "$file"
done

# FAST: Uses globbing (no process spawning)
for file in *.txt; do
    process_file "$file"
done

# String comparison vs regex
# SLOWER
if [[ $string =~ ^[0-9]+$ ]]; then
    echo "is number"
fi

# FASTER (if simple check)
if [[ $string == [0-9]* ]]; then
    echo "starts with number"
fi

# Avoid repeated expansions
# SLOW
for i in {1..1000000}; do
    echo "${array[@]}"
done

# FAST
array_str="${array[@]}"
for i in {1..1000000}; do
    echo "$array_str"
done
```

### Memoization & Caching

```bash
# Cache function results
declare -A function_cache

cached_function() {
    local key="$1"
    
    if [[ -v function_cache[$key] ]]; then
        echo "${function_cache[$key]}"
        return 0
    fi
    
    local result=$(expensive_operation "$key")
    function_cache[$key]="$result"
    echo "$result"
}

# File-based cache
cache_get() {
    local cache_file="$1"
    local cache_duration="${2:-3600}"  # 1 hour
    
    if [[ -f "$cache_file" ]]; then
        local file_age=$(($(date +%s) - $(stat -f%m "$cache_file" 2>/dev/null || echo 0)))
        if [[ $file_age -lt $cache_duration ]]; then
            cat "$cache_file"
            return 0
        fi
    fi
    return 1
}

cache_set() {
    local cache_file="$1"
    local data="$2"
    echo "$data" > "$cache_file"
}
```

### Parallel Processing

```bash
# GNU Parallel
parallel --jobs 4 'process_file {}' ::: *.txt

# xargs for parallel processing
find . -name "*.txt" | xargs -P 4 -I {} process_file {}

# Manual parallel with background jobs
process_batch() {
    local max_jobs=4
    local job_count=0
    
    for file in *.txt; do
        # Wait if we have enough background jobs
        while [[ $(jobs -r | wc -l) -ge $max_jobs ]]; do
            sleep 0.1
        done
        
        # Start background job
        process_file "$file" &
    done
    
    # Wait for all jobs to complete
    wait
}

# Process substitution with parallel
while read -r file; do
    process_file "$file" &
done < <(find . -name "*.txt")
wait
```

---

## 11. Security Best Practices

### Input Validation & Sanitization

```bash
# Never use eval without extreme caution
# DON'T: eval $untrusted_input

# Safe variable assignment
user_input="$1"  # Direct assignment is safe

# Validate before using in commands
if [[ $user_input =~ ^[a-zA-Z0-9]+$ ]]; then
    safe_command "$user_input"
fi

# Quote all variables to prevent globbing/word splitting
# BAD
echo $var

# GOOD
echo "$var"

# Use [[ ]] instead of [ ] for better security
# BAD
if [ "$var" = value ]; then
    echo "match"
fi

# GOOD
if [[ "$var" == "value" ]]; then
    echo "match"
fi
```

### File & Permissions

```bash
# Secure temporary file creation
temp_file=$(mktemp -t myapp.XXXXXX) || exit 1
trap "rm -f '$temp_file'" EXIT

# Set permissions before writing sensitive data
temp_file=$(mktemp -t myapp.XXXXXX) || exit 1
chmod 600 "$temp_file"
trap "rm -f '$temp_file'" EXIT

# Safe file operations
# Check before overwriting
if [[ -e "$file" ]]; then
    read -p "File exists. Overwrite? (y/n) " -r
    [[ $REPLY == [Yy] ]] || return 1
fi

# Verify permissions before reading sensitive files
if [[ -f "$sensitive_file" ]]; then
    perms=$(stat -f%A "$sensitive_file" 2>/dev/null || echo "unknown")
    if [[ ! "$perms" =~ ^-{2}0 ]]; then
        echo "WARNING: Sensitive file has world-readable permissions" >&2
    fi
fi
```

### String Handling & Injection Prevention

```bash
# Use printf instead of echo for better control
printf '%s\n' "$var"           # Safe
echo "$var"                    # Might interpret escape sequences

# Prevent command injection
# DON'T: 
system_command="ls -la $filename"
eval "$system_command"

# DO:
find . -name "$filename" -type f

# Avoid using shell metacharacters in variables
# DON'T:
filename="$(echo $user_input)"

# DO:
filename="$(printf '%q' "$user_input")"

# Use declare -a for array safety
declare -a safe_args
safe_args=("$arg1" "$arg2")
command "${safe_args[@]}"

# Don't trust $IFS
old_ifs="$IFS"
IFS=
read -r -a array < file.txt
IFS="$old_ifs"
```

### Environment & Credentials

```bash
# Don't hardcode credentials
# DON'T:
PASSWORD="secret123"

# DO: Use environment variables or files with restricted permissions
source /etc/myapp/credentials.conf  # chmod 600

# Never expose sensitive data in logs
# DON'T:
echo "Connecting with password: $PASSWORD" | tee -a "$LOG_FILE"

# DO:
echo "Connecting..." >> "$LOG_FILE"

# Unset sensitive variables when done
sensitive_var="$PASSWORD"
# ... use it ...
unset sensitive_var

# Verify script source
if [[ ! -O "$BASH_SOURCE" ]]; then
    echo "WARNING: Script not owned by current user" >&2
fi
```

---

## 12. Real-World Examples

### Example 1: Backup Script

```bash
#!/bin/bash
set -euo pipefail

readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"
readonly LOG_FILE="/var/log/backup.log"
readonly BACKUP_DIR="/backups"
readonly SOURCE_DIR="/home/user/data"
readonly MAX_BACKUPS=7

log() {
    local level="$1"
    shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR" "$*"
    exit 1
}

create_backup() {
    local backup_date=$(date +%Y-%m-%d_%H-%M-%S)
    local backup_file="${BACKUP_DIR}/backup_${backup_date}.tar.gz"
    
    log "INFO" "Starting backup to $backup_file"
    
    if tar -czf "$backup_file" -C "$(dirname "$SOURCE_DIR")" "$(basename "$SOURCE_DIR")" 2>&1 | tee -a "$LOG_FILE"; then
        log "INFO" "Backup completed successfully"
        echo "$backup_file"
    else
        error_exit "Backup failed"
    fi
}

cleanup_old_backups() {
    log "INFO" "Cleaning up old backups (keeping $MAX_BACKUPS)"
    
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -type f | sort -r | tail -n +$((MAX_BACKUPS + 1)) | while read -r old_backup; do
        log "INFO" "Removing old backup: $old_backup"
        rm -f "$old_backup"
    done
}

verify_backup() {
    local backup_file="$1"
    
    log "INFO" "Verifying backup integrity"
    
    if tar -tzf "$backup_file" > /dev/null 2>&1; then
        log "INFO" "Backup verification successful"
        return 0
    else
        log "ERROR" "Backup verification failed"
        return 1
    fi
}

main() {
    log "INFO" "$SCRIPT_NAME started"
    
    [[ -d "$BACKUP_DIR" ]] || mkdir -p "$BACKUP_DIR"
    [[ -d "$SOURCE_DIR" ]] || error_exit "Source directory not found: $SOURCE_DIR"
    
    local backup_file
    backup_file=$(create_backup)
    verify_backup "$backup_file"
    cleanup_old_backups
    
    log "INFO" "$SCRIPT_NAME completed successfully"
}

trap 'error_exit "Script interrupted"' SIGINT SIGTERM
main "$@"
```

### Example 2: System Health Monitor

```bash
#!/bin/bash
set -euo pipefail

readonly ALERT_THRESHOLD_CPU=80
readonly ALERT_THRESHOLD_MEMORY=85
readonly ALERT_THRESHOLD_DISK=90

check_cpu_usage() {
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}' | cut -d'.' -f1)
    
    if [[ $cpu_usage -gt $ALERT_THRESHOLD_CPU ]]; then
        echo "WARNING: High CPU usage: ${cpu_usage}%"
        return 1
    fi
    echo "OK: CPU usage: ${cpu_usage}%"
    return 0
}

check_memory_usage() {
    local mem_usage
    mem_usage=$(free | grep Mem | awk '{printf("%.0f", $3/$2 * 100)}')
    
    if [[ $mem_usage -gt $ALERT_THRESHOLD_MEMORY ]]; then
        echo "WARNING: High memory usage: ${mem_usage}%"
        return 1
    fi
    echo "OK: Memory usage: ${mem_usage}%"
    return 0
}

check_disk_usage() {
    local results=()
    
    while read -r line; do
        local disk_usage=$(echo "$line" | awk '{print $5}' | sed 's/%//')
        local mount=$(echo "$line" | awk '{print $NF}')
        
        if [[ $disk_usage -gt $ALERT_THRESHOLD_DISK ]]; then
            results+=("WARNING: High disk usage on $mount: ${disk_usage}%")
        else
            results+=("OK: Disk usage on $mount: ${disk_usage}%")
        fi
    done < <(df -h | tail -n +2)
    
    for result in "${results[@]}"; do
        echo "$result"
    done
}

check_services() {
    local services=("sshd" "nginx" "postgresql")
    
    for service in "${services[@]}"; do
        if systemctl is-active --quiet "$service"; then
            echo "OK: Service $service is running"
        else
            echo "WARNING: Service $service is not running"
        fi
    done
}

main() {
    echo "=== System Health Check ===="
    echo "Timestamp: $(date)"
    echo ""
    
    check_cpu_usage
    check_memory_usage
    check_disk_usage
    check_services
}

main "$@"
```

### Example 3: Log Analyzer

```bash
#!/bin/bash
set -euo pipefail

analyze_logs() {
    local log_file="$1"
    local pattern="${2:-ERROR}"
    
    if [[ ! -f "$log_file" ]]; then
        echo "Log file not found: $log_file" >&2
        return 1
    fi
    
    echo "=== Log Analysis Report ==="
    echo "File: $log_file"
    echo "Analysis Date: $(date)"
    echo ""
    
    # Count occurrences
    local total_lines
    total_lines=$(wc -l < "$log_file")
    
    local matching_lines
    matching_lines=$(grep -c "$pattern" "$log_file" || true)
    
    echo "Total lines: $total_lines"
    echo "Matching lines ($pattern): $matching_lines"
    echo "Percentage: $(( matching_lines * 100 / total_lines ))%"
    echo ""
    
    # Top error sources
    echo "=== Top 5 Error Sources ==="
    grep "$pattern" "$log_file" | \
        awk '{print $NF}' | \
        sort | uniq -c | sort -rn | head -5
    
    echo ""
    
    # Recent errors
    echo "=== 10 Most Recent Errors ==="
    grep "$pattern" "$log_file" | tail -10
}

main() {
    local log_file="${1:-.}"
    local pattern="${2:-ERROR}"
    
    if [[ -d "$log_file" ]]; then
        for file in "$log_file"/*.log; do
            analyze_logs "$file" "$pattern"
            echo "---"
        done
    else
        analyze_logs "$log_file" "$pattern"
    fi
}

main "$@"
```

---

## Additional Resources

### Useful References
- Bash Manual: https://www.gnu.org/software/bash/manual/
- ShellCheck: https://www.shellcheck.net/
- Google Shell Style Guide: https://google.github.io/styleguide/shellguide.html
- POSIX Shell: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html

### Tools & Commands
- `shellcheck`: Analyze shell scripts for errors
- `shfmt`: Format shell scripts
- `bash -x`: Debug scripts
- `strace`: Trace system calls

### Common Pitfalls to Avoid
1. Not quoting variables → word splitting and globbing errors
2. Using `[ ]` instead of `[[ ]]` → less reliable
3. Not checking file existence → runtime errors
4. Ignoring exit codes → silent failures
5. Using eval with untrusted input → security vulnerability
6. Not setting `set -euo pipefail` → harder to debug failures
7. Hardcoding paths → breaks on different systems
8. Not handling signals → orphaned processes
9. Inefficient loops → poor performance
10. Not validating input → security and stability issues

---

## Practice Exercises

1. **Write a deployment script** that:
   - Validates configuration
   - Backs up current version
   - Deploys new code
   - Verifies deployment
   - Rolls back on failure

2. **Create a monitoring script** that:
   - Checks system resources
   - Parses application logs
   - Sends alerts on thresholds
   - Generates reports

3. **Build a data processing pipeline** that:
   - Validates input files
   - Processes with awk/sed
   - Transforms data
   - Generates output

4. **Develop a configuration manager** that:
   - Loads configuration files
   - Validates settings
   - Provides defaults
   - Supports overrides

---

## Conclusion

This guide provides a comprehensive foundation for advanced shell scripting. Master these concepts through practice and always prioritize **readability, safety, and maintainability** in your scripts. Happy scripting!

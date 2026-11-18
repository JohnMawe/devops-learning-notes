# Bash Scripting Reference & Learning Notes
## Lessons 1–14 | Ubuntu / Bash

> A practical reference for Bash scripting, Linux automation, CLI tools, process management, and shell environments. Examples assume Bash on Ubuntu unless noted.

---

## Table of Contents

1. [Bash and the Shell](#1-bash-and-the-shell)
2. [Scripts, Shebangs, Permissions, and Execution](#2-scripts-shebangs-permissions-and-execution)
3. [Variables, Quoting, Expansion, and Data](#3-variables-quoting-expansion-and-data)
4. [Input, Output, Streams, and Command Substitution](#4-input-output-streams-and-command-substitution)
5. [Arguments and Positional Parameters](#5-arguments-and-positional-parameters)
6. [Arithmetic, Operators, and Comparisons](#6-arithmetic-operators-and-comparisons)
7. [Conditions and Decision Making](#7-conditions-and-decision-making)
8. [Loops](#8-loops)
9. [Functions, Scope, and Return Status](#9-functions-scope-and-return-status)
10. [Files, Directories, Pipes, and Text Processing](#10-files-directories-pipes-and-text-processing)
11. [Indexed and Associative Arrays](#11-indexed-and-associative-arrays)
12. [Command-Line Interfaces with `getopts`](#12-command-line-interfaces-with-getopts)
13. [Exit Codes, Error Handling, Strict Mode, and Traps](#13-exit-codes-error-handling-strict-mode-and-traps)
14. [Process Management](#14-process-management)
15. [Environment Variables, PATH, and Bash Configuration](#15-environment-variables-path-and-bash-configuration)
16. [Debugging and Testing](#16-debugging-and-testing)
17. [Safety and Good Practices](#17-safety-and-good-practices)
18. [Useful Command Cheat Sheet](#18-useful-command-cheat-sheet)
19. [Reusable Script Template](#19-reusable-script-template)
20. [Practice Projects and Next Steps](#20-practice-projects-and-next-steps)

---

## 1. Bash and the Shell

### What is Bash?

**Bash (Bourne Again Shell)** is:
- A command interpreter that reads and executes commands.
- A scripting language for automating tasks.
- A shell that provides variables, conditions, loops, functions, pipes, redirection, and process control.

The **Linux kernel** manages hardware, memory, processes, and other core operating-system responsibilities. A **shell** provides an interface through which users and scripts interact with the operating system.

Bash is one shell. Other shells include `sh`, `zsh`, `fish`, and others.

Check Bash version:

```bash
bash --version
echo "$BASH_VERSION"
```

### Interactive commands and scripts

A command entered directly in a terminal is interactive use. A file containing commands executed in sequence is a script.

Bash scripts are especially useful for:
- Repeating Linux commands.
- Managing files and directories.
- Checking services and system resources.
- Backups.
- Log analysis.
- Deployment tasks.
- Docker operations.
- Running Python programs with prepared environment/configuration.

---

## 2. Scripts, Shebangs, Permissions, and Execution

### Create a script

```bash
mkdir -p bash-learning
cd bash-learning
nano hello.sh
```

Example:

```bash
#!/bin/bash

# My first Bash script

echo "Hello, Linux!"
echo "I am learning Bash scripting."
pwd
whoami
```

### Shebang

The first line:

```bash
#!/bin/bash
```

tells the operating system which interpreter to use when executing the script directly.

The `#!` sequence is called the shebang.

### Comments

```bash
# This is a comment
```

Comments help explain the intent of code. The shebang is a special first-line use of `#`.

### Execute a script

Run it explicitly with Bash:

```bash
bash hello.sh
```

Or make it executable:

```bash
chmod +x hello.sh
./hello.sh
```

`chmod +x` adds execute permission. `./` means execute a file from the current directory.

Inspect permissions:

```bash
ls -l hello.sh
```

### Other useful permissions

```bash
chmod 755 script.sh
chmod 700 private_script.sh
chmod 644 notes.txt
```

Use permissions appropriate to the file. Avoid making everything executable or unnecessarily granting broad permissions.

---

## 3. Variables, Quoting, Expansion, and Data

### Assign variables

No spaces around `=`:

```bash
name="John"
project="AdviceAPI"
version=1
```

Incorrect:

```bash
name = "John"
```

Access a variable:

```bash
echo "$name"
printf 'Project: %s\n' "$project"
```

Braces delimit variable names:

```bash
echo "${project}_backend"
echo "${name}Developer"
```

### Bash variable data

Bash variables are generally text-like. Arithmetic contexts can interpret values as integers, but Bash does not provide ordinary typed variables like Python.

```bash
age=25
sum=$((10 + 5))
```

### Double quotes

Double quotes preserve spaces and allow variable expansion:

```bash
name="John Mawe"
echo "Hello, $name"
```

### Single quotes

Single quotes preserve literal text; variable expansion does not occur:

```bash
echo 'The value of $HOME is not expanded here'
```

### Quote expansions

Prefer:

```bash
rm -- "$filename"
cp -- "$source" "$destination"
echo "$message"
```

Unquoted expansions can undergo word splitting and pathname expansion. Quoting protects spaces and many unexpected characters.

### Environment variables

Common variables:

```bash
echo "$HOME"
echo "$USER"
echo "$PWD"
echo "$PATH"
echo "$SHELL"
echo "$HOSTNAME"
```

Export a variable for child processes:

```bash
export PROJECT="AdviceAPI"
export APP_ENV="development"
```

### Defaults for unset/empty values

```bash
name="${name:-Guest}"
output="${1:-report.txt}"
```

`${variable:-default}` substitutes the default if the variable is unset or empty.

Related forms:
- `${var-default}`: default if unset (but not if set empty).
- `${var:=default}`: assign default if unset or empty.
- `${var:?message}`: produce an error if unset or empty.
- `${var:+alternate}`: use alternate if set and nonempty.

With `set -u`, these forms are particularly helpful for safely handling optional variables.

### Unset variables

```bash
unset name
```

Removes a variable from the current shell. If exported, it is also removed from the environment.

### Boolean-like values

Bash has no standard native Boolean variable type. Common conventions:

```bash
verbose=false
verbose=true

if [[ "$verbose" == true ]]; then
    echo "Verbose mode"
fi
```

Command exit status is often a more natural success/failure indicator.

---

## 4. Input, Output, Streams, and Command Substitution

### Standard streams

| Stream | FD | Meaning |
|---|---:|---|
| stdin | 0 | Standard input |
| stdout | 1 | Standard output |
| stderr | 2 | Standard error |

### Output

```bash
echo "Hello"
printf 'Name: %s\n' "$name"
printf 'Count: %d\n' "$count"
printf 'Decimal: %.2f\n' "$value"
printf '%-18s %s\n' "Username:" "$USER"
```

`printf` is generally preferred for predictable formatting.

### Read user input

```bash
read -r -p "Enter your name: " name
echo "Hello, $name"
```

`-r` prevents backslashes from being interpreted as escapes.

Read multiple values:

```bash
read -r -p "Enter first and last name: " first last
```

Read a hidden value:

```bash
read -r -s -p "Password: " password
echo
```

Avoid printing secrets or storing them insecurely.

### Command substitution

Capture command output:

```bash
username=$(whoami)
current_date=$(date)
current_dir=$(pwd)
file_count=$(find . -maxdepth 1 -type f | wc -l)
```

Prefer `$(...)` over legacy backticks.

Command substitution strips trailing newline characters. It is not suitable for arbitrary binary data and can be unsuitable for preserving arbitrary multiline content exactly.

### Redirection

```bash
command > output.txt       # stdout: overwrite
command >> output.txt      # stdout: append
command 2> errors.txt     # stderr: overwrite
command 2>> errors.txt    # stderr: append
command < input.txt       # stdin from file
command > output.txt 2>&1 # stdout and stderr to same destination
command &> output.txt     # Bash shorthand for both streams
```

Order matters. `command >file 2>&1` redirects both to the file. `command 2>&1 >file` is not equivalent.

Discard output:

```bash
command >/dev/null 2>&1
```

### Pipes

```bash
command1 | command2
```

The stdout of the first command becomes stdin of the second.

Examples:

```bash
ls /etc | sort | head -10
ps aux | grep python
cat application.log | grep ERROR
```

Often avoid unnecessary `cat`:

```bash
grep ERROR application.log
wc -l < application.log
```

---

## 5. Arguments and Positional Parameters

Example invocation:

```bash
./script.sh AdviceAPI Python Development
```

| Parameter | Meaning |
|---|---|
| `$0` | Script name/path as invoked |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` | Third argument |
| `${10}` | Tenth argument |
| `$#` | Number of arguments |
| `"$@"` | All arguments, each preserved separately when quoted |
| `"$*"` | All arguments joined into one string in quoted context |

### Validate argument count

```bash
if [[ "$#" -ne 2 ]]; then
    echo "Usage: $0 NAME LANGUAGE" >&2
    exit 1
fi
```

### Loop through all arguments

```bash
for argument in "$@"; do
    printf 'Argument: %s\n' "$argument"
done
```

Use `"$@"` to preserve arguments separately, including those containing spaces.

### `shift`

```bash
echo "First: $1"
shift
echo "New first: $1"
```

`shift` removes the first positional parameter and shifts the rest left. It is useful for manual argument parsing.

### Validate files/paths

```bash
if [[ -e "$path" ]]; then
    echo "Path exists"
fi
```

Check directory:

```bash
[[ -d "$path" ]]
```

Check file:

```bash
[[ -f "$path" ]]
```

---

## 6. Arithmetic, Operators, and Comparisons

### Arithmetic expansion

```bash
sum=$((10 + 5))
difference=$((10 - 5))
product=$((10 * 5))
quotient=$((10 / 3))
remainder=$((10 % 3))
```

Bash integer division truncates; `10 / 3` produces `3`.

### Increment/decrement

```bash
count=$((count + 1))
count=$((count - 1))
count=$((count * 2))
count=$((count / 2))
```

Also:

```bash
((count++))
((count--))
((count += 5))
```

Arithmetic command `(( expression ))` returns an exit status based on the expression's value: zero evaluates to status 1; nonzero evaluates to status 0. Be mindful of this when using it with `set -e`.

### Numeric comparisons

In `[[ ... ]]` or `[ ... ]`:

| Operator | Meaning |
|---|---|
| `-eq` | equal |
| `-ne` | not equal |
| `-gt` | greater than |
| `-ge` | greater than or equal |
| `-lt` | less than |
| `-le` | less than or equal |

Example:

```bash
if [[ "$age" -ge 18 ]]; then
    echo "Adult"
fi
```

Arithmetic comparison:

```bash
if (( age >= 18 )); then
    echo "Adult"
fi
```

### String comparisons

```bash
[[ "$a" == "$b" ]]
[[ "$a" != "$b" ]]
[[ -z "$value" ]]  # empty
[[ -n "$value" ]]  # nonempty
```

In Bash, `[[ ... ]]` is generally recommended for conditional tests. `[ ... ]` remains common and more portable.

### Logical operators

```bash
[[ condition1 && condition2 ]]
[[ condition1 || condition2 ]]
[[ ! condition ]]
```

- `&&`: AND
- `||`: OR
- `!`: NOT

### Validate integers with regex

```bash
if [[ "$number" =~ ^[0-9]+$ ]]; then
    echo "Non-negative integer"
else
    echo "Invalid"
fi
```

This regex accepts zero or more digits? `+` means one or more, so empty strings are rejected. It accepts only non-negative integers.

Bash arithmetic is integer-only for ordinary arithmetic. For decimals, use tools such as `awk`, `bc` where available, or Python.

---

## 7. Conditions and Decision Making

### `if`, `elif`, `else`

```bash
if [[ condition ]]; then
    commands
elif [[ another_condition ]]; then
    commands
else
    commands
fi
```

Example:

```bash
if [[ "$environment" == production ]]; then
    echo "Production"
elif [[ "$environment" == development ]]; then
    echo "Development"
else
    echo "Unknown environment"
fi
```

### File tests

| Test | Meaning |
|---|---|
| `-e path` | Exists |
| `-f path` | Regular file |
| `-d path` | Directory |
| `-r path` | Readable |
| `-w path` | Writable |
| `-x path` | Executable |
| `-s path` | Nonempty |
| `-L path` | Symbolic link |

Example:

```bash
if [[ -f "$file" && -r "$file" ]]; then
    echo "Readable regular file"
fi
```

### `case`

Useful for menus and matching patterns:

```bash
case "$choice" in
    start)
        echo "Starting"
        ;;
    stop)
        echo "Stopping"
        ;;
    restart)
        echo "Restarting"
        ;;
    *)
        echo "Unknown option"
        ;;
esac
```

`;;` ends a case branch. `*` matches any remaining pattern.

### Test a command's status directly

```bash
if command -v docker >/dev/null 2>&1; then
    echo "Docker command exists"
else
    echo "Docker not found"
fi
```

Check Docker daemon:

```bash
if docker info >/dev/null 2>&1; then
    echo "Docker is available"
else
    echo "Docker unavailable or permission denied"
fi
```

---

## 8. Loops

### `for`

```bash
for item in one two three; do
    echo "$item"
done
```

### Arrays

```bash
items=("file1" "file2" "file3")

for item in "${items[@]}"; do
    echo "$item"
done
```

### Brace ranges

```bash
for number in {1..5}; do
    echo "$number"
done

for number in {0..10..2}; do
    echo "$number"
done
```

### C-style loop

```bash
for ((i=1; i<=5; i++)); do
    echo "$i"
done
```

### `while`

Runs while condition is true:

```bash
count=1

while (( count <= 5 )); do
    echo "$count"
    ((count++))
done
```

Be careful with `((count++))` and `set -e`; an arithmetic expression evaluating to zero can produce a nonzero command status. Safer alternatives in strict-mode scripts include `count=$((count + 1))` or `((++count))` when appropriate.

### `until`

Runs until the condition becomes true:

```bash
count=1

until (( count > 5 )); do
    echo "$count"
    count=$((count + 1))
done
```

### `break` and `continue`

```bash
for number in {1..10}; do
    if (( number == 5 )); then
        break
    fi
    echo "$number"
done
```

`break` exits the loop.

```bash
for number in {1..5}; do
    if (( number == 3 )); then
        continue
    fi
    echo "$number"
done
```

`continue` skips the current iteration.

### Infinite loop

```bash
while true; do
    commands
    sleep 1
done
```

Include a stop condition, delay, or signal-handling approach to avoid uncontrolled loops.

---

## 9. Functions, Scope, and Return Status

### Define and call

```bash
greet() {
    echo "Hello"
}

greet
```

### Function arguments

```bash
greet() {
    local name="$1"
    echo "Hello, $name"
}

greet "John"
```

Function positional parameters (`$1`, `$2`, `$@`, `$#`) refer to the function's arguments while it runs. They do not replace the script's original arguments for the function body.

### Local variables

```bash
my_function() {
    local result="temporary"
    echo "$result"
}
```

Use `local` to avoid unintentionally changing variables in the surrounding shell.

### Return status

Bash functions return an exit status (0–255), not arbitrary strings through `return`.

```bash
check_file() {
    [[ -f "$1" ]]
}

if check_file "config.py"; then
    echo "File exists"
else
    echo "Missing"
fi
```

### Capture output

```bash
get_username() {
    whoami
}

username=$(get_username)
```

Use stdout for returned data and stderr for diagnostics:

```bash
my_function() {
    echo "Working..." >&2
    printf '%s\n' "result"
}

result=$(my_function)
```

### Organize scripts

Functions help structure scripts:

```bash
print_header() {
    printf '===== %s =====\n' "$1"
}

show_system_info() {
    hostname
    whoami
    date
}

main() {
    print_header "System Report"
    show_system_info
}

main "$@"
```

---

## 10. Files, Directories, Pipes, and Text Processing

### Common file operations

```bash
touch file.txt
mkdir directory
mkdir -p parent/child
cp source destination
mv old new
rm file
rmdir empty_directory
```

Recursive removal:

```bash
rm -r directory
```

Force removal:

```bash
rm -rf directory
```

`rm -rf` is destructive. Verify paths and variables before using it, especially in scripts.

### Find files

```bash
find . -type f -name "*.py"
find . -type d
find . -mtime -1
find . -size +100M
```

Options:
- `-type f`: regular files.
- `-type d`: directories.
- `-name`: match name.
- `-mtime`: modification age in days.
- `-size`: size matching.

### Filenames with spaces and unusual characters

Safer loop:

```bash
while IFS= read -r -d '' file; do
    printf 'File: %s\n' "$file"
done < <(find . -type f -print0)
```

NUL-delimited processing (`-print0`, `read -d ''`) is safe for filenames containing spaces, tabs and newlines. Newlines alone are not a reliable delimiter for filenames.

### `grep`

```bash
grep "ERROR" application.log
grep -i "error" application.log
grep -n "ERROR" application.log
grep -v "INFO" application.log
grep -c "ERROR" application.log
grep -r "TODO" .
grep -E "ERROR|WARNING" application.log
```

### `cut`

```bash
cut -d ':' -f 1 /etc/passwd
cut -d ':' -f 1,3 /etc/passwd
```

`-d` sets delimiter; `-f` selects fields.

### `awk`

```bash
awk '{print $1}' file.txt
awk -F ':' '{print $1, $3}' /etc/passwd
awk '{total += $2} END {print "Total:", total}' data.txt
```

`-F` specifies field separator. `awk` is powerful for column extraction, calculations and reports.

### `sed`

Substitute text:

```bash
sed 's/ERROR/CRITICAL/' file.txt
sed 's/ERROR/CRITICAL/g' file.txt
```

The `g` replaces all matching occurrences per line.

In-place edit:

```bash
sed -i 's/old/new/g' file.txt
```

Use care with in-place edits; keep a backup or test the output first.

### `sort`

```bash
sort file.txt
sort -r file.txt
sort -n numbers.txt
sort -u file.txt
sort -k2 file.txt
```

### `wc`

```bash
wc -l file.txt
wc -w file.txt
wc -c file.txt
```

### `tee`

Display and save output:

```bash
command | tee output.txt
command | tee -a output.txt
```

`-a` appends.

### Log analyzer pattern

```bash
#!/bin/bash

log_file="$1"

if [[ ! -f "$log_file" ]]; then
    echo "Log file not found" >&2
    exit 1
fi

total_lines=$(wc -l < "$log_file")
error_count=$(grep -c "ERROR" "$log_file" || true)
warning_count=$(grep -c "WARNING" "$log_file" || true)

grep -n "ERROR" "$log_file" > error_report.txt || true

printf 'Lines: %s\nErrors: %s\nWarnings: %s\n' \
    "$total_lines" "$error_count" "$warning_count"
```

`grep -c` prints a count but returns nonzero if no matches. `|| true` can make a no-match result acceptable, but don't use it indiscriminately because it can mask other errors. For more robust code, distinguish expected no-match status from actual I/O failure.

---

## 11. Indexed and Associative Arrays

### Indexed array

```bash
services=("nginx" "ssh" "docker" "cron")
```

Indexes normally begin at zero:

```bash
echo "${services[0]}"
echo "${services[2]}"
```

All elements:

```bash
echo "${services[@]}"
```

Number of elements:

```bash
echo "${#services[@]}"
```

Indexes:

```bash
echo "${!services[@]}"
```

Loop:

```bash
for service in "${services[@]}"; do
    echo "$service"
done
```

Loop with indexes:

```bash
for index in "${!services[@]}"; do
    printf '%s: %s\n' "$index" "${services[$index]}"
done
```

Append:

```bash
services+=("redis")
```

Modify:

```bash
services[1]="ssh-server"
```

Remove element:

```bash
unset 'services[2]'
```

Removing an element does not necessarily renumber the remaining indexes.

### Associative array

Associative arrays use named keys.

```bash
declare -A service_status=(
    [nginx]="running"
    [ssh]="running"
    [docker]="stopped"
)
```

Access:

```bash
echo "${service_status[nginx]}"
```

Keys:

```bash
echo "${!service_status[@]}"
```

Values:

```bash
echo "${service_status[@]}"
```

Count:

```bash
echo "${#service_status[@]}"
```

Loop through keys:

```bash
for service in "${!service_status[@]}"; do
    printf '%s: %s\n' "$service" "${service_status[$service]}"
done
```

Associative arrays require Bash 4 or newer. Use Bash, not `sh`, to execute scripts that rely on them.

### Array quoting

Prefer:

```bash
"${array[@]}"
```

It preserves each element as a separate argument, including elements with spaces.

---

## 12. Command-Line Interfaces with `getopts`

`getopts` parses short options such as `-v`, `-f file`, and `-n name`.

### Option specification

```bash
while getopts "vf:n:h" option; do
    case "$option" in
        v)
            verbose=true
            ;;
        f)
            filename="$OPTARG"
            ;;
        n)
            name="$OPTARG"
            ;;
        h)
            echo "Usage: $0 [-v] [-f FILE] [-n NAME] [-h]"
            exit 0
            ;;
    esac
done
```

**Important spelling:** Bash's special variable is `OPTARG` with no space: `${OPTARG}`. `OPTARG` holds the value supplied to an option that requires an argument.

The option string:
- `v`: flag, no value.
- `f:`: `-f` requires a value.
- `n:`: `-n` requires a value.
- `h`: help flag.

### Error handling with a leading colon

```bash
while getopts ":vf:h" option; do
    case "$option" in
        v)
            verbose=true
            ;;
        f)
            filename="${OPTARG}"
            ;;
        h)
            echo "Help"
            exit 0
            ;;
        :)
            echo "Option -${OPTARG} requires an argument." >&2
            exit 1
            ;;
        \?)
            echo "Invalid option -${OPTARG}" >&2
            exit 1
            ;;
    esac
done
```

The leading colon enables silent error handling so the script can handle missing arguments and invalid options itself.

### `OPTIND`

`OPTIND` tracks the argument position that `getopts` is processing. Normally you don't need to modify it. Reset it to `1` when parsing arguments again in a function or another parsing pass.

### Limitations

`getopts` is designed for short options. GNU-style long options (`--output`, `--verbose`) usually require manual parsing or another parser.

---

## 13. Exit Codes, Error Handling, Strict Mode, and Traps

### Exit codes

Every command returns a status:
- `0`: success.
- Nonzero: failure or another condition.

Check the most recent status:

```bash
command
echo "$?"
```

`$?` is overwritten by the next command, so capture it immediately or test directly.

### `exit`

```bash
exit 0
exit 1
```

Terminates the script and returns a status to the caller.

### Prefer direct command checks

```bash
if mkdir backups; then
    echo "Created"
else
    echo "Failed" >&2
    exit 1
fi
```

Instead of running a command, executing several other commands, and then checking `$?`.

### `&&` and `||`

```bash
command1 && command2  # run second if first succeeds
command1 || command2  # run second if first fails
```

For complex workflows, `if` often makes error handling easier to understand.

### `set -e`

```bash
set -e
```

Causes Bash to exit when an unhandled command fails in many situations. It has exceptions (for example, commands used in conditions, certain pipelines/functions/compound contexts). It is not a substitute for explicit handling of expected failures.

### `set -u`

```bash
set -u
```

Treats references to unset variables as errors in normal cases. Use defaults for optional values:

```bash
name="${name:-Guest}"
```

### `set -o pipefail`

```bash
set -o pipefail
```

A pipeline returns a failure if a command in the pipeline fails, rather than reporting only the final command's status.

### Common strict-mode starting point

```bash
set -euo pipefail
```

Understand the behavior and exceptions before using it. Explicitly check important operations and expected failure conditions.

### Traps

Syntax:

```bash
trap 'commands' SIGNAL
```

Handle Ctrl+C:

```bash
trap 'echo "Interrupted"; exit 130' INT
```

Handle termination request:

```bash
trap 'echo "Termination requested"; exit 143' TERM
```

### EXIT trap and cleanup

```bash
temp_file=$(mktemp)
trap 'rm -f -- "$temp_file"' EXIT
```

Runs cleanup when the script exits in normal circumstances, including many error exits.

`SIGKILL` cannot be trapped, so cleanup is not guaranteed if a process is forcibly killed.

### Temporary directory pattern

```bash
#!/bin/bash

set -euo pipefail

temp_dir=$(mktemp -d)

cleanup() {
    rm -rf -- "$temp_dir"
}

trap cleanup EXIT

printf 'Temporary data\n' > "$temp_dir/data.txt"
cat "$temp_dir/data.txt"
```

`mktemp` creates a unique temporary path. Be extremely careful with `rm -rf`: verify that the target is the expected temporary directory and don't construct deletion paths from unvalidated input.

---

## 14. Process Management

### What is a process?

A process is a running instance of a program. Each process has a PID (process ID).

### Inspect processes

```bash
ps
ps -f
ps aux
ps -ef
```

Find by name:

```bash
pgrep nginx
pgrep -a nginx
pgrep -af "python app.py"
```

Useful `pgrep` options:
- `-a`: show PID and command.
- `-f`: match full command line.
- `-x`: exact process name.
- `-u USER`: select processes by user.

### Foreground and background

Foreground:

```bash
sleep 30
```

Background:

```bash
sleep 30 &
```

`&` launches the command asynchronously in the background, returning the terminal to the user.

### `$!`

```bash
sleep 30 &
pid=$!
echo "PID: $pid"
```

`$!` is the PID of the most recently started background process.

### Jobs

```bash
jobs
jobs -l
jobs -r
jobs -s
```

`jobs` lists jobs managed by the current shell. Job IDs such as `%1` are shell job identifiers, not PIDs.

### Foreground/background controls

- `fg`: bring a job to foreground.
- `fg %1`: foreground job 1.
- `bg`: resume a stopped job in background.
- `bg %1`: resume job 1.
- `Ctrl+C`: usually sends SIGINT to foreground process.
- `Ctrl+Z`: suspends foreground process.

### `kill` and signals

```bash
kill "$pid"             # default SIGTERM
kill -TERM "$pid"        # graceful termination request
kill -KILL "$pid"        # force termination
kill -STOP "$pid"        # stop
kill -CONT "$pid"        # continue
kill -0 "$pid"           # check existence/permission; does not terminate
```

Common signals:
- `SIGINT` (2): interrupt.
- `SIGTERM` (15): graceful termination request.
- `SIGKILL` (9): force kill; cannot be caught.
- `SIGSTOP`: stop; cannot be caught.
- `SIGCONT`: resume.

Try graceful termination before `SIGKILL`.

### `wait`

```bash
sleep 3 &
pid=$!

if wait "$pid"; then
    echo "Task succeeded"
else
    echo "Task failed"
fi
```

Wait for background tasks and retrieve their exit status.

```bash
wait
```

Waits for jobs launched by the current shell.

### Concurrent work

```bash
task_a &
pid_a=$!

task_b &
pid_b=$!

status_a=0
status_b=0

wait "$pid_a" || status_a=$?
wait "$pid_b" || status_b=$?

if (( status_a == 0 && status_b == 0 )); then
    echo "Both tasks succeeded"
else
    echo "At least one failed"
fi
```

Parallel work can save time if tasks are independent. Collect each status explicitly if you need all results.

### `nohup`

```bash
nohup python app.py > app.log 2>&1 &
```

`nohup` makes the process ignore SIGHUP. It can help a process survive terminal disconnection, but it is not a service manager.

For persistent services, prefer `systemd`. For interactive sessions that need detaching/reattaching, `tmux` is useful.

### `pkill`

```bash
pkill -TERM -x nginx
```

Sends signals to matching processes. Be careful with broad patterns; identify the target before terminating processes.

---

## 15. Environment Variables, PATH, and Bash Configuration

### Shell variable vs environment variable

```bash
project="AdviceAPI"
export language="Python"
```

A shell variable exists in the current shell. An exported variable is passed in the environment to child processes.

Example parent:

```bash
project="AdviceAPI"
export language="Python"
bash child.sh
```

Child:

```bash
echo "${project:-Not inherited}"
echo "${language:-Not inherited}"
```

The exported variable is inherited; the non-exported variable is not.

Child processes cannot automatically change the parent's environment.

### Environment commands

```bash
env
printenv
printenv APP_ENV
unset APP_ENV
```

Temporary variable for one command:

```bash
APP_ENV=development python app.py
```

### PATH

`PATH` is a colon-separated list of directories searched for commands.

```bash
echo "$PATH"
command -v python
type python
```

Add a directory temporarily:

```bash
export PATH="$HOME/scripts:$PATH"
```

Preserve the existing PATH. Avoid replacing it with only your new directory.

### Bash configuration

Common files:
- `~/.bashrc`: interactive Bash configuration, aliases, functions, prompt, shell settings.
- `~/.profile`: login-session environment setup; may load `.bashrc` in Ubuntu's default configuration.
- `/etc/profile`: system-wide login shell configuration.
- `/etc/bash.bashrc`: system-wide interactive Bash configuration on Ubuntu.

Examples in `~/.bashrc`:

```bash
alias ll='ls -lah'
alias gs='git status'
alias dps='docker ps'

export PATH="$HOME/scripts:$PATH"
export EDITOR="nano"
```

Reload current shell:

```bash
source ~/.bashrc
```

or:

```bash
. ~/.bashrc
```

`source` runs the file in the current shell. Executing it in a separate Bash process won't persist changes into the parent shell.

### Aliases

```bash
alias ll='ls -lah'
unalias ll
```

Aliases are primarily for interactive convenience. Avoid relying on aliases in scripts; use explicit commands.

### Functions in shell config

```bash
mkcd() {
    mkdir -p "$1" && cd "$1"
}
```

This works as an interactive shell function because it executes in the current shell. A child script cannot permanently change the parent's current directory.

### Useful variables

| Variable | Meaning |
|---|---|
| `$HOME` | Home directory |
| `$USER` | Username, commonly set |
| `$PWD` | Current directory |
| `$OLDPWD` | Previous directory |
| `$PATH` | Executable search path |
| `$SHELL` | Configured login shell path; not necessarily the currently running shell |
| `$HOSTNAME` | Hostname, commonly available |
| `$LANG` | Locale |
| `$EDITOR` | Preferred editor if configured |
| `$TERM` | Terminal type |
| `$BASH_VERSION` | Bash version |
| `$$` | Current shell PID |
| `$PPID` | Parent process PID |
| `$?` | Last command status |
| `$!` | Most recent background PID |
| `${BASH_SOURCE[0]}` | Current source/script path in Bash context |

### Script's own directory

```bash
script_dir="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
```

Use this to locate files relative to the script rather than the current working directory:

```bash
config_file="$script_dir/config/settings.conf"
```

### Pass configuration to Python

Bash:

```bash
export APP_ENV="development"
export DATABASE_HOST="localhost"
python app.py
```

Python:

```python
import os

app_env = os.getenv("APP_ENV", "production")
db_host = os.getenv("DATABASE_HOST", "localhost")
```

Environment variables are convenient configuration, but are not by themselves a secure secret-management mechanism.

---

## 16. Debugging and Testing

### Syntax check

```bash
bash -n script.sh
```

Checks syntax without executing the script.

### Trace execution

```bash
bash -x script.sh
```

Or inside the script:

```bash
set -x
```

Turn tracing off:

```bash
set +x
```

Tracing can reveal secrets. Avoid it or redact output when handling credentials/tokens.

### Print diagnostic information

```bash
printf 'Variable: <%s>\n' "$variable"
declare -p variable
```

For arrays:

```bash
declare -p services
```

### Check command availability

```bash
command -v docker >/dev/null 2>&1
```

### Validate script inputs

- Validate argument count.
- Validate expected types (for example, integers).
- Check paths and permissions.
- Quote variable expansions.
- Check important command statuses.
- Test error conditions, not only successful execution.

---

## 17. Safety and Good Practices

1. **Quote variable expansions:** `"$file"`, `"${array[@]}"`.
2. **Validate input:** count, format, paths, options, expected values.
3. **Prefer `[[ ... ]]` in Bash conditions:** clearer and less prone to word-splitting/globbing pitfalls than `[ ... ]`.
4. **Use `printf` for structured output:** more predictable than `echo`.
5. **Use direct status checks:** `if command; then` rather than checking `$?` after unrelated commands.
6. **Understand strict mode:** `set -euo pipefail` is helpful but not infallible.
7. **Handle expected failures:** missing grep matches, optional files, service checks, etc.
8. **Use temporary files safely:** `mktemp`, and cleanup via `trap`.
9. **Treat destructive commands carefully:** `rm -rf`, recursive permissions, disk operations, service termination.
10. **Avoid broad process matching:** inspect `pgrep` output before `pkill`.
11. **Don't use `SIGKILL` unless necessary:** graceful termination is preferable.
12. **Preserve PATH:** prepend/append directories instead of replacing it accidentally.
13. **Don't rely on interactive aliases in scripts.**
14. **Use Bash explicitly:** scripts using Bash-specific features should have a Bash shebang and be run with Bash, not `sh`.
15. **Avoid secrets in logs and shell traces.**
16. **Use meaningful names and functions:** make scripts readable and maintainable.
17. **Use absolute paths when appropriate in scheduled jobs:** cron has a limited environment.
18. **Test in a safe environment before automating destructive operations.**
19. **Use `--` where supported to mark the end of options when passing potentially option-like filenames.**
20. **Make failure visible:** return nonzero status when the overall task fails.

---

## 18. Useful Command Cheat Sheet

### Files and directories

```bash
pwd
ls -lah
touch file
mkdir -p path
cp source destination
mv old new
rm file
find . -type f -name "*.py"
```

### Text processing

```bash
grep -n "ERROR" file
cut -d ':' -f 1 file
awk '{print $1}' file
sed 's/old/new/g' file
sort file
uniq file
wc -l file
tee output.txt
```

### System information

```bash
whoami
hostname
date
uname -a
df -h
free -h
uptime
```

### Process management

```bash
ps aux
pgrep -af python
jobs -l
sleep 10 &
pid=$!
wait "$pid"
kill -TERM "$pid"
```

### Bash debugging

```bash
bash -n script.sh
bash -x script.sh
```

### Environment

```bash
echo "$PATH"
env
printenv
command -v python
type python
source ~/.bashrc
```

### CLI parsing

```bash
while getopts ":vf:h" option; do
    case "$option" in
        v) verbose=true ;;
        f) filename="${OPTARG}" ;;
        h) echo "Help"; exit 0 ;;
        :) echo "Missing argument"; exit 1 ;;
        \?) echo "Invalid option"; exit 1 ;;
    esac
done
```

---

## 19. Reusable Script Template

This template includes a Bash shebang, strict-mode starting point, argument validation, functions, logging to stderr, and a main function.

```bash
#!/bin/bash

set -euo pipefail

usage() {
    cat <<EOF
Usage: $0 DIRECTORY

Description:
    Example Bash utility.

Arguments:
    DIRECTORY    Directory to process

Options:
    -h           Show this help
EOF
}

log() {
    printf '[INFO] %s\n' "$*" >&2
}

error() {
    printf '[ERROR] %s\n' "$*" >&2
}

main() {
    if [[ "$#" -ne 1 ]]; then
        usage >&2
        exit 1
    fi

    local directory="$1"

    if [[ ! -d "$directory" ]]; then
        error "Directory does not exist: $directory"
        exit 1
    fi

    log "Processing: $directory"

    # Implement task here.

    log "Finished."
}

main "$@"
```

This is a starting template, not a guarantee that strict mode will automatically handle every error. Explicitly check critical operations and expected failure conditions.

---

## 20. Practice Projects and Next Steps

The best way to consolidate Bash is to build progressively more useful tools.

### Project 1 — System Information Report

Collect:
- Hostname.
- Username.
- Current date.
- OS/kernel information.
- CPU information.
- Memory.
- Disk usage.

Use functions and `printf`. Add an option to save to a file.

### Project 2 — Log Analyzer

- Accept a log filename.
- Validate it.
- Count errors and warnings.
- Show matching lines.
- Save a report.
- Handle logs without matches gracefully.

Use `grep`, `awk`, `sort`, `wc`, and redirection.

### Project 3 — Backup Tool

- Validate source.
- Create timestamped archive.
- Store backups in a configured directory.
- Check archive creation.
- Clean temporary files.
- Report size.
- Optionally implement retention (remove old backups) with careful validation.

### Project 4 — Linux System Monitor

- CPU, memory and disk checks.
- Monitor selected services.
- Alert when thresholds are exceeded.
- Add `getopts` options.
- Save logs.
- Handle interruption.
- Consider concurrent data collection.

### Project 5 — Docker Helper

- Verify Docker command and daemon.
- List containers.
- Inspect a selected container.
- Show logs.
- Start/stop/restart selected containers.
- Validate container names.
- Use clear help and error messages.

### Project 6 — Bash + Python Automation

Use Bash for:
- Calling Linux commands.
- Checking services.
- Managing files.
- Setting environment variables.
- Orchestrating steps.

Use Python for:
- Complex logic.
- Data analysis.
- APIs.
- Structured JSON.
- Advanced reporting.

Example architecture:

```text
Bash orchestration
      |
      +--> collect Linux metrics
      |
      +--> check services
      |
      +--> save raw output
      |
      +--> invoke Python
               |
               +--> parse and analyze data
               +--> generate structured report
               +--> return result/status
      |
      +--> Bash handles notification / follow-up
```

### Suggested next learning topics

After these Bash lessons, deepen the practical side:
- Cron scheduling.
- `systemd` services and timers.
- Shell scripting security.
- Bash testing (for example, Bats).
- `jq` for JSON.
- SSH automation.
- Docker automation.
- Bash/Python integration.
- CI/CD script design.

---

## Final checklist: Bash skills covered in Lessons 1–14

- [x] What Bash is and how it differs from the Linux kernel
- [x] Shebangs, scripts, permissions, and execution
- [x] Variables, quoting, expansion, and environment basics
- [x] `echo`, `printf`, `read`, stdin/stdout/stderr
- [x] Command substitution and redirection
- [x] Positional parameters and argument validation
- [x] Arithmetic and comparisons
- [x] `if`, `elif`, `else`, `case`
- [x] `for`, `while`, `until`, `break`, `continue`
- [x] Functions, local scope, and return statuses
- [x] Files, directories, `find`, pipes, and text processing
- [x] Indexed and associative arrays
- [x] `getopts`, flags, options, and help
- [x] Exit codes and error handling
- [x] `set -e`, `set -u`, `pipefail` and their limitations
- [x] `trap`, signals, and cleanup
- [x] PIDs, jobs, foreground/background processes
- [x] `wait`, `kill`, `pgrep`, `ps`, `nohup`
- [x] Concurrent background tasks
- [x] Environment inheritance and `export`
- [x] PATH and command lookup
- [x] `.bashrc`, `.profile`, aliases, and functions
- [x] Script-relative paths with `BASH_SOURCE`
- [x] Debugging with `bash -n` and `bash -x`
- [x] Safe scripting practices and reusable patterns

**The goal now is to turn these concepts into working automation tools, not merely memorize syntax.**

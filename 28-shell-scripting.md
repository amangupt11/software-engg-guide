# 🐚 Shell Scripting — Production Engineering Guide

> **Production-grade shell scripting reference for Bash and Unix/Linux environments.**
>
> Focus: reliable automation, maintainability, portability, error handling, testing, security, CI/CD, deployment, operations, and production scripting practices.

---

# 1. Shell Scripting Scope

Shell scripting is best suited for:

```text
Automation
System administration
Build scripts
Deployment scripts
CI/CD
File operations
Process orchestration
Environment setup
Operational tooling
Developer workflows
```

Use a general-purpose language instead when the problem becomes application-like:

```text
Complex business logic
Large applications
Complex data structures
High-performance computation
Advanced concurrency
Large test suites
Long-lived application services
```

---

# 2. Shell vs Bash

```text
Shell
├── sh
├── Bash
├── Zsh
├── Dash
├── Ksh
└── Fish
```

Bash is widely used for Linux automation, but Bash-specific features are not necessarily portable to POSIX `sh`.

Therefore always declare the intended shell.

Bash:

```bash
#!/usr/bin/env bash
```

POSIX shell:

```sh
#!/bin/sh
```

---

# 3. Shebang

Recommended Bash:

```bash
#!/usr/bin/env bash
```

Make executable:

```bash
chmod +x script.sh
```

Run:

```bash
./script.sh
```

Or:

```bash
bash script.sh
```

---

# 4. Script Structure

Recommended production structure:

```text
#!/usr/bin/env bash

Strict-mode configuration
Constants / defaults
Logging
Cleanup
Validation
Functions
Main workflow
main "$@"
```

Example:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly APP_NAME="myapp"

log() {
    printf '[%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$*"
}

validate() {
    :
}

run() {
    :
}

main() {
    validate
    run
}

main "$@"
```

---

# 5. Strict Mode

Common production baseline:

```bash
set -Eeuo pipefail
```

Equivalent concepts:

```bash
set -e
set -u
set -o pipefail
```

`-E` enables ERR-trap inheritance in relevant Bash contexts.

Understand these options before adopting them blindly; `set -e` has documented exceptions.

Official Bash reference:

https://www.gnu.org/software/bash/manual/bash.html

---

# 6. Exit Codes

Success:

```bash
exit 0
```

Failure:

```bash
exit 1
```

Last command:

```bash
echo "$?"
```

Function:

```bash
return 1
```

Check:

```bash
if command; then
    echo "Success"
else
    echo "Failure"
fi
```

Production principle:

```text
0     = success
non-0 = failure
```

Use documented exit-code contracts for individual commands.

---

# 7. Arguments

Script:

```bash
#!/usr/bin/env bash

echo "Script: $0"
echo "First: $1"
echo "Second: $2"
echo "Count: $#"
```

Run:

```bash
./script.sh hello world
```

All arguments:

```bash
"$@"
```

Important:

```bash
"$@"
```

preserves argument boundaries.

Avoid:

```bash
$@
```

when arguments may contain spaces or special characters.

---

# 8. Default Arguments

```bash
NAME="${1:-Aman}"
ENVIRONMENT="${2:-development}"
```

Required:

```bash
: "${API_URL:?API_URL is required}"
```

---

# 9. Variables

Correct:

```bash
NAME="Aman"
PORT=8080
```

Incorrect:

```bash
NAME = "Aman"
```

Read:

```bash
echo "$NAME"
```

Prefer quoting:

```bash
echo "$NAME"
```

instead of:

```bash
echo $NAME
```

---

# 10. Constants

Bash does not provide immutable variables in the same sense as strongly typed languages, but `readonly` can enforce read-only shell variables:

```bash
readonly APP_NAME="myapp"
readonly PORT=8080
```

Attempting to modify:

```bash
PORT=9090
```

will fail.

---

# 11. Environment Variables

Set:

```bash
export APP_ENV=production
```

Read:

```bash
echo "$APP_ENV"
```

Check:

```bash
printenv APP_ENV
```

Use defaults:

```bash
APP_ENV="${APP_ENV:-development}"
```

Never store production secrets directly in source-controlled scripts.

---

# 12. Quoting

Single quotes:

```bash
'text'
```

Double quotes:

```bash
"text"
```

Variable expansion occurs inside double quotes:

```bash
echo "$NAME"
```

Not inside single quotes:

```bash
echo '$NAME'
```

Production rule:

> Quote variable expansions unless you intentionally need word splitting or glob expansion.

---

# 13. Command Substitution

Preferred:

```bash
DATE="$(date)"
```

Example:

```bash
CURRENT_DIR="$(pwd)"
```

Avoid legacy backticks:

```bash
`date`
```

---

# 14. Arithmetic

```bash
COUNT=10
TOTAL=$((COUNT + 5))
```

Increment:

```bash
((COUNT++))
```

Condition:

```bash
if (( COUNT > 10 )); then
    echo "Large"
fi
```

---

# 15. Strings

```bash
NAME="Aman Gupta"
```

Length:

```bash
echo "${#NAME}"
```

Substring:

```bash
echo "${NAME:0:4}"
```

Default:

```bash
VALUE="${VALUE:-default}"
```

Required:

```bash
: "${VALUE:?VALUE is required}"
```

---

# 16. Arrays

```bash
SERVICES=("api" "web" "worker")
```

Access:

```bash
echo "${SERVICES[0]}"
```

All:

```bash
printf '%s\n' "${SERVICES[@]}"
```

Count:

```bash
echo "${#SERVICES[@]}"
```

Loop:

```bash
for service in "${SERVICES[@]}"; do
    echo "$service"
done
```

---

# 17. Associative Arrays

Bash:

```bash
declare -A CONFIG

CONFIG[environment]="production"
CONFIG[port]="8080"
```

Read:

```bash
echo "${CONFIG[environment]}"
```

Requires Bash versions supporting associative arrays.

---

# 18. Functions

```bash
log() {
    echo "Application started"
}
```

Call:

```bash
log
```

Arguments:

```bash
greet() {
    local name="$1"
    echo "Hello $name"
}

greet "Aman"
```

Use `local` for function-local variables.

---

# 19. Function Return Values

Success:

```bash
validate() {
    return 0
}
```

Failure:

```bash
validate() {
    return 1
}
```

Use command output separately:

```bash
get_version() {
    printf '%s\n' "1.0.0"
}

VERSION="$(get_version)"
```

---

# 20. Conditions

File:

```bash
if [[ -f "$FILE" ]]; then
    echo "File exists"
fi
```

Directory:

```bash
if [[ -d "$DIR" ]]; then
    echo "Directory exists"
fi
```

String:

```bash
if [[ "$ENV" == "production" ]]; then
    echo "Production"
fi
```

Numeric:

```bash
if (( PORT > 1024 )); then
    echo "Non-privileged port"
fi
```

---

# 21. File Test Operators

Common:

```text
-f  regular file
-d  directory
-e  exists
-r  readable
-w  writable
-x  executable
-s  non-empty
-L  symbolic link
```

Example:

```bash
if [[ -r "$CONFIG_FILE" ]]; then
    echo "Readable"
fi
```

---

# 22. String Tests

```bash
if [[ -z "$VALUE" ]]; then
    echo "Empty"
fi
```

Not empty:

```bash
if [[ -n "$VALUE" ]]; then
    echo "Has value"
fi
```

Equality:

```bash
[[ "$ENV" == "production" ]]
```

Pattern matching:

```bash
[[ "$FILE" == *.log ]]
```

Regex:

```bash
[[ "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]
```

---

# 23. Logical Operators

```bash
if [[ "$ENV" == "production" && "$READY" == "true" ]]; then
    echo "Ready"
fi
```

OR:

```bash
if [[ "$ENV" == "production" || "$ENV" == "staging" ]]; then
    echo "Deployable"
fi
```

Negation:

```bash
if [[ ! -f "$FILE" ]]; then
    echo "Missing"
fi
```

---

# 24. Case

```bash
case "$ENV" in
    development)
        echo "Development"
        ;;
    staging)
        echo "Staging"
        ;;
    production)
        echo "Production"
        ;;
    *)
        echo "Invalid environment" >&2
        exit 1
        ;;
esac
```

---

# 25. For Loops

Array:

```bash
for item in "${ITEMS[@]}"; do
    echo "$item"
done
```

Files:

```bash
for file in ./*.log; do
    [[ -e "$file" ]] || continue
    echo "$file"
done
```

Numeric:

```bash
for ((i=0; i<10; i++)); do
    echo "$i"
done
```

---

# 26. While Loops

```bash
while [[ "$COUNT" -lt 10 ]]; do
    echo "$COUNT"
    ((COUNT++))
done
```

Read file safely:

```bash
while IFS= read -r line; do
    printf '%s\n' "$line"
done < input.txt
```

---

# 27. Until

```bash
until systemctl is-active --quiet myapp; do
    sleep 2
done
```

Add timeout logic in production to avoid infinite waits.

---

# 28. Loop Control

Continue:

```bash
continue
```

Break:

```bash
break
```

Example:

```bash
for file in *.log; do
    [[ -f "$file" ]] || continue

    if grep -q "critical" "$file"; then
        break
    fi
done
```

---

# 29. Pipes

```bash
ps aux | grep nginx
```

Production example:

```bash
journalctl -u myapp |
    grep -i error |
    tail -n 50
```

Use structured tools where available instead of brittle text parsing.

---

# 30. Redirection

Overwrite:

```bash
command > output.log
```

Append:

```bash
command >> output.log
```

stderr:

```bash
command 2> error.log
```

Both:

```bash
command > output.log 2>&1
```

Bash shorthand:

```bash
command &> output.log
```

---

# 31. Here Documents

```bash
cat > config.conf <<'EOF'
APP_ENV=production
PORT=8080
EOF
```

Quoted `EOF` prevents variable expansion.

Unquoted:

```bash
cat <<EOF
Environment: $APP_ENV
EOF
```

---

# 32. Here Strings

```bash
grep "error" <<< "$LOG_LINE"
```

---

# 33. Temporary Files

Use:

```bash
TMP_FILE="$(mktemp)"
trap 'rm -f "$TMP_FILE"' EXIT
```

Avoid:

```bash
TMP_FILE="/tmp/myapp.tmp"
```

because predictable temporary filenames can create security and collision problems.

---

# 34. Cleanup with trap

```bash
cleanup() {
    rm -f "$TMP_FILE"
}

trap cleanup EXIT
```

Signals:

```bash
trap 'exit 130' INT
trap 'exit 143' TERM
```

Production scripts should clean temporary resources and handle interruption appropriately.

---

# 35. ERR Trap

```bash
set -E
trap 'echo "Error on line $LINENO" >&2' ERR
```

Better:

```bash
trap 'rc=$?; printf "ERROR: line=%s status=%s command=%s\n" "$LINENO" "$rc" "$BASH_COMMAND" >&2' ERR
```

Understand Bash `ERR` trap semantics before relying on it for every possible failure.

---

# 36. Logging

Simple:

```bash
log() {
    printf '[%s] %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$*"
}
```

Usage:

```bash
log "Deployment started"
```

stderr:

```bash
log_error() {
    printf '[%s] ERROR: %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$*" >&2
}
```

---

# 37. Log Levels

Recommended:

```text
DEBUG
INFO
WARN
ERROR
```

Example:

```bash
log_info() {
    printf '[INFO] %s\n' "$*"
}

log_warn() {
    printf '[WARN] %s\n' "$*" >&2
}

log_error() {
    printf '[ERROR] %s\n' "$*" >&2
}
```

Never log:

```text
Passwords
API keys
Access tokens
Private keys
Session secrets
Sensitive personal data
```

---

# 38. Input Validation

Required:

```bash
: "${ENVIRONMENT:?ENVIRONMENT is required}"
```

Allow-list:

```bash
case "$ENVIRONMENT" in
    development|staging|production)
        ;;
    *)
        echo "Invalid environment" >&2
        exit 1
        ;;
esac
```

Numeric:

```bash
[[ "$PORT" =~ ^[0-9]+$ ]] || {
    echo "Invalid port" >&2
    exit 1
}
```

---

# 39. Path Validation

```bash
if [[ ! -d "$APP_DIR" ]]; then
    echo "Application directory does not exist" >&2
    exit 1
fi
```

Dangerous:

```bash
rm -rf "$TARGET"
```

Before destructive operations:

```bash
[[ -n "$TARGET" ]] || exit 1
[[ "$TARGET" != "/" ]] || exit 1
```

Prefer explicit allow-listed paths for destructive automation.

---

# 40. Safe Command Execution

Quote:

```bash
cp -- "$SOURCE" "$DESTINATION"
```

Delete:

```bash
rm -- "$FILE"
```

Use `--` where supported to prevent filenames beginning with `-` from being interpreted as options.

---

# 41. Avoid `eval`

Avoid:

```bash
eval "$USER_INPUT"
```

`eval` can turn data into executable shell code.

Prefer arrays:

```bash
COMMAND=(curl -fsS "$URL")

"${COMMAND[@]}"
```

This preserves argument boundaries.

---

# 42. Command Arrays

Bad:

```bash
COMMAND="curl -H 'Authorization: Bearer $TOKEN' $URL"
$COMMAND
```

Better:

```bash
COMMAND=(
    curl
    -fsS
    -H "Authorization: Bearer $TOKEN"
    "$URL"
)

"${COMMAND[@]}"
```

---

# 43. `read`

```bash
read -r NAME
```

Prompt:

```bash
read -r -p "Environment: " ENV
```

Secret:

```bash
read -r -s -p "Password: " PASSWORD
printf '\n'
```

Prefer an approved secret manager instead of interactive passwords in automation.

---

# 44. `getopts`

Production-style options:

```bash
while getopts ":e:f:" opt; do
    case "$opt" in
        e) ENV="$OPTARG" ;;
        f) FILE="$OPTARG" ;;
        *)
            echo "Usage: $0 -e environment -f file" >&2
            exit 2
            ;;
    esac
done
```

Run:

```bash
./deploy.sh -e production -f config.yml
```

---

# 45. Long Options

Bash does not provide a native universal `getopts` long-option interface.

For long options, consider:

```text
Manual argument parsing
getopt
A higher-level CLI framework
```

Validate the parser behavior on the target platform.

---

# 46. `printf`

Prefer:

```bash
printf '%s\n' "$NAME"
```

over:

```bash
echo "$NAME"
```

for predictable formatting.

Formatted:

```bash
printf 'Port: %d\n' "$PORT"
```

---

# 47. Text Processing

Common tools:

```text
grep
sed
awk
cut
sort
uniq
tr
head
tail
wc
xargs
```

Example:

```bash
grep -i "error" app.log |
    sort |
    uniq -c |
    sort -nr
```

---

# 48. `sed`

Replace:

```bash
sed 's/development/production/g' config.txt
```

Print lines:

```bash
sed -n '1,20p' config.txt
```

In-place editing differs by platform; test scripts when portability matters.

---

# 49. `awk`

Fields:

```bash
awk '{print $1}' file.txt
```

CSV-like:

```bash
awk -F',' '{print $1, $3}' data.csv
```

Use robust parsers for complex CSV rather than assuming simple comma-separated text.

---

# 50. `cut`

```bash
cut -d':' -f1 /etc/passwd
```

Characters:

```bash
cut -c1-10 file.txt
```

---

# 51. `sort` / `uniq`

```bash
sort names.txt
```

Unique:

```bash
sort names.txt | uniq
```

Counts:

```bash
sort names.txt | uniq -c
```

---

# 52. `xargs`

```bash
printf '%s\n' *.tmp | xargs rm --
```

Safer with unusual filenames:

```bash
find . -type f -name '*.tmp' -print0 |
    xargs -0 rm --
```

Always consider whether the command can handle zero input.

---

# 53. `find`

```bash
find . -type f -name '*.log'
```

Large files:

```bash
find /var -type f -size +500M
```

Recent:

```bash
find . -type f -mtime -1
```

Execute:

```bash
find . -type f -name '*.log' -exec gzip -- {} \;
```

---

# 54. `grep` in Scripts

```bash
if grep -q "healthy" health.txt; then
    echo "Healthy"
fi
```

Recursive:

```bash
grep -R --line-number "TODO" src/
```

Case insensitive:

```bash
grep -Ri "error" logs/
```

---

# 55. JSON

Avoid parsing JSON with:

```bash
grep
sed
awk
```

Prefer:

```bash
jq
```

Example:

```bash
STATUS="$(jq -r '.status' response.json)"
```

API:

```bash
curl -fsS "$API_URL" | jq -r '.status'
```

---

# 56. YAML

Do not treat YAML as plain text when structure matters.

Use an appropriate YAML parser/tool, such as:

```text
yq
```

Validate:

```bash
yq '.' config.yaml
```

---

# 57. HTTP Automation

```bash
curl -fsS https://example.com/health
```

Fail on HTTP errors:

```bash
curl -f https://example.com/health
```

Timeout:

```bash
curl --connect-timeout 5 --max-time 30 -fsS "$URL"
```

POST:

```bash
curl -fsS \
    -H "Content-Type: application/json" \
    -d '{"name":"Aman"}' \
    "$API_URL"
```

---

# 58. Retry Logic

Simple:

```bash
for attempt in 1 2 3 4 5; do
    if curl -fsS "$URL"; then
        exit 0
    fi

    sleep $((attempt * 2))
done

exit 1
```

Production retry design should consider:

```text
Maximum attempts
Timeout
Backoff
Jitter
Retryable errors
Non-retryable errors
Idempotency
Rate limits
Total execution deadline
```

---

# 59. Timeouts

Avoid infinite waits.

Example:

```bash
timeout 60s ./long-running-command.sh
```

Check:

```bash
if timeout 60s ./command.sh; then
    echo "Completed"
else
    echo "Timed out or failed" >&2
    exit 1
fi
```

---

# 60. Process Management

Start:

```bash
./app &
```

PID:

```bash
PID=$!
```

Wait:

```bash
wait "$PID"
```

Check:

```bash
kill -0 "$PID"
```

For production services, prefer service managers such as `systemd`.

---

# 61. Signals

Common:

```text
SIGINT
SIGTERM
SIGKILL
SIGHUP
SIGQUIT
```

Graceful:

```bash
kill -TERM "$PID"
```

Force:

```bash
kill -KILL "$PID"
```

Prefer graceful termination before `SIGKILL`.

---

# 62. Locking

Prevent duplicate executions with an appropriate locking mechanism.

Example:

```bash
flock -n /var/run/myapp.lock ./job.sh
```

This helps prevent:

```text
Duplicate cron jobs
Concurrent deployment
Overlapping backups
Race conditions
```

Validate lock paths and permissions in production.

---

# 63. Concurrency

Background jobs:

```bash
job1 &
job2 &
wait
```

Parallel execution:

```bash
for item in "${ITEMS[@]}"; do
    process "$item" &
done

wait
```

Do not blindly parallelize production operations.

Consider:

```text
CPU
Memory
I/O
Network
Rate limits
Ordering
Shared state
Failure handling
```

---

# 64. Temporary Workspace

Production pattern:

```bash
TMP_DIR="$(mktemp -d)"

cleanup() {
    rm -rf -- "$TMP_DIR"
}

trap cleanup EXIT
```

Use secure temporary directories and clean them on exit.

---

# 65. Configuration

Separate:

```text
Code
Configuration
Secrets
Environment
```

Example:

```bash
CONFIG_FILE="${CONFIG_FILE:-/etc/myapp/config}"
```

Validate:

```bash
[[ -r "$CONFIG_FILE" ]] || {
    echo "Configuration not readable" >&2
    exit 1
}
```

---

# 66. Secrets

Never commit:

```text
PASSWORD=...
API_KEY=...
TOKEN=...
PRIVATE_KEY=...
```

Prefer:

```text
Cloud secret managers
Vault systems
CI/CD secret stores
OS credential mechanisms
Protected environment variables
```

Do not print secrets:

```bash
set -x
```

can expose sensitive values.

---

# 67. `set -x` Security Warning

Debug:

```bash
set -x
```

Disable:

```bash
set +x
```

Never enable tracing around commands containing secrets unless output is properly controlled.

---

# 68. ShellCheck

Install according to your platform.

Run:

```bash
shellcheck script.sh
```

Official:

https://www.shellcheck.net/

ShellCheck should be part of CI for production shell scripts.

---

# 69. Formatting

A common formatter is:

```text
shfmt
```

Example:

```bash
shfmt -w script.sh
```

Official:

https://github.com/mvdan/sh

Use a consistent formatting policy across the repository.

---

# 70. Testing

A common Bash testing framework:

```text
Bats
```

Official:

https://bats-core.readthedocs.io/

Example concept:

```bash
@test "application starts" {
    run ./app.sh
    [ "$status" -eq 0 ]
}
```

Use tests for:

```text
Argument validation
Configuration
Functions
Exit codes
Deployment logic
Failure handling
Idempotency
```

---

# 71. Static Quality Pipeline

Recommended:

```text
shfmt
↓
ShellCheck
↓
Bats
↓
Security checks
↓
CI
```

Example:

```bash
shfmt -d .
shellcheck scripts/*.sh
bats tests/
```

---

# 72. CI/CD Shell Script

Example:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

main() {
    echo "Installing dependencies"
    npm ci

    echo "Running tests"
    npm test

    echo "Building"
    npm run build
}

main "$@"
```

Use the project's official package manager and lockfile.

---

# 73. Deployment Script

Production pattern:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly APP_NAME="myapp"
readonly RELEASE_DIR="/opt/myapp/releases"

validate() {
    [[ -d "$RELEASE_DIR" ]] || {
        echo "Release directory missing" >&2
        return 1
    }
}

deploy() {
    echo "Deploying $APP_NAME"
}

verify() {
    systemctl is-active --quiet "$APP_NAME"
}

main() {
    validate
    deploy
    verify
}

main "$@"
```

---

# 74. Idempotency

Bad:

```bash
echo "entry" >> /etc/myapp.conf
```

Every execution adds another entry.

Better:

```bash
grep -qxF "entry" /etc/myapp.conf ||
    echo "entry" >> /etc/myapp.conf
```

For complex configuration, use configuration-management tools instead of growing shell scripts indefinitely.

---

# 75. Atomic File Updates

Avoid partially written production configuration.

Pattern:

```bash
TMP_FILE="$(mktemp)"

generate_config > "$TMP_FILE"

mv -- "$TMP_FILE" /etc/myapp/config
```

For critical files, also consider:

```text
Correct permissions
fsync requirements
Backup/versioning
Validation before replacement
Rollback
```

---

# 76. Deployment Verification

After deployment:

```bash
systemctl is-active --quiet myapp
```

HTTP:

```bash
curl -fsS http://localhost:8080/health
```

Version:

```bash
curl -fsS http://localhost:8080/version
```

Verify:

```text
Process
Port
Health
Version
Dependencies
Logs
Metrics
```

---

# 77. Rollback

Production deployment should have:

```text
Deploy
↓
Verify
↓
Failure?
↓
Rollback
↓
Verify rollback
```

Example:

```bash
if ! verify; then
    rollback
    verify_rollback
    exit 1
fi
```

Rollback should be tested before it is needed.

---

# 78. systemd Integration

Instead of:

```bash
nohup ./app &
```

prefer:

```text
systemd service
```

Operations:

```bash
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp
sudo systemctl status myapp
sudo systemctl enable myapp
```

Logs:

```bash
journalctl -u myapp
```

---

# 79. Cron Integration

For lightweight scheduled tasks:

```bash
crontab -e
```

Example:

```cron
0 2 * * * /opt/scripts/backup.sh
```

Production requirements:

```text
Absolute paths
Logging
Locking
Timeout
Error handling
Monitoring
Alerting
```

For systemd-based environments, `systemd` timers may provide better service integration.

---

# 80. Shell Portability

Bash-specific:

```bash
[[ ... ]]
(( ... ))
arrays
associative arrays
mapfile
process substitution
```

POSIX shell:

```sh
[ ... ]
case
for
while
functions
```

If portability is required:

```sh
#!/bin/sh
```

and write POSIX-compatible code.

If Bash is required:

```bash
#!/usr/bin/env bash
```

Do not claim a Bash script is POSIX-compatible.

---

# 81. Common Shell Mistakes

Avoid:

```bash
rm -rf "$DIR"/*
```

without validating the variable.

Avoid:

```bash
for file in $(find . -type f)
```

because whitespace/newline handling can break filenames.

Prefer:

```bash
while IFS= read -r -d '' file; do
    ...
done < <(find . -type f -print0)
```

or:

```bash
find . -type f -exec ... {} +
```

---

# 82. Avoid Parsing `ls`

Bad:

```bash
for file in $(ls); do
    ...
done
```

Better:

```bash
for file in ./*; do
    [[ -e "$file" ]] || continue
    ...
done
```

Or use:

```bash
find
```

for recursive file discovery.

---

# 83. Avoid Useless `cat`

Avoid:

```bash
cat file.txt | grep error
```

Prefer:

```bash
grep error file.txt
```

Use pipelines when they improve clarity or are actually required.

---

# 84. Command Existence

```bash
if command -v jq >/dev/null 2>&1; then
    echo "jq installed"
else
    echo "jq is required" >&2
    exit 1
fi
```

Production prerequisite validation should happen before destructive or irreversible work.

---

# 85. Dependency Validation

```bash
required_commands=(
    curl
    jq
    git
)

for command in "${required_commands[@]}"; do
    command -v "$command" >/dev/null 2>&1 || {
        echo "Missing dependency: $command" >&2
        exit 1
    }
done
```

---

# 86. Working Directory

Do not assume the script is executed from its own directory.

Determine script directory:

```bash
SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"
```

Then:

```bash
CONFIG="$SCRIPT_DIR/config/config.yml"
```

This makes scripts less dependent on the caller's current directory.

---

# 87. Shell Options

Inspect:

```bash
set -o
```

Useful:

```bash
set -e
set -u
set -o pipefail
set -E
```

Debug:

```bash
set -x
```

Disable:

```bash
set +x
```

---

# 88. `IFS`

`IFS` controls word splitting.

Safe line reading:

```bash
while IFS= read -r line; do
    printf '%s\n' "$line"
done < file.txt
```

Do not globally modify `IFS` unless you understand the consequences.

---

# 89. Process Substitution

Bash:

```bash
diff <(sort file1.txt) <(sort file2.txt)
```

Useful for comparing command output without creating intermediate files.

---

# 90. Command Substitution vs Process Substitution

Command substitution:

```bash
RESULT="$(command)"
```

Process substitution:

```bash
diff <(command1) <(command2)
```

Use according to the data flow required.

---

# 91. Subshells

Parentheses create a subshell:

```bash
(
    cd /tmp
    pwd
)
```

The parent shell's directory remains unchanged.

Braces execute in the current shell:

```bash
{
    echo "hello"
}
```

---

# 92. Environment Isolation

Run with temporary environment:

```bash
APP_ENV=production ./deploy.sh
```

Multiple:

```bash
APP_ENV=production PORT=8080 ./app.sh
```

This avoids permanently changing the caller's environment.

---

# 93. Secure PATH

For privileged automation:

```text
Do not trust an uncontrolled PATH
```

Use:

```bash
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
export PATH
```

Use absolute executable paths when security requirements justify it.

---

# 94. Umask

Inspect:

```bash
umask
```

Example:

```bash
umask 027
```

This can reduce default permissions for newly created files.

Choose values based on the application's requirements and security policy.

---

# 95. File Descriptor Management

Standard:

```text
0 → stdin
1 → stdout
2 → stderr
```

Redirect:

```bash
command 2>error.log
```

Both:

```bash
command >output.log 2>&1
```

Custom descriptor:

```bash
exec 3>debug.log
printf '%s\n' "debug" >&3
exec 3>&-
```

---

# 96. Production Script Template

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

readonly APP_NAME="myapp"
readonly SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)"

TMP_DIR=""

log() {
    printf '[%s] %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$*"
}

error() {
    printf '[%s] ERROR: %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$*" >&2
}

cleanup() {
    if [[ -n "$TMP_DIR" && -d "$TMP_DIR" ]]; then
        rm -rf -- "$TMP_DIR"
    fi
}

on_error() {
    local rc=$?
    error "line=$LINENO status=$rc command=$BASH_COMMAND"
    exit "$rc"
}

validate() {
    command -v curl >/dev/null 2>&1 ||
        { error "curl is required"; return 1; }

    return 0
}

deploy() {
    log "Deploying $APP_NAME"
}

verify() {
    log "Verifying $APP_NAME"
}

main() {
    trap cleanup EXIT
    trap on_error ERR

    TMP_DIR="$(mktemp -d)"

    validate
    deploy
    verify

    log "Completed successfully"
}

main "$@"
```

---

# 97. Production Checklist

```text
[ ] Correct shebang
[ ] Bash/POSIX requirement documented
[ ] Strict mode understood
[ ] Arguments validated
[ ] Environment validated
[ ] Dependencies checked
[ ] Variables quoted
[ ] Paths validated
[ ] Secrets protected
[ ] No unsafe eval
[ ] Temporary files secure
[ ] Cleanup implemented
[ ] Timeouts configured
[ ] Retries bounded
[ ] Exit codes checked
[ ] Logging implemented
[ ] ShellCheck passes
[ ] Formatter passes
[ ] Tests pass
[ ] CI passes
[ ] Code reviewed
[ ] Rollback defined
[ ] Monitoring/alerting considered
```

---

# 98. CI Quality Gate

Recommended:

```text
Pull Request
     ↓
shfmt
     ↓
ShellCheck
     ↓
Unit Tests
     ↓
Integration Tests
     ↓
Security Checks
     ↓
Code Review
     ↓
Merge
     ↓
Build
     ↓
Deploy
     ↓
Health Check
```

---

# 99. Repository Structure

Recommended:

```text
project/
├── scripts/
│   ├── build.sh
│   ├── test.sh
│   ├── deploy.sh
│   └── backup.sh
├── tests/
│   └── *.bats
├── config/
├── docs/
├── Makefile
└── README.md
```

For larger organizations:

```text
scripts/
├── lib/
├── deployment/
├── infrastructure/
├── development/
└── operations/
```

---

# 100. Official Shell / Linux Sources

## GNU Bash

https://www.gnu.org/software/bash/manual/bash.html

https://www.gnu.org/software/bash/

## POSIX Shell

https://pubs.opengroup.org/onlinepubs/9799919799/utilities/V3_chap02.html

## Ubuntu Documentation

https://docs.ubuntu.com/

## Ubuntu Server

https://ubuntu.com/server/docs/

## ShellCheck

https://www.shellcheck.net/

## Bats

https://bats-core.readthedocs.io/

## shfmt

https://github.com/mvdan/sh

---

# 🏁 Final Production Principle

> **A production shell script should be treated as software, not as a collection of terminal commands.**

Use:

```text
Explicit Shell
     ↓
Validated Inputs
     ↓
Safe Quoting
     ↓
Controlled Execution
     ↓
Error Handling
     ↓
Logging
     ↓
Testing
     ↓
Static Analysis
     ↓
CI/CD
     ↓
Deployment
     ↓
Verification
     ↓
Rollback
```

The goal is not to write the shortest shell script.

The goal is to write automation that is:

```text
Reliable
Maintainable
Secure
Idempotent
Observable
Testable
Portable where required
Recoverable
Production-safe
```

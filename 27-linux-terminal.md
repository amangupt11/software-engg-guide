# 🐧 Linux Terminal — Ubuntu / Bash Production Engineering Guide

> **Production-grade Linux terminal reference focused on Ubuntu, Bash, system administration, software development, DevOps, networking, security, automation, troubleshooting, and production operations.**

> **Scope:** Ubuntu is the primary distribution example, while most commands use standard Linux/POSIX/GNU tooling and therefore apply to many Linux distributions with package-management differences.

Ubuntu maintains official documentation for current LTS releases and its Server documentation specifically covers command-line operation, package management, networking, security, services, storage, and server administration. citeturn0search7turn0search17

---

# 1. Linux Terminal Architecture

```text
Terminal Emulator
       ↓
Shell
       ↓
Bash
       ↓
Linux Commands / GNU Utilities
       ↓
Kernel / System Calls
       ↓
Hardware / Virtual Infrastructure
```

Important distinction:

```text
Terminal → application that displays a shell
Shell    → command interpreter
Bash     → commonly used shell
Command  → program or shell builtin
Script   → executable shell program
```

Bash is both a command interpreter and a programming language with variables, control flow, functions, pipelines, redirection, history, aliases, and job control. citeturn0search0turn0search5

---

# 2. Ubuntu Versions

Check release:

```bash
cat /etc/os-release
```

Ubuntu version:

```bash
lsb_release -a
```

Kernel:

```bash
uname -a
uname -r
```

Architecture:

```bash
uname -m
```

Ubuntu official documentation:

https://ubuntu.com/server/docs/

Ubuntu documentation directory:

https://docs.ubuntu.com/

---

# 3. Terminal Basics

Current directory:

```bash
pwd
```

List:

```bash
ls
```

Detailed:

```bash
ls -la
```

Change directory:

```bash
cd /var/log
```

Home:

```bash
cd ~
```

Previous directory:

```bash
cd -
```

Parent:

```bash
cd ..
```

Root:

```bash
cd /
```

Clear:

```bash
clear
```

---

# 4. Linux Filesystem

Important directories:

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```

Common engineering locations:

```text
/etc       Configuration
/var/log   Logs
/home      User files
/opt       Optional application software
/usr       User-space programs/libraries
/tmp       Temporary files
/run       Runtime state
```

---

# 5. Create Directories

```bash
mkdir projects
```

Nested:

```bash
mkdir -p projects/backend/src
```

Multiple:

```bash
mkdir -p src tests docs
```

---

# 6. Create Files

```bash
touch README.md
```

Create with content:

```bash
echo "Hello" > file.txt
```

Append:

```bash
echo "Another line" >> file.txt
```

Here document:

```bash
cat > config.txt <<'EOF'
APP_ENV=development
PORT=8080
EOF
```

---

# 7. Read Files

```bash
cat file.txt
```

Long files:

```bash
less file.txt
```

First lines:

```bash
head file.txt
head -n 20 file.txt
```

Last lines:

```bash
tail file.txt
tail -n 50 file.txt
```

Follow logs:

```bash
tail -f /var/log/app.log
```

---

# 8. Copy / Move / Rename

Copy:

```bash
cp source.txt destination.txt
```

Directory:

```bash
cp -r src backup
```

Move:

```bash
mv old.txt new.txt
```

Rename:

```bash
mv old-name.txt new-name.txt
```

Move directory:

```bash
mv old-folder /opt/new-folder
```

---

# 9. Delete

File:

```bash
rm file.txt
```

Directory:

```bash
rm -r folder
```

Force:

```bash
rm -rf folder
```

> ⚠️ `rm -rf` is destructive. Always verify the path before execution, especially when running as `root`.

Production rule:

```text
Validate path
↓
Confirm target
↓
Prefer least privilege
↓
Delete
```

---

# 10. Find Files

By name:

```bash
find . -name "*.log"
```

Case-insensitive:

```bash
find . -iname "*.log"
```

Files only:

```bash
find . -type f
```

Directories only:

```bash
find . -type d
```

Modified recently:

```bash
find . -type f -mtime -1
```

Large files:

```bash
find . -type f -size +500M
```

---

# 11. Search Text

`grep`:

```bash
grep "error" app.log
```

Case-insensitive:

```bash
grep -i "error" app.log
```

Recursive:

```bash
grep -R "TODO" .
```

Line numbers:

```bash
grep -n "error" app.log
```

Extended patterns:

```bash
grep -E "error|failed|critical" app.log
```

---

# 12. Locate Commands

```bash
which node
```

More robust command discovery:

```bash
command -v node
```

All matching paths:

```bash
type -a node
```

Search common executable paths:

```bash
whereis node
```

---

# 13. Command Help

Manual:

```bash
man ls
```

Search manual database:

```bash
apropos network
```

Command location:

```bash
type cd
type echo
type ls
```

Bash builtin help:

```bash
help cd
help printf
help set
```

Version:

```bash
ls --version
```

---

# 14. Shell Variables

```bash
NAME="Aman"
COUNT=10
ENABLED=true
```

Read:

```bash
echo "$NAME"
echo "$COUNT"
```

Important:

```bash
NAME="Aman"
```

not:

```bash
NAME = "Aman"
```

Export:

```bash
export APP_ENV=production
```

Read:

```bash
echo "$APP_ENV"
```

---

# 15. Environment Variables

```bash
env
```

```bash
printenv
```

Specific:

```bash
printenv PATH
```

PATH:

```bash
echo "$PATH"
```

Add for current shell:

```bash
export PATH="$HOME/bin:$PATH"
```

Do not blindly modify global PATH in production systems.

---

# 16. Quoting

Single quotes:

```bash
echo '$HOME'
```

Double quotes:

```bash
echo "$HOME"
```

Double quotes expand variables.

Always quote variables when paths/data may contain spaces or special characters:

```bash
rm -- "$file"
cp -- "$source" "$destination"
```

---

# 17. Command Substitution

Modern syntax:

```bash
DATE=$(date)
```

Example:

```bash
CURRENT_DIR=$(pwd)
echo "$CURRENT_DIR"
```

Prefer:

```bash
$(command)
```

over legacy backticks:

```bash
`command`
```

---

# 18. Exit Codes

```bash
echo $?
```

Typical convention:

```text
0     → success
non-0 → failure
```

Script:

```bash
if command; then
    echo "Success"
else
    echo "Failure"
fi
```

Explicit:

```bash
exit 0
exit 1
```

Always verify the specific command's documented exit-code contract.

---

# 19. Command Chaining

Sequential:

```bash
command1 ; command2
```

Only if success:

```bash
command1 && command2
```

Only if failure:

```bash
command1 || command2
```

Example:

```bash
npm install && npm test
```

---

# 20. Pipes

```bash
ps aux | grep nginx
```

Multiple:

```bash
ps aux |
    grep nginx |
    grep -v grep
```

Common pattern:

```text
command
 ↓
filter
 ↓
transform
 ↓
output
```

---

# 21. Redirection

Output overwrite:

```bash
command > output.txt
```

Append:

```bash
command >> output.txt
```

Standard error:

```bash
command 2> error.log
```

Both:

```bash
command > output.log 2>&1
```

Modern shorthand in Bash:

```bash
command &> output.log
```

Input:

```bash
command < input.txt
```

---

# 22. Wildcards / Globbing

```bash
*.log
*.js
src/*.ts
```

Examples:

```bash
ls *.md
rm *.tmp
```

Recursive globbing requires appropriate shell configuration; do not assume `**` behaves recursively in every shell configuration.

---

# 23. Permissions

Inspect:

```bash
ls -l
```

Example:

```text
-rwxr-xr--
```

Conceptually:

```text
owner
group
others
```

Change mode:

```bash
chmod +x script.sh
```

Numeric:

```bash
chmod 755 script.sh
chmod 644 config.json
```

Recursive:

```bash
chmod -R 755 directory
```

> Avoid recursive permission changes unless you understand the required permissions for every file.

---

# 24. Ownership

```bash
ls -l
```

Change owner:

```bash
sudo chown user file.txt
```

Owner + group:

```bash
sudo chown user:group file.txt
```

Recursive:

```bash
sudo chown -R user:group /opt/myapp
```

Use carefully.

---

# 25. sudo

Run as elevated user:

```bash
sudo command
```

Open root shell only when justified:

```bash
sudo -i
```

Check identity:

```bash
whoami
id
```

Production principle:

```text
Normal user
↓
sudo only for required operation
↓
Return to least privilege
```

Avoid permanently running application services as `root`.

---

# 26. Users

Current:

```bash
whoami
```

Identity:

```bash
id
```

All users:

```bash
cat /etc/passwd
```

Add:

```bash
sudo useradd -m appuser
```

Set password:

```bash
sudo passwd appuser
```

Ubuntu commonly provides higher-level user-management tooling as well.

---

# 27. Groups

Current:

```bash
groups
```

Create:

```bash
sudo groupadd developers
```

Add user:

```bash
sudo usermod -aG developers username
```

Inspect:

```bash
getent group developers
```

> The `-a` option is important when adding supplementary groups; omitting it can replace existing supplementary group memberships.

---

# 28. Processes

List:

```bash
ps aux
```

Search:

```bash
ps aux | grep nginx
```

Interactive:

```bash
top
```

If installed:

```bash
htop
```

Process tree:

```bash
pstree
```

Find PID:

```bash
pgrep nginx
```

---

# 29. Kill Processes

Graceful:

```bash
kill PID
```

Force:

```bash
kill -9 PID
```

By name:

```bash
pkill nginx
```

> Prefer graceful termination first. `SIGKILL` (`-9`) prevents the application from performing normal cleanup.

---

# 30. Jobs

Run background:

```bash
command &
```

List:

```bash
jobs
```

Foreground:

```bash
fg
```

Background:

```bash
bg
```

Suspend:

```text
Ctrl+Z
```

Terminate:

```text
Ctrl+C
```

---

# 31. nohup

Run after terminal disconnect:

```bash
nohup ./app.sh > app.log 2>&1 &
```

For production services, prefer `systemd` rather than using `nohup` as a service manager.

---

# 32. systemd

Ubuntu uses `systemd` as the service manager on standard Ubuntu Server installations.

Status:

```bash
systemctl status nginx
```

Start:

```bash
sudo systemctl start nginx
```

Stop:

```bash
sudo systemctl stop nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

Reload:

```bash
sudo systemctl reload nginx
```

Enable at boot:

```bash
sudo systemctl enable nginx
```

Disable:

```bash
sudo systemctl disable nginx
```

Enable + start:

```bash
sudo systemctl enable --now nginx
```

Check:

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

---

# 33. systemd Logs

```bash
journalctl
```

Service:

```bash
journalctl -u nginx
```

Follow:

```bash
journalctl -u nginx -f
```

Recent:

```bash
journalctl -u nginx --since "1 hour ago"
```

Boot:

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

---

# 34. Services / Production Pattern

A typical production application:

```text
Application
   ↓
systemd unit
   ↓
Process
   ↓
journalctl
   ↓
Monitoring
```

Do not rely on terminal sessions for production service lifecycle.

---

# 35. APT Package Management

Ubuntu recommends APT for installing and managing Debian packages. citeturn0search8

Update package index:

```bash
sudo apt update
```

Upgrade:

```bash
sudo apt upgrade
```

Full upgrade:

```bash
sudo apt full-upgrade
```

Search:

```bash
apt search nginx
```

Show:

```bash
apt show nginx
```

Install:

```bash
sudo apt install nginx
```

Remove:

```bash
sudo apt remove nginx
```

Purge configuration:

```bash
sudo apt purge nginx
```

Remove unused dependencies:

```bash
sudo apt autoremove
```

Clean cache:

```bash
sudo apt clean
```

Ubuntu package search:

https://packages.ubuntu.com/ citeturn0search9

---

# 36. apt vs apt-get

Interactive administration:

```bash
apt
```

Examples:

```bash
sudo apt update
sudo apt install nginx
```

Lower-level/script-oriented interface:

```bash
apt-get
```

For scripts, prefer commands with stable documented behavior and explicit error handling. Ubuntu documents `apt-get` as the command-line interface to the APT package system. citeturn0search14

---

# 37. Package Information

Installed:

```bash
dpkg -l
```

Find:

```bash
dpkg -l | grep nginx
```

Package files:

```bash
dpkg -L nginx
```

Package metadata:

```bash
dpkg -s nginx
```

Find which package owns a file:

```bash
dpkg -S /path/to/file
```

---

# 38. Snap

Ubuntu also supports Snap packages.

Installed:

```bash
snap list
```

Search:

```bash
snap find nginx
```

Install:

```bash
sudo snap install package-name
```

Refresh:

```bash
sudo snap refresh
```

Remove:

```bash
sudo snap remove package-name
```

Use the packaging format appropriate to the application's official distribution model.

---

# 39. Disk Usage

Filesystem:

```bash
df -h
```

Inodes:

```bash
df -i
```

Directory:

```bash
du -sh /var/log
```

Top directories:

```bash
du -h --max-depth=1 /var | sort -h
```

---

# 40. Mounts

```bash
mount
```

Disk/block devices:

```bash
lsblk
```

UUID:

```bash
blkid
```

Mount:

```bash
sudo mount /dev/sdb1 /mnt/data
```

Unmount:

```bash
sudo umount /mnt/data
```

> Verify device names carefully. Mounting or formatting the wrong device can cause data loss.

---

# 41. `/etc/fstab`

Inspect:

```bash
cat /etc/fstab
```

After changes:

```bash
sudo mount -a
```

> Validate `/etc/fstab` carefully before rebooting. A malformed entry can prevent normal boot/mount behavior.

---

# 42. Memory

```bash
free -h
```

Detailed:

```bash
cat /proc/meminfo
```

Swap:

```bash
swapon --show
```

---

# 43. CPU

```bash
lscpu
```

Load:

```bash
uptime
```

Real-time:

```bash
top
```

---

# 44. Network Interfaces

```bash
ip addr
```

Short:

```bash
ip -br addr
```

Links:

```bash
ip link
```

Routes:

```bash
ip route
```

Statistics:

```bash
ip -s link
```

---

# 45. Connectivity

Ping:

```bash
ping example.com
```

DNS:

```bash
getent hosts example.com
```

If installed:

```bash
dig example.com
```

Route:

```bash
tracepath example.com
```

TCP connection:

```bash
nc -vz example.com 443
```

---

# 46. Listening Ports

Modern Linux:

```bash
ss -lntup
```

TCP:

```bash
ss -lnt
```

UDP:

```bash
ss -lnu
```

Specific port:

```bash
ss -lntp | grep :8080
```

---

# 47. SSH

Connect:

```bash
ssh user@server
```

Specific port:

```bash
ssh -p 2222 user@server
```

Identity file:

```bash
ssh -i ~/.ssh/id_ed25519 user@server
```

Copy:

```bash
scp file.txt user@server:/tmp/
```

Recursive:

```bash
scp -r ./app user@server:/opt/
```

For large/reliable synchronization:

```bash
rsync -avz ./app/ user@server:/opt/app/
```

---

# 48. SSH Keys

Generate modern Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "developer@example.com"
```

Start agent:

```bash
eval "$(ssh-agent -s)"
```

Add key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Copy public key:

```bash
ssh-copy-id user@server
```

Never share:

```text
~/.ssh/id_ed25519
```

Share only:

```text
~/.ssh/id_ed25519.pub
```

---

# 49. Git

```bash
git --version
git status
git add .
git commit -m "Add feature"
git pull
git push
git switch -c feature/example
git switch main
git log --oneline --decorate --graph
```

Find:

```bash
command -v git
```

---

# 50. Node.js

```bash
node --version
npm --version
command -v node
command -v npm
```

Install dependencies:

```bash
npm install
```

Tests:

```bash
npm test
```

Build:

```bash
npm run build
```

For production, use the project's declared package manager and lockfile.

---

# 51. Python

```bash
python3 --version
python3 -m pip --version
command -v python3
```

Virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install:

```bash
python -m pip install -r requirements.txt
```

Deactivate:

```bash
deactivate
```

---

# 52. Java

```bash
java -version
javac -version
command -v java
```

Maven:

```bash
mvn -version
mvn test
mvn package
```

Gradle:

```bash
./gradlew build
./gradlew test
```

Prefer the project's Gradle Wrapper over requiring a globally installed Gradle version.

---

# 53. .NET

```bash
dotnet --info
dotnet --version
command -v dotnet
```

```bash
dotnet restore
dotnet build
dotnet test
dotnet publish
```

---

# 54. Docker

```bash
docker version
docker ps
docker images

docker build -t myapp:latest .
docker run --rm -p 8080:8080 myapp:latest

docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

---

# 55. Kubernetes

```bash
kubectl version --client
kubectl config get-contexts
kubectl get pods
kubectl get services
kubectl get deployments
kubectl logs POD_NAME
kubectl describe pod POD_NAME
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
```

---

# 56. File Compression

Tar:

```bash
tar -czf backup.tar.gz ./app
```

Extract:

```bash
tar -xzf backup.tar.gz
```

List:

```bash
tar -tzf backup.tar.gz
```

Zip:

```bash
zip -r app.zip app/
```

Unzip:

```bash
unzip app.zip
```

---

# 57. Archives / Backups

Production backup principle:

```text
Identify data
↓
Create backup
↓
Verify backup
↓
Store safely
↓
Test restoration
```

A backup that has never been restored successfully should not be treated as a proven recovery mechanism.

---

# 58. Bash Scripts

Create:

```bash
touch deploy.sh
chmod +x deploy.sh
```

Run:

```bash
./deploy.sh
```

Or:

```bash
bash deploy.sh
```

Shebang:

```bash
#!/usr/bin/env bash
```

---

# 59. Production Bash Script

Recommended baseline:

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

trap 'echo "ERROR: line=$LINENO command=$BASH_COMMAND" >&2' ERR

main() {
    echo "Starting deployment"

    # Validate
    # Deploy
    # Verify

    echo "Deployment completed"
}

main "$@"
```

Understand the behavior of:

```bash
set -e
set -u
set -o pipefail
```

Do not copy `set -e` blindly into every script without understanding Bash's exception rules.

Official Bash reference:
https://www.gnu.org/software/bash/manual/bash.html citeturn0search0turn0search13

---

# 60. Bash Parameters

```bash
#!/usr/bin/env bash

NAME="${1:-Aman}"

echo "Hello $NAME"
```

All arguments:

```bash
"$@"
```

Argument count:

```bash
$#
```

Script path:

```bash
$0
```

Last exit code:

```bash
$?
```

Process ID:

```bash
$$
```

---

# 61. Functions

```bash
#!/usr/bin/env bash

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

deploy() {
    log "Deploying application"
}

deploy
```

Prefer functions for reusable script logic.

---

# 62. Conditions

```bash
if [[ -f "$file" ]]; then
    echo "File exists"
else
    echo "Missing"
fi
```

Directory:

```bash
if [[ -d "$directory" ]]; then
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
if (( count > 10 )); then
    echo "Large"
fi
```

---

# 63. Loops

For:

```bash
for file in *.log; do
    echo "$file"
done
```

C-style:

```bash
for ((i=0; i<10; i++)); do
    echo "$i"
done
```

While:

```bash
while read -r line; do
    echo "$line"
done < input.txt
```

---

# 64. Case

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
        echo "Unknown environment" >&2
        exit 1
        ;;
esac
```

---

# 65. Arrays

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

---

# 66. Bash Debugging

Syntax check:

```bash
bash -n script.sh
```

Trace:

```bash
bash -x script.sh
```

Verbose shell:

```bash
set -x
```

Disable:

```bash
set +x
```

Use tracing carefully because commands may expose sensitive values.

---

# 67. ShellCheck

ShellCheck is a static-analysis tool for shell scripts.

Official:
https://www.shellcheck.net/

Typical:

```bash
shellcheck deploy.sh
```

Use ShellCheck in CI for production Bash.

---

# 68. Cron

List:

```bash
crontab -l
```

Edit:

```bash
crontab -e
```

Example:

```cron
0 2 * * * /opt/scripts/backup.sh
```

For new production services, timers managed by `systemd` may be preferable when the environment already uses systemd.

---

# 69. Environment Configuration

Common:

```text
/etc/environment
/etc/profile
/etc/profile.d/
/etc/bash.bashrc
~/.bashrc
~/.profile
```

Inspect:

```bash
cat ~/.bashrc
```

Reload:

```bash
source ~/.bashrc
```

Do not put secrets directly into shell startup files when an approved secret-management mechanism is available.

---

# 70. Logs

Traditional locations:

```bash
/var/log/
```

List:

```bash
ls -lah /var/log
```

Application:

```bash
tail -f /var/log/app.log
```

Systemd:

```bash
journalctl
```

Service:

```bash
journalctl -u myapp
```

---

# 71. System Troubleshooting

CPU:

```bash
top
uptime
lscpu
```

Memory:

```bash
free -h
```

Disk:

```bash
df -h
du -sh /var/*
```

Processes:

```bash
ps aux
```

Ports:

```bash
ss -lntup
```

Network:

```bash
ip -br addr
ip route
```

Logs:

```bash
journalctl -p err -b
```

---

# 72. Disk / Storage Troubleshooting

```bash
df -h
df -i
lsblk
blkid
mount
du -xh /var | sort -h
```

Find large files:

```bash
find /var -type f -size +500M -exec ls -lh {} \;
```

---

# 73. Permissions Troubleshooting

```bash
ls -la
id
groups
namei -l /path/to/file
getfacl /path/to/file
```

Check each parent directory when diagnosing access problems.

---

# 74. Process Troubleshooting

Find:

```bash
pgrep -af nginx
```

Inspect:

```bash
ps -fp PID
```

Open files:

```bash
lsof -p PID
```

Network sockets:

```bash
ss -lntup
```

---

# 75. Security Basics

Check current identity:

```bash
id
```

Open ports:

```bash
ss -lntup
```

Listening services:

```bash
systemctl --type=service --state=running
```

SSH configuration should be managed carefully and according to Ubuntu/OpenSSH security guidance.

Never:

```text
Expose unnecessary ports
Run applications as root
Store plaintext production secrets
Disable security controls to bypass errors
Install unknown packages blindly
```

---

# 76. Firewall — UFW

Ubuntu commonly provides UFW as a simplified firewall management interface.

Status:

```bash
sudo ufw status verbose
```

Enable:

```bash
sudo ufw enable
```

Allow SSH:

```bash
sudo ufw allow OpenSSH
```

Allow HTTPS:

```bash
sudo ufw allow 443/tcp
```

Remove:

```bash
sudo ufw delete allow 443/tcp
```

> Be careful when changing firewall rules over SSH. Always ensure you will not lock yourself out.

---

# 77. Package Security

Before installing a package:

```text
Official repository
↓
Correct package
↓
Version compatibility
↓
Security updates
↓
Dependencies
↓
License
```

Ubuntu package search:

https://packages.ubuntu.com/ citeturn0search9

Prefer official Ubuntu repositories or the software vendor's documented installation method.

---

# 78. CI/CD with Bash

Typical:

```text
Checkout
↓
Dependency Restore
↓
Lint
↓
Test
↓
Security Scan
↓
Build
↓
Package
↓
Publish
↓
Deploy
↓
Verify
```

Example:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

npm ci
npm test
npm run build
```

---

# 79. Exit-Safe Deployment

```bash
#!/usr/bin/env bash

set -Eeuo pipefail

main() {
    ./validate.sh
    ./build.sh
    ./deploy.sh
    ./verify.sh
}

main "$@"
```

Do not assume the last command succeeded merely because the script continued. Explicitly validate critical operations.

---

# 80. Idempotent Automation

Preferred model:

```text
Current State
↓
Desired State
↓
Compare
↓
Change Only When Necessary
↓
Verify
```

Bad:

```text
Every execution creates another resource
```

Good:

```text
Resource exists?
    ↓ yes → verify
    ↓ no  → create
```

Idempotency matters for:

```text
Configuration
Deployments
Infrastructure
Users
Directories
Services
Scheduled Jobs
CI/CD
```

---

# 81. Production Application Layout

Example:

```text
/opt/myapp/
├── bin/
├── config/
├── current/
├── releases/
├── shared/
│   ├── logs/
│   └── data/
└── backups/
```

Avoid placing mutable application state inside an immutable release directory.

---

# 82. Deployment Pattern

```text
Build Artifact
      ↓
Upload
      ↓
Create Release
      ↓
Install Dependencies
      ↓
Validate Configuration
      ↓
Run Migration if required
      ↓
Switch Release
      ↓
Restart / Reload
      ↓
Health Check
      ↓
Monitor
```

Prefer atomic/reversible release strategies where appropriate.

---

# 83. Health Checks

HTTP:

```bash
curl -f http://localhost:8080/health
```

HTTPS:

```bash
curl -f https://example.com/health
```

Service:

```bash
systemctl is-active --quiet myapp
```

Port:

```bash
ss -lnt | grep ':8080'
```

---

# 84. curl

GET:

```bash
curl https://example.com
```

Headers:

```bash
curl -I https://example.com
```

JSON:

```bash
curl -H "Accept: application/json" \
     https://api.example.com/users
```

POST:

```bash
curl -X POST \
     -H "Content-Type: application/json" \
     -d '{"name":"Aman"}' \
     https://api.example.com/users
```

Fail on HTTP errors:

```bash
curl -f https://example.com/health
```

---

# 85. jq

When installed:

```bash
curl -s https://api.example.com/health | jq
```

Extract:

```bash
curl -s https://api.example.com/health |
    jq -r '.status'
```

Use `jq` for structured JSON instead of fragile text parsing.

---

# 86. Git + SSH Workflow

```bash
ssh-keygen -t ed25519 -C "developer@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

git clone git@github.com:org/repository.git
cd repository
git status
```

Never commit:

```text
Private keys
.env secrets
Cloud credentials
Tokens
Passwords
```

---

# 87. Docker + Linux

```bash
docker version
docker ps
docker compose up -d
docker compose ps
docker compose logs -f
docker compose restart
docker compose down
```

Container troubleshooting:

```bash
docker logs CONTAINER
docker inspect CONTAINER
docker exec -it CONTAINER sh
```

---

# 88. Kubernetes + Linux

```bash
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
kubectl get deployments -A
kubectl logs POD -n NAMESPACE
kubectl describe pod POD -n NAMESPACE
```

Production rule:

```text
Observe
↓
Diagnose
↓
Change minimally
↓
Verify
```

---

# 89. Bash Script Quality Gate

Before merging:

```bash
bash -n script.sh
shellcheck script.sh
```

Then:

```text
Unit / integration tests
↓
Code review
↓
Security review where applicable
↓
CI
↓
Controlled deployment
```

---

# 90. Production Bash Checklist

```text
[ ] Correct shebang
[ ] Input validation
[ ] Quoted variables
[ ] Error handling
[ ] Exit codes
[ ] Idempotency
[ ] Logging
[ ] No hard-coded secrets
[ ] ShellCheck
[ ] Tests
[ ] CI validation
[ ] Least privilege
[ ] Safe temporary files
[ ] Cleanup / traps where required
[ ] Documentation
```

---

# 91. Temporary Files

Prefer secure temporary-file mechanisms.

Example:

```bash
tmp_file=$(mktemp)
trap 'rm -f "$tmp_file"' EXIT
```

Avoid predictable temporary filenames such as:

```bash
/tmp/myapp.tmp
```

---

# 92. Secure File Handling

Prefer:

```bash
cp -- "$source" "$destination"
rm -- "$file"
```

Use:

```text
Quoted variables
-- to terminate options where supported
Validated paths
Least privilege
```

---

# 93. Process Signals

Common:

```text
SIGTERM → graceful termination request
SIGINT  → interrupt
SIGHUP  → hangup / commonly reload semantics depending on application
SIGKILL → immediate termination
```

Example:

```bash
kill -TERM PID
```

Graceful shutdown should generally be attempted before:

```bash
kill -KILL PID
```

---

# 94. Traps

Cleanup:

```bash
cleanup() {
    rm -f "$tmp_file"
}

trap cleanup EXIT
```

Signal:

```bash
trap 'echo "Interrupted"; exit 130' INT TERM
```

Use traps for predictable cleanup and controlled shutdown.

---

# 95. Production Service Principle

Prefer:

```text
systemd
+
journalctl
+
health checks
+
monitoring
```

over:

```text
terminal session
+
nohup
+
manual restart
```

for long-running production services.

---

# 96. Linux Terminal vs Windows Shells

| Area | Ubuntu/Linux Terminal | Windows CMD | PowerShell |
|---|---|---|---|
| Shell | Bash commonly | `cmd.exe` | PowerShell |
| Package manager | APT / Snap | WinGet | WinGet / PSResourceGet |
| Service manager | systemd | SCM | systemd unavailable natively |
| Process tools | `ps`, `top`, `kill` | `tasklist`, `taskkill` | `Get-Process` |
| Networking | `ip`, `ss`, `dig` | `ipconfig`, `netstat` | `Get-Net*` |
| Script | `.sh` | `.bat` / `.cmd` | `.ps1` |
| Structured objects | Text-oriented | Text-oriented | Object pipeline |
| Server automation | 🟢 Excellent | 🟠 Compatibility | 🟢 Excellent |
| Cross-platform shell | 🟢 Unix/Linux standard | 🔴 Windows | 🟢 PowerShell 7+ |

---

# 97. When to Use Bash

Use Bash for:

```text
Linux server administration
CI/CD
Container environments
Build scripts
Deployment automation
Infrastructure automation
Unix tooling
System troubleshooting
Developer workflows
```

Use another language when the problem requires:

```text
Complex application logic
Large-scale data processing
Strong typing
Advanced testing
Complex concurrency
Long-lived application services
```

Do not force Bash to become an application programming language.

---

# 98. Official Ubuntu Sources

## Ubuntu Documentation

https://docs.ubuntu.com/ citeturn0search16

## Ubuntu Server Documentation

https://ubuntu.com/server/docs/ citeturn0search17

## Official Ubuntu Documentation Portal

https://help.ubuntu.com/ citeturn0search7

## Package Management

https://ubuntu.com/server/docs/how-to/software/package-management/ citeturn0search8

## Ubuntu Package Search

https://packages.ubuntu.com/ citeturn0search9

---

# 99. Official GNU / Bash Sources

## Bash Reference Manual

https://www.gnu.org/software/bash/manual/bash.html citeturn0search0

## GNU Bash Manual

https://www.gnu.org/software/bash/manual/ citeturn0search1

The Bash reference covers shell syntax, quoting, pipelines, variables, functions, conditionals, loops, redirection, job control, scripting, and Bash-specific features. citeturn0search0turn0search13

---

# 100. Final Production Principle

> **Linux terminal engineering is not memorizing commands. It is understanding the operating system, shell, filesystem, processes, networking, security, package management, services, automation, and recovery model.**

```text
Linux
 ↓
Shell
 ↓
Commands
 ↓
Pipelines
 ↓
Scripts
 ↓
Automation
 ↓
Services
 ↓
Observability
 ↓
Security
 ↓
Production Operations
```

A senior engineer should be comfortable with:

```text
Filesystem
Processes
Users
Groups
Permissions
Bash
Networking
DNS
SSH
APT
systemd
journalctl
Logs
Storage
Disks
Git
Node.js
Python
Java
.NET
Docker
Kubernetes
CI/CD
Security
Troubleshooting
Production Automation
```

**Core rule:**

```text
Correct command
+
Correct permissions
+
Correct target
+
Validation
+
Error handling
+
Observability
+
Recovery plan
=
Production-grade Linux operation
```

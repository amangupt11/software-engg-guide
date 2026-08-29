# 🖥️ Windows Command Prompt (cmd.exe) — Production Engineering Guide

> **Complete Windows Command Shell reference for command-line operations, batch scripting, file management, networking, administration, troubleshooting, automation, and CI/CD.**

> **Important:** Microsoft currently recommends PowerShell rather than Windows Commands for the most robust and up-to-date Windows automation. `cmd.exe` remains important for legacy systems, `.bat`/`.cmd` automation, installers, recovery workflows, and compatibility. citeturn0search0turn0search1

---

## 1. Command Prompt Architecture

```text
Windows Terminal / Console Host
            ↓
        cmd.exe
            ↓
Windows Commands + External Programs
            ↓
       .bat / .cmd
```

Distinguish:

```text
Terminal → application hosting a shell
Shell    → command interpreter
Command  → executable or shell command
Script   → .bat / .cmd automation
```

Official Microsoft overview:

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands

---

# 2. Start Command Prompt

From Run / Start:

```text
cmd
```

From another command shell:

```cmd
cmd
```

Execute one command and exit:

```cmd
cmd /c dir
```

Execute and remain open:

```cmd
cmd /k dir
```

Microsoft documents `/c` as execute-and-exit and `/k` as execute-and-remain-open. citeturn0search1

---

# 3. Command Prompt Help

General help:

```cmd
help
```

Command help:

```cmd
help dir
```

Most commands support:

```cmd
dir /?
```

Examples:

```cmd
copy /?
robocopy /?
ipconfig /?
tasklist /?
schtasks /?
```

Official command reference:

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/

---

# 4. Current Directory

```cmd
cd
```

Change directory:

```cmd
cd C:\Projects
```

Change drive:

```cmd
D:
```

Change drive and directory:

```cmd
cd /d D:\Projects
```

Parent:

```cmd
cd ..
```

Root:

```cmd
cd \
```

Home-like user directory:

```cmd
cd %USERPROFILE%
```

---

# 5. List Files

```cmd
dir
```

Detailed:

```cmd
dir /a
```

Hidden/system:

```cmd
dir /a:h
dir /a:s
```

Recursive:

```cmd
dir /s
```

Files only:

```cmd
dir /a-d
```

Directories only:

```cmd
dir /ad
```

Sort:

```cmd
dir /o:n
dir /o:-n
```

---

# 6. Create Directories

```cmd
mkdir projects
```

Alias:

```cmd
md projects
```

Nested:

```cmd
mkdir src\components
```

---

# 7. Create Files

Simple:

```cmd
type nul > README.md
```

Create with text:

```cmd
echo Hello > file.txt
```

Append:

```cmd
echo Another line >> file.txt
```

Multiple files:

```cmd
type nul > 01-engineering-foundations.md
type nul > 02-software-development-lifecycle.md
type nul > 03-requirements-and-planning.md
```

> `>` overwrites; `>>` appends.

---

# 8. Read Files

```cmd
type file.txt
```

Large files:

```cmd
more file.txt
```

Search:

```cmd
find "error" app.log
```

Case-insensitive:

```cmd
find /i "error" app.log
```

---

# 9. Copy

```cmd
copy source.txt destination.txt
```

Multiple files:

```cmd
copy *.txt backup\
```

Directories:

```cmd
xcopy source destination /E /I
```

For robust large directory synchronization, prefer:

```cmd
robocopy source destination /E
```

Official:

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy

---

# 10. Move / Rename

Move:

```cmd
move old.txt backup\
```

Rename:

```cmd
ren old.txt new.txt
```

Alias:

```cmd
rename old.txt new.txt
```

---

# 11. Delete

File:

```cmd
del file.txt
```

Multiple:

```cmd
del *.tmp
```

Directory:

```cmd
rmdir folder
```

Recursive:

```cmd
rmdir /s folder
```

Quiet:

```cmd
rmdir /s /q folder
```

> Always verify the current directory and target before destructive commands.

---

# 12. File Attributes

View:

```cmd
attrib
```

Hidden:

```cmd
attrib +h file.txt
```

Remove hidden:

```cmd
attrib -h file.txt
```

Read-only:

```cmd
attrib +r file.txt
attrib -r file.txt
```

---

# 13. Search Files

```cmd
where git
```

Find text recursively:

```cmd
findstr /s /i "TODO" *.js
```

Regex:

```cmd
findstr /r /s /i "error.*failed" *.log
```

Find files by pattern:

```cmd
dir /s /b *.md
```

---

# 14. Environment Variables

Display:

```cmd
set
```

Specific:

```cmd
set PATH
```

Read:

```cmd
echo %PATH%
echo %USERNAME%
echo %USERPROFILE%
echo %TEMP%
```

Set for current process:

```cmd
set APP_ENV=development
```

Use:

```cmd
echo %APP_ENV%
```

Remove:

```cmd
set APP_ENV=
```

Permanent user/system environment variables should normally be managed through Windows configuration tools, installers, or controlled automation rather than blindly changing them in application scripts.

Microsoft documents environment-variable substitution using `%VARIABLE%`. citeturn0search13

---

# 15. PATH

Inspect:

```cmd
echo %PATH%
```

Find executable:

```cmd
where node
where npm
where git
where python
where java
```

This is one of the fastest ways to diagnose:

```text
command not found
wrong executable
multiple installed versions
PATH ordering
```

---

# 16. Command Chaining

Run sequentially:

```cmd
command1 & command2
```

Run next only if previous succeeds:

```cmd
command1 && command2
```

Run next only if previous fails:

```cmd
command1 || command2
```

Example:

```cmd
npm install && npm test
```

Microsoft documents `&`, `&&`, and `||` as command operators. citeturn0search1

---

# 17. Redirection

Output to file:

```cmd
dir > files.txt
```

Append:

```cmd
dir >> files.txt
```

Standard input:

```cmd
command < input.txt
```

Redirect stderr:

```cmd
command 2> error.txt
```

Redirect stdout + stderr:

```cmd
command > output.txt 2>&1
```

---

# 18. Pipe

```cmd
dir | more
```

Search command output:

```cmd
tasklist | findstr node
```

Network:

```cmd
ipconfig /all | findstr DNS
```

---

# 19. Escape Character

Command Prompt uses:

```text
^
```

Example:

```cmd
echo Hello ^& World
```

Special characters include:

```text
<
>
|
&
^
```

Escape when literal interpretation is required.

---

# 20. Variables in Batch Scripts

```bat
@echo off

set NAME=Aman

echo Hello %NAME%
```

Batch scripts use `%VARIABLE%` expansion.

---

# 21. Batch Script Basics

File:

```text
script.bat
```

or:

```text
script.cmd
```

Run:

```cmd
script.bat
```

Explicit:

```cmd
call script.bat
```

Execute:

```cmd
cmd /c script.bat
```

---

# 22. Batch Parameters

`script.bat`:

```bat
@echo off

echo First: %1
echo Second: %2
echo All: %*
```

Run:

```cmd
script.bat hello world
```

Special variables:

```text
%0 → script name
%1 → first argument
%2 → second argument
%* → all arguments
```

---

# 23. IF

```bat
@echo off

if "%APP_ENV%"=="production" (
    echo Production
) else (
    echo Non-production
)
```

File exists:

```bat
if exist app.exe (
    echo Found
)
```

Not exists:

```bat
if not exist app.exe (
    echo Missing
)
```

---

# 24. FOR

Simple:

```bat
for %%F in (*.txt) do echo %%F
```

Recursive:

```bat
for /r %%F in (*.log) do echo %%F
```

Directories:

```bat
for /d %%D in (*) do echo %%D
```

In interactive Command Prompt, use a single `%`:

```cmd
for %F in (*.txt) do echo %F
```

In `.bat` files, use `%%`.

---

# 25. CALL

Call another batch script:

```bat
call build.bat
```

Call with parameters:

```bat
call deploy.bat production
```

---

# 26. SETLOCAL / ENDLOCAL

Isolate environment changes:

```bat
@echo off

setlocal

set APP_ENV=production

echo %APP_ENV%

endlocal
```

Useful for reliable batch scripts.

---

# 27. Delayed Expansion

Useful inside loops where variables change during execution:

```bat
@echo off

setlocal EnableDelayedExpansion

set COUNT=0

for %%F in (*.txt) do (
    set /a COUNT+=1
    echo !COUNT!: %%F
)

endlocal
```

---

# 28. Arithmetic

```cmd
set /a RESULT=10+20
echo %RESULT%
```

Batch:

```bat
set /a COUNT+=1
set /a TOTAL=PRICE*QUANTITY
```

---

# 29. Strings

```cmd
set NAME=Aman
echo %NAME%
```

Substring:

```cmd
echo %NAME:~0,3%
```

Replace:

```cmd
set TEXT=hello-world
echo %TEXT:-= %
```

---

# 30. Process Management

List:

```cmd
tasklist
```

Filter:

```cmd
tasklist | findstr node
```

Detailed:

```cmd
tasklist /v
```

Kill by PID:

```cmd
taskkill /PID 1234
```

Force:

```cmd
taskkill /PID 1234 /F
```

By image:

```cmd
taskkill /IM node.exe
```

Force by image:

```cmd
taskkill /IM node.exe /F
```

---

# 31. Start Applications

```cmd
start notepad.exe
```

Open directory:

```cmd
start .
```

Open URL:

```cmd
start https://learn.microsoft.com
```

Wait for a program:

```cmd
start /wait setup.exe
```

Microsoft's `start` command supports launching applications and separate command windows. citeturn0search8

---

# 32. Services

Query:

```cmd
sc query
```

Specific:

```cmd
sc query Spooler
```

Start:

```cmd
sc start Spooler
```

Stop:

```cmd
sc stop Spooler
```

Configure:

```cmd
sc config Spooler start= auto
```

> `sc` is a powerful service-management tool. Use elevated privileges and change control for production systems.

---

# 33. Windows Networking

IP configuration:

```cmd
ipconfig
ipconfig /all
```

Release:

```cmd
ipconfig /release
```

Renew:

```cmd
ipconfig /renew
```

DNS cache:

```cmd
ipconfig /displaydns
ipconfig /flushdns
```

Ping:

```cmd
ping example.com
```

Trace route:

```cmd
tracert example.com
```

Path analysis:

```cmd
pathping example.com
```

ARP:

```cmd
arp -a
```

Routes:

```cmd
route print
```

---

# 34. DNS

```cmd
nslookup example.com
nslookup example.com 1.1.1.1
```

Modern PowerShell alternative:

```powershell
Resolve-DnsName example.com
```

---

# 35. Network Connections

```cmd
netstat -ano
```

Listening ports:

```cmd
netstat -ano | findstr LISTENING
```

Specific port:

```cmd
netstat -ano | findstr :8080
```

Then map PID:

```cmd
tasklist | findstr 1234
```

---

# 36. Firewall

Windows Firewall administration is commonly performed with:

```cmd
netsh advfirewall
```

Inspect:

```cmd
netsh advfirewall show allprofiles
```

For modern automation, PowerShell's `NetSecurity` module provides a richer object-based interface.

---

# 37. Users

```cmd
whoami
whoami /user
whoami /groups
```

Local users:

```cmd
net user
```

Details:

```cmd
net user username
```

Create:

```cmd
net user appuser * /add
```

Disable:

```cmd
net user appuser /active:no
```

Enable:

```cmd
net user appuser /active:yes
```

---

# 38. Groups

List:

```cmd
net localgroup
```

Members:

```cmd
net localgroup Administrators
```

Add:

```cmd
net localgroup Administrators username /add
```

Remove:

```cmd
net localgroup Administrators username /delete
```

> Administrator membership is a privileged operation. Use least privilege.

---

# 39. File Permissions

Inspect:

```cmd
icacls file.txt
```

Directory:

```cmd
icacls C:\Projects
```

Grant:

```cmd
icacls file.txt /grant username:R
```

Grant modify:

```cmd
icacls folder /grant username:M
```

Inheritance and ACL changes can have broad effects. Review before applying.

---

# 40. Disk Management

Disk information:

```cmd
wmic diskdrive get model,size,status
```

> WMIC is deprecated on modern Windows. Prefer PowerShell CIM cmdlets or the supported Windows management tools for new automation.

DiskPart:

```cmd
diskpart
```

Inside DiskPart:

```text
list disk
list volume
```

> DiskPart can destroy partitions/data. Never run destructive DiskPart commands without verifying the target disk.

---

# 41. System Information

```cmd
systeminfo
```

Computer name:

```cmd
hostname
```

Architecture:

```cmd
echo %PROCESSOR_ARCHITECTURE%
```

Version:

```cmd
ver
```

---

# 42. Date / Time

```cmd
date /t
time /t
```

---

# 43. System File Checks

System File Checker:

```cmd
sfc /scannow
```

DISM:

```cmd
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
```

Repair when appropriate:

```cmd
DISM /Online /Cleanup-Image /RestoreHealth
```

Use elevated Command Prompt.

---

# 44. Shutdown / Restart

Shutdown:

```cmd
shutdown /s /t 0
```

Restart:

```cmd
shutdown /r /t 0
```

Abort:

```cmd
shutdown /a
```

Remote systems require appropriate permissions and configuration.

---

# 45. Event Logs

Legacy command:

```cmd
wevtutil el
```

Query:

```cmd
wevtutil qe System /c:20 /f:text
```

Export:

```cmd
wevtutil epl System system.evtx
```

For advanced filtering and object-oriented analysis, PowerShell `Get-WinEvent` is generally preferable.

---

# 46. Scheduled Tasks

List:

```cmd
schtasks /query
```

Detailed:

```cmd
schtasks /query /fo LIST /v
```

Run:

```cmd
schtasks /run /tn "MyTask"
```

Create:

```cmd
schtasks /create /sc daily /tn "MyTask" /tr "C:\Scripts\backup.cmd" /st 23:00
```

Delete:

```cmd
schtasks /delete /tn "MyTask" /f
```

Official:
https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks

---

# 47. Robocopy

Basic:

```cmd
robocopy C:\Source D:\Backup
```

Include subdirectories:

```cmd
robocopy C:\Source D:\Backup /E
```

Mirror:

```cmd
robocopy C:\Source D:\Backup /MIR
```

> `/MIR` can delete files from the destination to make it match the source. Verify source and destination carefully.

Retry controls:

```cmd
robocopy C:\Source D:\Backup /E /R:3 /W:5
```

Logging:

```cmd
robocopy C:\Source D:\Backup /E /LOG:backup.log
```

Official:
https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy

---

# 48. Compression

Windows supports archive operations through PowerShell:

```powershell
Compress-Archive -Path .\src -DestinationPath .\src.zip
Expand-Archive .\src.zip -DestinationPath .\out
```

`cmd.exe` itself does not provide a modern general-purpose ZIP workflow comparable to PowerShell.

---

# 49. Git

```cmd
git --version
git status
git add .
git commit -m "Add feature"
git pull
git push
git branch
git switch -c feature/example
git switch main
git log --oneline --decorate --graph
```

Locate:

```cmd
where git
```

---

# 50. Node.js / npm

```cmd
node --version
npm --version
where node
where npm
```

Install:

```cmd
npm install
```

Run tests:

```cmd
npm test
```

Build:

```cmd
npm run build
```

---

# 51. Python

```cmd
python --version
py --version
where python
```

Create virtual environment:

```cmd
py -m venv .venv
```

Activate:

```cmd
.venv\Scripts\activate
```

Install:

```cmd
python -m pip install -r requirements.txt
```

---

# 52. Java

```cmd
java -version
javac -version
where java
where javac
```

Maven:

```cmd
mvn -version
mvn test
mvn package
```

Gradle:

```cmd
gradle --version
gradle build
```

Prefer the project's Gradle Wrapper where available:

```cmd
gradlew.bat build
```

---

# 53. .NET

```cmd
dotnet --info
dotnet --version
where dotnet
```

Restore:

```cmd
dotnet restore
```

Build:

```cmd
dotnet build
```

Test:

```cmd
dotnet test
```

Publish:

```cmd
dotnet publish
```

---

# 54. Docker

```cmd
docker version
docker ps
docker images

docker build -t myapp:latest .
docker run --rm -p 8080:8080 myapp:latest

docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

---

# 55. Kubernetes

```cmd
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

# 56. Windows Terminal

Windows Terminal can host:

```text
Command Prompt
PowerShell
WSL
Other command-line applications
```

It supports tabs, panes, Unicode/UTF-8, GPU-accelerated rendering, profiles, and customization. citeturn0search7

Launch:

```cmd
wt
```

Open Command Prompt:

```cmd
wt cmd
```

New tab:

```cmd
wt new-tab cmd
```

Official:
https://learn.microsoft.com/en-us/windows/terminal/

---

# 57. Batch Script Logging

```bat
@echo off

set LOGFILE=%~dp0deploy.log

echo [%date% %time%] Deployment started >> "%LOGFILE%"

call build.bat >> "%LOGFILE%" 2>&1

if errorlevel 1 (
    echo [%date% %time%] Deployment failed >> "%LOGFILE%"
    exit /b 1
)

echo [%date% %time%] Deployment completed >> "%LOGFILE%"
exit /b 0
```

---

# 58. Exit Codes

Check:

```cmd
echo %ERRORLEVEL%
```

Batch:

```bat
if errorlevel 1 (
    echo Failed
    exit /b 1
)
```

Success:

```bat
exit /b 0
```

Important:

```text
0    → normally success
non-0 → normally failure
```

Always consult the specific command's documentation for its actual exit-code contract.

---

# 59. Production Batch Script Pattern

```bat
@echo off
setlocal EnableExtensions EnableDelayedExpansion

set "APP_ENV=production"
set "LOGFILE=%~dp0deploy.log"

echo [%date% %time%] Starting deployment >> "%LOGFILE%"

call :validate
if errorlevel 1 exit /b 1

call :deploy
if errorlevel 1 exit /b 1

call :verify
if errorlevel 1 exit /b 1

echo [%date% %time%] Deployment successful >> "%LOGFILE%"

endlocal
exit /b 0

:validate
echo Validating...
if not exist "app.exe" (
    echo app.exe not found
    exit /b 1
)
exit /b 0

:deploy
echo Deploying...
rem deployment commands
exit /b 0

:verify
echo Verifying...
rem verification commands
exit /b 0
```

---

# 60. Batch Script Engineering Rules

```text
[ ] @echo off
[ ] setlocal
[ ] Quote paths
[ ] Validate inputs
[ ] Check ERRORLEVEL
[ ] Use exit /b
[ ] Log important operations
[ ] Avoid hard-coded secrets
[ ] Avoid destructive commands without validation
[ ] Use absolute paths when ambiguity is dangerous
[ ] Keep scripts in source control
[ ] Test before production
```

---

# 61. Security

Avoid:

```bat
set PASSWORD=production-secret
```

Avoid executing untrusted downloaded content.

Be careful with:

```cmd
del
rmdir /s
format
diskpart
reg
sc
net user
net localgroup
icacls
```

Security principles:

```text
Least Privilege
Input Validation
Safe Quoting
Secret Management
Change Control
Code Review
Auditing
```

---

# 62. Troubleshooting Toolkit

### Current location

```cmd
cd
```

### Environment

```cmd
set
```

### PATH executable

```cmd
where program
```

### Process

```cmd
tasklist
```

### Kill process

```cmd
taskkill /PID 1234 /F
```

### Services

```cmd
sc query
```

### IP

```cmd
ipconfig /all
```

### DNS

```cmd
nslookup example.com
```

### Connectivity

```cmd
ping example.com
```

### Route

```cmd
tracert example.com
```

### Ports

```cmd
netstat -ano
```

### Events

```cmd
wevtutil qe System /c:20 /f:text
```

### System

```cmd
systeminfo
```

---

# 63. Command Prompt vs PowerShell

| Capability | Command Prompt | PowerShell |
|---|---|---|
| Legacy `.bat` / `.cmd` | 🟢 Excellent | 🟢 Can invoke |
| Windows built-in commands | 🟢 Excellent | 🟢 Can invoke |
| Object pipeline | 🔴 No | 🟢 Yes |
| Advanced scripting | 🟠 Limited | 🟢 Excellent |
| Structured data | 🟠 Limited | 🟢 Excellent |
| Modern Windows automation | 🟠 Legacy/compatibility | 🟢 Preferred |
| Cross-platform automation | 🔴 Limited | 🟢 PowerShell 7+ |
| Existing legacy tooling | 🟢 Excellent | 🟢 Excellent |

Microsoft explicitly recommends PowerShell for the most robust, current Windows automation. citeturn0search0

---

# 64. When to Use cmd.exe

Use Command Prompt when:

```text
Legacy .bat/.cmd scripts
Legacy installers
Recovery environments
Existing enterprise scripts
Simple one-line commands
Compatibility requirements
Tools specifically documented for cmd.exe
```

Prefer PowerShell when:

```text
New automation
Complex scripting
Structured data
Windows administration
API automation
Object processing
Cross-platform scripting
Testing
Modern CI/CD
```

---

# 65. Official Microsoft Sources

## Windows Commands

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands

## `cmd`

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cmd

## Command Reference A-Z

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/

## `help`

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/help

## `robocopy`

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy

## `start`

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/start

## Windows Terminal

https://learn.microsoft.com/en-us/windows/terminal/

## PowerShell / Windows Command Shell Concepts

https://learn.microsoft.com/en-us/powershell/scripting/what-is-a-command-shell

---

# 🏁 Final Principle

> **Command Prompt is a compatibility-focused Windows command shell and batch automation environment. PowerShell is the preferred modern Windows automation platform.**

Use the right tool:

```text
Simple / Legacy
      ↓
cmd.exe
      ↓
.bat / .cmd

Modern / Production Automation
      ↓
PowerShell
      ↓
.ps1 / Modules
```

Treat batch scripts as production software when they affect production systems:

```text
Validate
↓
Quote safely
↓
Check exit codes
↓
Log
↓
Test
↓
Review
↓
Automate
↓
Monitor
```

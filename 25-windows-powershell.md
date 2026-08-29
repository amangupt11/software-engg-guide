# 🪟 Windows PowerShell — Production Engineering Guide

> Complete PowerShell reference for terminal usage, Windows administration, automation, scripting, DevOps, CI/CD, cloud operations, troubleshooting, security, and production operations.

## 1. Versions

Windows commonly has:

```text
Windows PowerShell 5.1
PowerShell 7+
```

PowerShell 7 installs side-by-side with Windows PowerShell 5.1; it does not replace it. Some legacy modules still require 5.1. [Microsoft — Install PowerShell](https://learn.microsoft.com/powershell/scripting/install/install-powershell-on-windows)

```powershell
$PSVersionTable
$PSVersionTable.PSVersion
$PSVersionTable.PSEdition
pwsh
powershell.exe
```

## 2. Install / Upgrade

Microsoft recommends WinGet for PowerShell installation on Windows clients; MSI is useful for many server and enterprise deployments.

```powershell
winget search --id Microsoft.PowerShell --exact
winget install --id Microsoft.PowerShell --source winget
winget upgrade --id Microsoft.PowerShell
winget uninstall --id Microsoft.PowerShell
pwsh --version
```

Official:
- https://learn.microsoft.com/powershell/
- https://learn.microsoft.com/powershell/scripting/install/install-powershell-on-windows
- https://learn.microsoft.com/windows/package-manager/

## 3. Terminal Navigation

```powershell
Get-Location
Set-Location C:\Projects
Set-Location ..
Set-Location $HOME
Get-ChildItem
Get-ChildItem -Force
Clear-Host
```

Common aliases:

```powershell
pwd
cd
ls
dir
cls
```

For production scripts prefer the full cmdlet names.

## 4. Files and Directories

```powershell
New-Item -ItemType Directory -Path .\src
New-Item -ItemType File -Path .\README.md
Copy-Item .\source.txt .\destination.txt
Copy-Item .\src .\backup -Recurse
Move-Item .\old.txt .\new.txt
Rename-Item .\old.txt .\new.txt
Remove-Item .\file.txt
Remove-Item .\folder -Recurse
```

Create many files:

```powershell
$files = @(
    "README.md",
    "01-engineering-foundations.md",
    "02-software-development-lifecycle.md"
)

$files | ForEach-Object {
    New-Item -ItemType File -Path $_ -Force
}
```

Find:

```powershell
Get-ChildItem . -Recurse -File
Get-ChildItem . -Recurse -Filter *.md -File
Get-ChildItem . -Recurse -Filter "*test*"
```

## 5. Read / Write Files

```powershell
Get-Content .\README.md
Get-Content .\README.md -Raw
Get-Content .\app.log -Tail 50
Get-Content .\app.log -Wait

Set-Content .\output.txt "Hello"
Add-Content .\output.txt "Another line"

Get-Process | Out-File .\processes.txt
```

## 6. Variables / Environment

```powershell
$name = "Aman"
$count = 10
$enabled = $true

$env:APP_ENV
$env:APP_ENV = "development"
Remove-Item Env:APP_ENV

Get-ChildItem Env:
$env:PATH
```

## 7. Arrays / Hashtables

```powershell
$items = @("API", "Web", "Database")
$items[0]
$items.Count

$config = @{
    Environment = "production"
    Port = 8080
}

$config.Environment
$config["Port"]
```

## 8. Conditions

```powershell
if ($env:APP_ENV -eq "production") {
    Write-Host "Production"
}
elseif ($env:APP_ENV -eq "staging") {
    Write-Host "Staging"
}
else {
    Write-Host "Development"
}
```

Operators:

```text
-eq -ne -gt -ge -lt -le
-and -or -not !
-like -match
```

## 9. Loops

```powershell
foreach ($item in $items) {
    Write-Host $item
}

$items | ForEach-Object {
    Write-Host $_
}

for ($i = 0; $i -lt 10; $i++) {
    Write-Host $i
}

while ($condition) {
    # work
}
```

## 10. Functions

```powershell
function Get-Greeting {
    param(
        [Parameter(Mandatory)]
        [string]$Name
    )

    "Hello $Name"
}

Get-Greeting -Name "Aman"
```

Production-style:

```powershell
function Get-PortStatus {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [int]$Port
    )

    Test-NetConnection localhost -Port $Port
}
```

Use approved verbs:

```powershell
Get-Verb
```

## 11. Commands / Help

PowerShell commands generally use `Verb-Noun`.

```powershell
Get-Command
Get-Command *service*
Get-Command Get-Process -Syntax

Get-Help Get-Process
Get-Help Get-Process -Detailed
Get-Help Get-Process -Examples
Get-Help Get-Process -Online
Get-Help about_*
```

Microsoft documents `Get-Command` as the discovery mechanism for cmdlets, functions, aliases, scripts, and applications.

Official:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/get-command

## 12. Objects / Pipeline

PowerShell pipelines pass objects.

```powershell
Get-Process | Get-Member

Get-Process |
    Where-Object CPU -gt 100 |
    Sort-Object CPU -Descending |
    Select-Object -First 10 Name, Id, CPU
```

Common pipeline stages:

```text
Get
↓
Where
↓
ForEach
↓
Sort
↓
Select
↓
Export
```

Formatting is for presentation:

```powershell
Get-Process | Format-Table Name, Id, CPU
Get-Process | Format-List *
```

Do not feed `Format-*` output into later data-processing commands.

## 13. Data Conversion

```powershell
Import-Csv .\data.csv
Get-Process | Export-Csv .\processes.csv -NoTypeInformation

$json = Get-Content .\config.json -Raw | ConvertFrom-Json
$config | ConvertTo-Json -Depth 10

Get-Process | Export-Clixml .\processes.xml
```

## 14. Processes

```powershell
Get-Process
Get-Process chrome
Start-Process notepad.exe
Start-Process .\app.exe -Wait
Stop-Process -Name notepad
Stop-Process -Name notepad -Force
```

## 15. Services

```powershell
Get-Service
Get-Service *sql*
Start-Service Spooler
Stop-Service Spooler
Restart-Service Spooler
Set-Service Spooler -StartupType Automatic
Get-Service Spooler | Format-List *
```

## 16. Windows Information

```powershell
Get-ComputerInfo
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_ComputerSystem
Get-CimInstance Win32_BIOS
Get-CimInstance Win32_Processor
Get-CimInstance Win32_PhysicalMemory
Get-CimInstance Win32_LogicalDisk
Get-PSDrive
```

## 17. Networking

```powershell
Get-NetIPConfiguration
Get-NetIPAddress
Get-NetAdapter
Get-NetRoute
Get-DnsClient

Resolve-DnsName example.com
Test-Connection example.com
Test-NetConnection example.com -Port 443
Get-NetTCPConnection
Get-NetTCPConnection -LocalPort 8080
Get-NetAdapterStatistics
```

## 18. HTTP / REST APIs

```powershell
Invoke-RestMethod https://api.example.com/health
Invoke-WebRequest https://example.com
```

POST JSON:

```powershell
$body = @{
    name = "Aman"
    role = "developer"
} | ConvertTo-Json

Invoke-RestMethod `
    -Uri "https://api.example.com/users" `
    -Method Post `
    -ContentType "application/json" `
    -Body $body
```

Headers:

```powershell
$headers = @{
    Authorization = "Bearer $token"
}

Invoke-RestMethod `
    -Uri "https://api.example.com/users" `
    -Headers $headers
```

## 19. Registry / Providers

PowerShell providers expose specialized stores using a path model.

```powershell
Get-PSProvider
Get-PSDrive
Get-ChildItem Env:
Get-ChildItem HKCU:\
Get-ChildItem HKLM:\
Get-ChildItem Cert:\
```

Read registry:

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

Create:

```powershell
New-Item "HKCU:\Software\MyCompany\MyApp"
```

Set:

```powershell
New-ItemProperty `
    -Path "HKCU:\Software\MyCompany\MyApp" `
    -Name "Environment" `
    -Value "Development" `
    -PropertyType String
```

Use registry changes carefully and under change control for production.

Official:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_providers

## 20. Modules

```powershell
Get-Module
Get-Module -ListAvailable
Import-Module ModuleName
Remove-Module ModuleName
Get-Command -Module ModuleName
```

A module can contain cmdlets, functions, providers, variables, and related resources.

Official:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_modules

## 21. PowerShell Package / Module Management

PowerShell Gallery:

https://www.powershellgallery.com/

PowerShell 7.4+ includes `Microsoft.PowerShell.PSResourceGet`, which Microsoft identifies as the preferred package manager for PowerShell modules.

```powershell
Get-Command Install-PSResource
Find-PSResource -Name Pester
Install-PSResource -Name Pester
Get-PSResourceRepository
```

Legacy PowerShellGet:

```powershell
Find-Module
Install-Module
Update-Module
Uninstall-Module
```

## 22. Pester Testing

```powershell
Install-PSResource -Name Pester
Invoke-Pester
```

Example:

```powershell
Describe "Calculator" {
    It "adds two numbers" {
        2 + 3 | Should -Be 5
    }
}
```

Use tests for reusable automation and deployment tooling.

Official:
https://pester.dev/

## 23. PSScriptAnalyzer

Official:
https://learn.microsoft.com/powershell/utility-modules/psscriptanalyzer/overview

```powershell
Install-PSResource -Name PSScriptAnalyzer
Invoke-ScriptAnalyzer -Path .\script.ps1
Invoke-ScriptAnalyzer -Path . -Recurse
```

Use it as a CI quality gate.

## 24. Error Handling

```powershell
try {
    Get-Item .\missing.txt -ErrorAction Stop
}
catch {
    Write-Error $_
}
finally {
    # cleanup
}
```

Useful:

```powershell
$Error[0]
$Error[0].Exception
$Error.Clear()
```

For production scripts:

```powershell
$ErrorActionPreference = "Stop"
```

or prefer local `-ErrorAction Stop` where narrower scope is clearer.

## 25. Logging

```powershell
Write-Verbose "Connecting"
Write-Information "Deployment started"
Write-Warning "Configuration missing"
Write-Error "Deployment failed"
```

Do not log:

```text
Passwords
Tokens
API keys
Private keys
Sensitive personal data
```

## 26. Execution Policy

Inspect:

```powershell
Get-ExecutionPolicy -List
```

Set only when required and at the narrowest appropriate scope:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Do not use `Bypass` as a blanket solution to execute untrusted scripts. Execution policy is only one security control.

## 27. Production Script

```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory)]
    [ValidateSet("development", "staging", "production")]
    [string]$Environment
)

$ErrorActionPreference = "Stop"

function Test-Prerequisites {
    # validate dependencies
}

function Deploy-Application {
    param([string]$TargetEnvironment)
    # deployment
}

try {
    Test-Prerequisites
    Deploy-Application -TargetEnvironment $Environment
    exit 0
}
catch {
    Write-Error $_
    exit 1
}
```

Production scripts should normally have:

```text
Input validation
Error handling
Logging
Exit codes
Idempotency
Testing
Static analysis
Documentation
```

## 28. Jobs

```powershell
$job = Start-Job {
    Get-Process
}

Get-Job
Receive-Job $job
Stop-Job $job
Remove-Job $job
```

Parallel:

```powershell
1..10 | ForEach-Object -Parallel {
    $_ * 2
}
```

Use parallelism only after considering CPU, memory, I/O, ordering, rate limits, and failure behavior.

## 29. Remoting

PowerShell remoting supports temporary and persistent remote sessions.

```powershell
Enter-PSSession -ComputerName SERVER01
Exit-PSSession

Invoke-Command -ComputerName SERVER01 -ScriptBlock {
    Get-Service
}

$session = New-PSSession -ComputerName SERVER01

Invoke-Command -Session $session -ScriptBlock {
    Get-Process
}

Remove-PSSession $session
```

Multiple systems:

```powershell
Invoke-Command `
    -ComputerName SERVER01, SERVER02 `
    -ScriptBlock { Get-ComputerInfo }
```

Remoting requires appropriate configuration, authentication, authorization, and network access.

Official:
https://learn.microsoft.com/powershell/scripting/security/remoting/

## 30. Scheduled Tasks

```powershell
Get-ScheduledTask
Get-ScheduledTask -TaskName "*Backup*"
Start-ScheduledTask -TaskName "MyTask"
Stop-ScheduledTask -TaskName "MyTask"
Get-ScheduledTask -TaskName "MyTask" | Get-ScheduledTaskInfo
```

## 31. Windows Event Logs

```powershell
Get-WinEvent -ListLog *
Get-WinEvent -LogName System -MaxEvents 20
Get-WinEvent -LogName Application -MaxEvents 50
```

Targeted filter:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "System"
    Level   = 2
} -MaxEvents 50
```

## 32. Local Users / Groups

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"
```

Create:

```powershell
New-LocalUser -Name "appuser"
```

Disable / enable:

```powershell
Disable-LocalUser -Name "appuser"
Enable-LocalUser -Name "appuser"
```

Never grant Administrator rights without a documented requirement.

## 33. Permissions

```powershell
Get-Acl .\file.txt
```

Use least privilege and review ACL changes before applying them.

## 34. WinGet

```powershell
winget search vscode
winget install --id Microsoft.VisualStudioCode
winget list
winget upgrade --id Microsoft.VisualStudioCode
winget uninstall --id Microsoft.VisualStudioCode
```

Official:
https://learn.microsoft.com/windows/package-manager/

## 35. Git

```powershell
git status
git add .
git commit -m "Add feature"
git pull
git push
git switch -c feature/example
git switch main
git log --oneline --decorate --graph
```

Native command exit code:

```powershell
git status
if ($LASTEXITCODE -ne 0) {
    throw "Git failed"
}
```

## 36. Docker

```powershell
docker version
docker images
docker ps
docker ps -a

docker build -t myapp:latest .
docker run --rm -p 8080:8080 myapp:latest

docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

## 37. Kubernetes

```powershell
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

## 38. Cloud CLIs

Examples:

```powershell
az login
aws configure
gcloud auth login
```

Use official cloud credential mechanisms. Never hard-code cloud secrets.

## 39. CI/CD

PowerShell works well in:

```text
GitHub Actions
Azure DevOps
GitLab CI
Jenkins
TeamCity
Buildkite
```

Recommended:

```text
Checkout
↓
Restore
↓
PSScriptAnalyzer
↓
Pester
↓
Security
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

```powershell
.\build.ps1

if ($LASTEXITCODE -ne 0) {
    exit 1
}
```

## 40. Performance Troubleshooting

CPU:

```powershell
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10 Name, Id, CPU
```

Memory:

```powershell
Get-Process |
    Sort-Object WorkingSet64 -Descending |
    Select-Object -First 10 Name, Id,
        @{N="MemoryMB";E={[math]::Round($_.WorkingSet64 / 1MB, 2)}}
```

Network:

```powershell
Get-NetAdapterStatistics
Get-NetTCPConnection
```

Disk:

```powershell
Get-PSDrive -PSProvider FileSystem
```

For deep Windows performance analysis, use appropriate Windows performance diagnostics in addition to PowerShell.

## 41. Security Rules

```text
[ ] Least privilege
[ ] No secrets in source
[ ] Approved credential store
[ ] Input validation
[ ] Path validation
[ ] Avoid Invoke-Expression with untrusted input
[ ] PSScriptAnalyzer
[ ] Tests
[ ] Code review
[ ] Restricted remoting
[ ] Safe logging
[ ] Protected CI/CD credentials
```

Avoid:

```powershell
Invoke-Expression $userInput
```

Avoid piping untrusted downloads directly into execution.

## 42. Idempotent Automation

Prefer:

```text
Current State
↓
Desired State
↓
Change Only If Required
↓
Verify
```

Example:

```powershell
if (-not (Test-Path $path)) {
    New-Item -ItemType Directory -Path $path
}
```

Idempotency is especially important for:

```text
Configuration
Infrastructure
Deployments
Services
Scheduled Tasks
CI/CD
```

## 43. Profiles

```powershell
$PROFILE
Test-Path $PROFILE
New-Item -ItemType File -Path $PROFILE -Force
notepad $PROFILE
```

Do not put critical production automation only in a personal profile. Keep reusable engineering scripts in source control.

## 44. Module Structure

```text
MyCompany.Tools/
├── MyCompany.Tools.psd1
├── MyCompany.Tools.psm1
├── Public/
├── Private/
├── Tests/
└── README.md
```

Production module:

```text
Public API
Private implementation
Manifest
Tests
Documentation
Versioning
```

## 45. Enterprise Workflow

```text
Developer
↓
PowerShell 7+
↓
Git
↓
PSScriptAnalyzer
↓
Pester
↓
Code Review
↓
CI/CD
↓
Approved Secrets
↓
Controlled Deployment
↓
Logs / Metrics
↓
Operations
```

For legacy Windows-specific administration, Windows PowerShell 5.1 may remain necessary.

## 46. Official Microsoft Sources

PowerShell:
https://learn.microsoft.com/powershell/

Install:
https://learn.microsoft.com/powershell/scripting/install/install-powershell-on-windows

Cmdlets:
https://learn.microsoft.com/powershell/module/

About topics:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/

Get-Command:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/get-command

Modules:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_modules

Providers:
https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_providers

Remoting:
https://learn.microsoft.com/powershell/scripting/security/remoting/

PowerShell Gallery:
https://www.powershellgallery.com/

PSScriptAnalyzer:
https://learn.microsoft.com/powershell/utility-modules/psscriptanalyzer/overview

Windows Package Manager:
https://learn.microsoft.com/windows/package-manager/

## 47. Production Principle

> **Treat PowerShell automation as production software.**

```text
Design
↓
Validate
↓
Implement
↓
Test
↓
Analyze
↓
Review
↓
Automate
↓
Deploy
↓
Observe
↓
Maintain
```

A senior engineer should be able to use PowerShell for:

```text
Files
Processes
Services
Windows
Registry
Users
Groups
Networking
HTTP APIs
Events
Logs
Modules
Packages
Git
Docker
Kubernetes
Cloud
CI/CD
Testing
Security
Remoting
Production Automation
```

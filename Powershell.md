## Getting Help

| PowerShell                       | Linux (bash)                 | Description                            |
| -------------------------------- | ---------------------------- | -------------------------------------- |
| `Get-Help Get-Process`           | `man ps`                     | Show help/manual for a command         |
| `Get-Help Get-Process -Examples` | `man ps` (see EXAMPLES)      | Show examples                          |
| `Get-Help Get-Process -Full`     | `man -a ps`                  | Full detailed help                     |
| `Get-Help *network*`             | `apropos network`            | Search help topics                     |
| `Get-Command *process*`          | `compgen -c \| grep process` | Find commands matching a pattern       |
| `Get-Command -Verb Get`          | —                            | List all commands with a specific verb |
| `Get-Alias ls`                   | `type ls`                    | Show what an alias maps to             |
| `Update-Help`                    | —                            | Update local help files                |

```powershell
# Get help for a specific parameter
Get-Help Get-Process -Parameter Name

# Show online help in browser
Get-Help Get-Process -Online
```

---

## Navigation & File System

|PowerShell|Linux|Description|
|---|---|---|
|`Get-Location` / `pwd`|`pwd`|Print current directory|
|`Set-Location C:\Users` / `cd C:\Users`|`cd /home`|Change directory|
|`Set-Location ..` / `cd ..`|`cd ..`|Go up one level|
|`Set-Location ~` / `cd ~`|`cd ~`|Go to home directory|
|`Set-Location -`|`cd -`|Go to previous directory|
|`Get-ChildItem` / `ls` / `dir`|`ls`|List directory contents|
|`Get-ChildItem -Force`|`ls -a`|Include hidden items|
|`Get-ChildItem -Recurse`|`ls -R`|List recursively|
|`Get-ChildItem -l`|`ls -l`|Long format listing|
|`Push-Location C:\temp`|`pushd /tmp`|Push directory onto stack|
|`Pop-Location`|`popd`|Pop directory from stack|

```powershell
# List only .txt files
Get-ChildItem *.txt

# List files sorted by size (largest first)
Get-ChildItem | Sort-Object Length -Descending

# List files modified in the last 7 days
Get-ChildItem | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) }

# Bash equivalent:
# find . -mtime -7 -type f
```

---

## File & Directory Operations

|PowerShell|Linux|Description|
|---|---|---|
|`New-Item file.txt -ItemType File`|`touch file.txt`|Create a new file|
|`New-Item myfolder -ItemType Directory` / `mkdir myfolder`|`mkdir myfolder`|Create a directory|
|`New-Item -Path a/b/c -ItemType Directory -Force`|`mkdir -p a/b/c`|Create nested directories|
|`Copy-Item file.txt backup.txt` / `cp`|`cp file.txt backup.txt`|Copy a file|
|`Copy-Item folder -Recurse dest`|`cp -r folder dest`|Copy directory recursively|
|`Move-Item file.txt C:\dest` / `mv`|`mv file.txt /dest`|Move/rename a file|
|`Remove-Item file.txt` / `rm`|`rm file.txt`|Delete a file|
|`Remove-Item folder -Recurse`|`rm -rf folder`|Delete a directory recursively|
|`Remove-Item folder -Recurse -Force`|`rm -rf folder`|Force delete|
|`Rename-Item old.txt new.txt`|`mv old.txt new.txt`|Rename a file|
|`Test-Path C:\file.txt`|`test -e file.txt`|Check if path exists|

```powershell
# Create file with content
"Hello World" | Out-File hello.txt
# Linux: echo "Hello World" > hello.txt

# Copy multiple files by extension
Copy-Item *.log C:\Logs\

# Delete files older than 30 days
Get-ChildItem C:\Logs | Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } | Remove-Item

# Bash equivalent:
# find /logs -mtime +30 -type f -delete
```

---

## Viewing & Editing Files

|PowerShell|Linux|Description|
|---|---|---|
|`Get-Content file.txt` / `cat`|`cat file.txt`|Display file contents|
|`Get-Content file.txt -Head 10`|`head -n 10 file.txt`|Show first N lines|
|`Get-Content file.txt -Tail 10`|`tail -n 10 file.txt`|Show last N lines|
|`Get-Content file.txt -Wait`|`tail -f file.txt`|Follow file (live)|
|`more file.txt`|`less file.txt`|Page through content|
|`notepad file.txt`|`nano file.txt`|Open in text editor|
|`Set-Content file.txt "text"`|`echo "text" > file.txt`|Overwrite file content|
|`Add-Content file.txt "text"`|`echo "text" >> file.txt`|Append to file|
|`Clear-Content file.txt`|`> file.txt` or `truncate -s 0 file.txt`|Clear file contents|
|`(Get-Content file.txt).Count`|`wc -l file.txt`|Count lines|
|`Get-Item file.txt \| Select-Object Length`|`wc -c file.txt`|File size in bytes|

```powershell
# View specific lines (e.g., lines 5-10)
Get-Content file.txt | Select-Object -Skip 4 -First 6
# Linux: sed -n '5,10p' file.txt

# Replace text in a file
(Get-Content file.txt) -replace 'foo', 'bar' | Set-Content file.txt
# Linux: sed -i 's/foo/bar/g' file.txt

# Compare two files
Compare-Object (Get-Content a.txt) (Get-Content b.txt)
# Linux: diff a.txt b.txt
```

---

## Searching & Filtering

|PowerShell|Linux|Description|
|---|---|---|
|`Select-String "error" file.txt`|`grep "error" file.txt`|Search for pattern in file|
|`Select-String "error" *.log`|`grep "error" *.log`|Search across multiple files|
|`Select-String -Pattern "error" -CaseSensitive`|`grep -c "error"`|Case-sensitive search|
|`Select-String "error" file.txt -NotMatch`|`grep -v "error" file.txt`|Lines NOT matching|
|`Select-String "err.*r" -Pattern file.txt`|`grep -E "err.*r" file.txt`|Regex search|
|`Get-ChildItem -Recurse \| Select-String "todo"`|`grep -r "todo" .`|Recursive content search|
|`Where-Object { $_.Name -like "*.log" }`|`find . -name "*.log"`|Filter by property|
|`Find files: Get-ChildItem -Recurse -Filter "*.log"`|`find . -name "*.log"`|Find files by name|

```powershell
# Search and show line numbers
Select-String "error" app.log | Select-Object LineNumber, Line

# Find files larger than 100MB
Get-ChildItem -Recurse | Where-Object { $_.Length -gt 100MB }
# Linux: find . -size +100M

# Find files by name recursively
Get-ChildItem -Recurse -Filter "*.config"
# Linux: find . -name "*.config"

# Grep-like with context lines (not built-in, pipe trick)
Select-String "error" app.log -Context 2,2
```

---

## Piping & Redirection

| PowerShell                         | Linux                 | Description               |
| ---------------------------------- | --------------------- | ------------------------- |
| `cmd1 \| cmd2`                     | `cmd1 \| cmd2`        | Pipe objects/text         |
| `cmd > file.txt`                   | `cmd > file.txt`      | Redirect output to file   |
| `cmd >> file.txt`                  | `cmd >> file.txt`     | Append output to file     |
| `cmd 2> errors.txt`                | `cmd 2> errors.txt`   | Redirect stderr           |
| `cmd *> all.txt`                   | `cmd > all.txt 2>&1`  | Redirect all streams      |
| `cmd \| Out-Null`                  | `cmd > /dev/null`     | Discard output            |
| `cmd \| Tee-Object file.txt`       | `cmd \| tee file.txt` | Output to screen AND file |
| `cmd \| Out-File file.txt`         | `cmd > file.txt`      | Write to file             |
| `cmd \| Out-File file.txt -Append` | `cmd >> file.txt`     | Append to file            |

```powershell
# PowerShell pipes OBJECTS, not text — this is a key difference from Linux
Get-Process | Where-Object { $_.CPU -gt 10 } | Sort-Object CPU -Descending | Select-Object -First 5

# Linux equivalent (text-based):
# ps aux --sort=-%cpu | head -6

# Count items coming through a pipe
Get-ChildItem *.txt | Measure-Object
# Linux: ls *.txt | wc -l

# Select specific properties from piped objects
Get-Process | Select-Object Name, CPU, WorkingSet | Format-Table
```

> **Key difference:** PowerShell pipes **objects** with properties and methods. Linux pipes **text streams**. This makes PowerShell filtering more powerful but different in philosophy.

---

## Variables

|PowerShell|Linux (bash)|Description|
|---|---|---|
|`$name = "Alice"`|`name="Alice"`|Assign a variable|
|`$num = 42`|`num=42`|Assign a number|
|`$name`|`echo $name`|Print variable value|
|`"Hello $name"`|`echo "Hello $name"`|String interpolation|
|`Remove-Variable name`|`unset name`|Delete a variable|
|`Get-Variable`|`set`|List all variables|
|`$null`|`""` / `unset`|Null/empty value|
|`[int]$x = 5`|—|Typed variable|

```powershell
# Multiple assignment
$a, $b, $c = 1, 2, 3

# Swap variables
$a, $b = $b, $a

# Automatic variables
$PSVersionTable        # PowerShell version info
$HOME                  # Home directory
$PWD                   # Current directory
$_  or  $PSItem        # Current pipeline object
$?                     # Last command success (True/False)
$LASTEXITCODE          # Exit code of last native command
$Error[0]              # Most recent error
$Args                  # Script arguments array

# bash equivalents:
# $?  → exit code (0=success), $HOME, $PWD, $@
```

---

## Strings

|PowerShell|Linux (bash)|Description|
|---|---|---|
|`"Hello" + " World"`|`echo "Hello World"`|Concatenate|
|`"Ha" * 3`|`printf 'Ha%.0s' {1..3}`|Repeat string|
|`"hello".Length`|`echo ${#str}` / `expr length "$str"`|String length|
|`"hello".ToUpper()`|`echo ${str^^}`|Uppercase|
|`"HELLO".ToLower()`|`echo ${str,,}`|Lowercase|
|`" hi ".Trim()`|`echo $str \| xargs`|Trim whitespace|
|`"hello"[0]`|`echo ${str:0:1}`|Character at index|
|`"hello".Substring(1,3)`|`echo ${str:1:3}`|Substring|
|`"hello".Replace("l","r")`|`echo ${str//l/r}`|Replace|
|`"hello".Contains("ell")`|`[[ "$str" == *ell* ]]`|Contains|
|`"hello".StartsWith("he")`|`[[ "$str" == he* ]]`|Starts with|
|`"hello".Split(",")`|`IFS=',' read -a arr <<< "$str"`|Split string|
|`"a","b","c" -join "-"`|`arr=("a" "b" "c"); IFS='-'; echo "${arr[*]}"`|Join array|
|`$str -match "pattern"`|`[[ "$str" =~ pattern ]]`|Regex match|

```powershell
# Here-string (multiline)
$text = @"
Line 1
Line 2
Line 3
"@
# Linux: text=$(cat <<EOF
# Line 1
# ...
# EOF)

# Format strings
"Name: {0}, Age: {1}" -f "Alice", 30

# String interpolation vs literal
$x = 5
"Value is $x"    # → "Value is 5"    (interpolated)
'Value is $x'    # → "Value is $x"   (literal, single quotes)

# Test if string is empty
if ([string]::IsNullOrEmpty($str)) { "Empty!" }
# bash: if [ -z "$str" ]; then echo "Empty!"; fi
```

---

## Arrays & Collections

|PowerShell|Linux (bash)|Description|
|---|---|---|
|`$arr = 1, 2, 3`|`arr=(1 2 3)`|Create array|
|`$arr = @(1, 2, 3)`|`arr=(1 2 3)`|Explicit array|
|`$arr[0]`|`${arr[0]}`|Access element|
|`$arr[-1]`|`${arr[-1]}`|Last element|
|`$arr[1..3]`|`${arr[@]:1:3}`|Slice|
|`$arr.Count`|`${#arr[@]}`|Array length|
|`$arr += 4`|`arr+=(4)`|Append element|
|`$arr -contains 2`|`[[ " ${arr[@]} " =~ " 2 " ]]`|Check if contains|
|`$arr \| Sort-Object`|`IFS=$'\n' sorted=($(sort <<< "${arr[*]}"))`|Sort array|
|`$arr \| Select-Object -Unique`|`printf '%s\n' "${arr[@]}" \| sort -u`|Unique values|

```powershell
# Create and manipulate arrays
$fruits = @("apple", "banana", "cherry")
$fruits += "date"                          # Add item
$fruits = $fruits | Where-Object { $_ -ne "banana" }  # Remove item

# Iterate
foreach ($fruit in $fruits) {
    Write-Host $fruit
}

# Map / transform (ForEach-Object)
$lengths = $fruits | ForEach-Object { $_.Length }

# Filter
$long = $fruits | Where-Object { $_.Length -gt 5 }

# bash array iteration:
# for fruit in "${fruits[@]}"; do echo "$fruit"; done

# ArrayList (mutable, faster for large collections)
$list = [System.Collections.ArrayList]@()
$list.Add("item1")
$list.Remove("item1")
```

---

## Hash Tables (Dictionaries)

|PowerShell|Linux (bash 4+)|Description|
|---|---|---|
|`$h = @{key="val"}`|`declare -A h; h[key]="val"`|Create hash table|
|`$h["key"]` or `$h.key`|`${h[key]}`|Access value|
|`$h["new"] = "val"`|`h[new]="val"`|Add/update entry|
|`$h.Remove("key")`|`unset h[key]`|Remove entry|
|`$h.Keys`|`${!h[@]}`|Get all keys|
|`$h.Values`|`${h[@]}`|Get all values|
|`$h.ContainsKey("k")`|`[[ -v h[k] ]]`|Check key exists|
|`$h.Count`|`${#h[@]}`|Number of entries|

```powershell
# Create a hash table
$person = @{
    Name = "Alice"
    Age  = 30
    City = "Brisbane"
}

# Access
$person.Name       # → "Alice"
$person["Age"]     # → 30

# Iterate
foreach ($key in $person.Keys) {
    Write-Host "$key = $($person[$key])"
}

# Ordered hash table (preserves insertion order)
$ordered = [ordered]@{ a = 1; b = 2; c = 3 }

# Nested hash table
$config = @{
    DB = @{ Host = "localhost"; Port = 5432 }
    App = @{ Debug = $true }
}
$config.DB.Host    # → "localhost"
```

---

## Conditionals

```powershell
# if / elseif / else
if ($x -gt 10) {
    "Greater than 10"
} elseif ($x -eq 10) {
    "Exactly 10"
} else {
    "Less than 10"
}

# bash equivalent:
# if [ $x -gt 10 ]; then ... elif [ $x -eq 10 ]; then ... else ... fi

# Comparison operators
# PowerShell   Linux/bash     Meaning
# -eq          ==  -eq        Equal
# -ne          !=  -ne        Not equal
# -gt          >   -gt        Greater than
# -lt          <   -lt        Less than
# -ge          >=  -ge        Greater or equal
# -le          <=  -le        Less or equal
# -like        (glob)         Wildcard match  ("file*")
# -match       =~             Regex match
# -not / !     !              Logical NOT
# -and         &&             Logical AND
# -or          ||             Logical OR

# Switch statement
switch ($day) {
    "Monday"  { "Start of work week" }
    "Friday"  { "End of work week" }
    "Saturday" { "Weekend!" }
    "Sunday"  { "Weekend!" }
    default   { "Midweek" }
}

# Switch with regex
switch -Regex ($input) {
    "^\d+"  { "Starts with number" }
    "^[A-Z]" { "Starts with uppercase" }
}

# Ternary-style (PowerShell 7+)
$result = $x -gt 0 ? "Positive" : "Non-positive"
# bash: result=$([ $x -gt 0 ] && echo "Positive" || echo "Non-positive")
```

---

## Loops

```powershell
# foreach loop
foreach ($item in $collection) {
    Write-Host $item
}
# bash: for item in "${arr[@]}"; do echo "$item"; done

# for loop
for ($i = 0; $i -lt 5; $i++) {
    Write-Host $i
}
# bash: for ((i=0; i<5; i++)); do echo $i; done

# while loop
$i = 0
while ($i -lt 5) {
    Write-Host $i
    $i++
}
# bash: while [ $i -lt 5 ]; do echo $i; ((i++)); done

# do-while
do {
    $input = Read-Host "Enter value"
} while ($input -ne "quit")

# do-until (loop until condition is TRUE)
do {
    $input = Read-Host "Enter value"
} until ($input -eq "quit")

# ForEach-Object (pipeline loop)
1..5 | ForEach-Object { $_ * 2 }
# bash: for i in {1..5}; do echo $((i*2)); done

# Loop control
foreach ($i in 1..10) {
    if ($i -eq 3) { continue }   # Skip (bash: continue)
    if ($i -eq 7) { break }      # Exit loop (bash: break)
    Write-Host $i
}

# Ranges
1..10           # → 1,2,3,...,10
10..1           # → 10,9,8,...,1
'a'..'e'        # → a,b,c,d,e
```

---

## Functions

```powershell
# Basic function
function Say-Hello {
    Write-Host "Hello!"
}
Say-Hello

# Function with parameters
function Greet {
    param(
        [string]$Name,
        [int]$Times = 1      # Default value
    )
    for ($i = 0; $i -lt $Times; $i++) {
        Write-Host "Hello, $Name!"
    }
}
Greet -Name "Alice" -Times 3

# Return values
function Add-Numbers {
    param([int]$a, [int]$b)
    return $a + $b
}
$sum = Add-Numbers -a 5 -b 3

# Advanced function (with pipeline support)
function Get-BigFiles {
    [CmdletBinding()]
    param(
        [string]$Path = ".",
        [long]$MinSizeMB = 10
    )
    Get-ChildItem -Path $Path -Recurse -File |
        Where-Object { $_.Length -gt ($MinSizeMB * 1MB) }
}

# bash function equivalent:
# greet() {
#     local name=$1
#     echo "Hello, $name!"
# }
# greet "Alice"

# Validate parameters
function Set-Age {
    param(
        [ValidateRange(0,150)]
        [int]$Age
    )
    Write-Host "Age set to $Age"
}
```

---

## Error Handling

```powershell
# Try / Catch / Finally
try {
    Get-Item "C:\nonexistent" -ErrorAction Stop
} catch [System.IO.FileNotFoundException] {
    Write-Host "File not found: $($_.Exception.Message)"
} catch {
    Write-Host "Unexpected error: $($_.Exception.Message)"
} finally {
    Write-Host "This always runs"
}

# bash equivalent:
# command || { echo "Error occurred"; exit 1; }

# Error action preferences
Get-Item "missing" -ErrorAction SilentlyContinue  # Suppress error
Get-Item "missing" -ErrorAction Stop              # Treat as terminating
Get-Item "missing" -ErrorAction Ignore            # Ignore completely

# Global preference
$ErrorActionPreference = "Stop"    # Make all errors terminating
$ErrorActionPreference = "Continue" # Default

# Check last error
if ($?) { "Success" } else { "Failed" }

# Throw custom error
throw "Something went wrong"
throw [System.ArgumentException]"Invalid argument"

# Trap (older style)
trap {
    Write-Host "Error: $($_.Exception.Message)"
    continue
}
```

---

## Processes

|PowerShell|Linux|Description|
|---|---|---|
|`Get-Process`|`ps aux`|List running processes|
|`Get-Process -Name chrome`|`ps aux \| grep chrome`|Find by name|
|`Stop-Process -Name chrome`|`kill $(pgrep chrome)`|Kill by name|
|`Stop-Process -Id 1234`|`kill 1234`|Kill by PID|
|`Stop-Process -Id 1234 -Force`|`kill -9 1234`|Force kill|
|`Start-Process notepad`|`notepad &`|Start a process|
|`Start-Process cmd -Wait`|—|Start and wait|
|`Get-Process \| Sort-Object CPU -Desc`|`ps aux --sort=-%cpu`|Sort by CPU|
|`Get-Process \| Sort-Object WS -Desc`|`ps aux --sort=-%mem`|Sort by memory|
|`Wait-Process -Id 1234`|`wait 1234`|Wait for process|

```powershell
# Get top 5 CPU-consuming processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU, Id

# Start a process with arguments
Start-Process "python" -ArgumentList "script.py", "--verbose"

# Run command and capture output
$output = & python script.py 2>&1

# Check if a process is running
if (Get-Process -Name "nginx" -ErrorAction SilentlyContinue) {
    "nginx is running"
}
# bash: if pgrep -x "nginx" > /dev/null; then echo "running"; fi
```

---

## Networking

|PowerShell|Linux|Description|
|---|---|---|
|`Test-Connection google.com`|`ping google.com`|Ping a host|
|`Test-NetConnection google.com -Port 443`|`nc -zv google.com 443`|Test TCP port|
|`Resolve-DnsName google.com`|`dig google.com` / `nslookup google.com`|DNS lookup|
|`Get-NetIPAddress`|`ip addr` / `ifconfig`|Show IP addresses|
|`Get-NetAdapter`|`ip link`|Show network adapters|
|`Get-NetTCPConnection`|`ss -tuln` / `netstat -tuln`|Show connections|
|`Invoke-WebRequest https://url`|`curl https://url`|HTTP GET request|
|`Invoke-RestMethod https://api`|`curl -s https://api \| jq`|REST API call|
|`Get-NetRoute`|`ip route` / `route -n`|Show routing table|

```powershell
# Download a file
Invoke-WebRequest -Uri "https://example.com/file.zip" -OutFile "file.zip"
# Linux: curl -O https://example.com/file.zip
#         wget https://example.com/file.zip

# Make REST API call and parse JSON
$response = Invoke-RestMethod -Uri "https://api.github.com/users/octocat"
$response.name   # Access JSON properties directly

# Linux: curl -s https://api.github.com/users/octocat | jq '.name'

# Show listening ports
Get-NetTCPConnection -State Listen | Select-Object LocalPort, State | Sort-Object LocalPort
# Linux: ss -tlnp

# Test connectivity
Test-Connection -ComputerName google.com -Count 4 -Quiet
# → True / False

# Get public IP
(Invoke-RestMethod https://api.ipify.org?format=json).ip
# Linux: curl -s https://api.ipify.org
```

---

## Users & Permissions

|PowerShell|Linux|Description|
|---|---|---|
|`whoami`|`whoami`|Current user|
|`$env:USERNAME`|`$USER`|Current username|
|`Get-LocalUser`|`cat /etc/passwd`|List local users|
|`New-LocalUser "bob"`|`useradd bob`|Create local user|
|`Set-LocalUser -Name "bob" -Password ...`|`passwd bob`|Set password|
|`Remove-LocalUser "bob"`|`userdel bob`|Delete user|
|`Get-LocalGroup`|`cat /etc/group`|List groups|
|`Add-LocalGroupMember -Group "Admins" -Member "bob"`|`usermod -aG sudo bob`|Add to group|
|`Get-Acl file.txt`|`ls -l file.txt`|Get file permissions|
|`Set-Acl file.txt $acl`|`chmod`/`chown`|Set permissions|

```powershell
# Check if running as Administrator
$isAdmin = ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
# Linux: [ "$EUID" -eq 0 ] && echo "root"

# Get file permissions
Get-Acl C:\file.txt | Format-List

# Grant full control to a user
$acl = Get-Acl "C:\folder"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("alice","FullControl","Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\folder" $acl
```

---

## Services

|PowerShell|Linux (systemd)|Description|
|---|---|---|
|`Get-Service`|`systemctl list-units`|List all services|
|`Get-Service -Name wuauserv`|`systemctl status wuauserv`|Status of a service|
|`Start-Service -Name wuauserv`|`systemctl start wuauserv`|Start a service|
|`Stop-Service -Name wuauserv`|`systemctl stop wuauserv`|Stop a service|
|`Restart-Service -Name wuauserv`|`systemctl restart wuauserv`|Restart a service|
|`Set-Service -Name svc -StartupType Automatic`|`systemctl enable svc`|Enable auto-start|
|`Set-Service -Name svc -StartupType Disabled`|`systemctl disable svc`|Disable auto-start|

```powershell
# List running services
Get-Service | Where-Object { $_.Status -eq "Running" }
# Linux: systemctl list-units --type=service --state=running

# Wait for a service to start
$timeout = 30
$elapsed = 0
while ((Get-Service -Name "MyService").Status -ne "Running" -and $elapsed -lt $timeout) {
    Start-Sleep -Seconds 2
    $elapsed += 2
}
```

---

## Registry

_(Windows-only — no Linux equivalent)_

```powershell
# Navigate registry like a file system
Set-Location HKLM:\SOFTWARE\Microsoft
Get-ChildItem    # List registry keys

# Read a value
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" -Name "ProductName"

# Set a value
Set-ItemProperty -Path "HKCU:\Software\MyApp" -Name "Theme" -Value "Dark"

# Create a key
New-Item -Path "HKCU:\Software\MyApp" -Force

# Delete a value
Remove-ItemProperty -Path "HKCU:\Software\MyApp" -Name "Theme"

# Delete a key
Remove-Item -Path "HKCU:\Software\MyApp" -Recurse
```

---

## Environment Variables

|PowerShell|Linux|Description|
|---|---|---|
|`$env:PATH`|`$PATH`|Access env variable|
|`$env:USERNAME`|`$USER`|Username|
|`$env:COMPUTERNAME`|`$HOSTNAME`|Hostname|
|`$env:TEMP`|`$TMPDIR` / `/tmp`|Temp directory|
|`$env:USERPROFILE`|`$HOME`|Home directory|
|`[System.Environment]::GetEnvironmentVariables()`|`env` / `printenv`|List all env vars|
|`$env:MYVAR = "value"`|`export MYVAR="value"`|Set env variable|
|`Remove-Item Env:\MYVAR`|`unset MYVAR`|Remove env variable|

```powershell
# Add to PATH for current session
$env:PATH += ";C:\MyTools"
# Linux: export PATH="$PATH:/mytools"

# Set permanently (user scope)
[System.Environment]::SetEnvironmentVariable("MYVAR", "value", "User")

# Set permanently (system scope — requires admin)
[System.Environment]::SetEnvironmentVariable("MYVAR", "value", "Machine")

# bash equivalent (add to ~/.bashrc or ~/.bash_profile):
# echo 'export MYVAR="value"' >> ~/.bashrc

# List env vars matching a pattern
Get-ChildItem Env: | Where-Object { $_.Name -like "*JAVA*" }
# Linux: env | grep JAVA
```

---

## Scripting Essentials

```powershell
# ── Script parameters ───────────────────────────────────────────────
param(
    [Parameter(Mandatory=$true)]
    [string]$InputFile,

    [Parameter(Mandatory=$false)]
    [string]$OutputDir = "C:\Output",

    [switch]$Verbose   # boolean flag
)
# Linux: positional: $1, $2; flags: getopts

# ── Execution policy ────────────────────────────────────────────────
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned   # Allow local scripts
Set-ExecutionPolicy Bypass -Scope Process  # For current session only

# ── Running scripts ─────────────────────────────────────────────────
.\script.ps1                       # Run in current scope
& "C:\path\to script.ps1"          # Run with spaces in path
. .\script.ps1                     # Dot-source (imports into scope)
# Linux: bash script.sh  OR  ./script.sh  OR  source script.sh

# ── Write output ────────────────────────────────────────────────────
Write-Host "To screen only"
Write-Output "To pipeline"
Write-Verbose "Only with -Verbose"    # Requires [CmdletBinding()]
Write-Warning "Warning message"
Write-Error "Error message"
Write-Debug "Debug message"
# Linux: echo, printf, >&2 for stderr

# ── User input ──────────────────────────────────────────────────────
$name = Read-Host "Enter your name"
$pass = Read-Host "Password" -AsSecureString
# Linux: read -p "Enter name: " name; read -s -p "Password: " pass

# ── Measure execution time ───────────────────────────────────────────
Measure-Command { Get-ChildItem -Recurse }
# Linux: time find . -type f

# ── Comment styles ──────────────────────────────────────────────────
# Single-line comment (same as bash)
<#
    Multi-line comment block
    (bash uses : << 'EOF' ... EOF or just # each line)
#>

# ── Modules ─────────────────────────────────────────────────────────
Get-Module -ListAvailable          # List installed modules
Import-Module ActiveDirectory      # Import a module
Install-Module -Name Pester        # Install from PSGallery
Find-Module -Name "*azure*"        # Search PSGallery
# Linux: apt/pip/npm install equivalent
```

---

## Useful One-Liners

```powershell
# System uptime
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
# Linux: uptime

# Disk usage
Get-PSDrive C | Select-Object Used, Free
# Linux: df -h

# Top 10 largest files in current dir
Get-ChildItem -Recurse -File | Sort-Object Length -Descending | Select-Object -First 10 Name, @{N="MB";E={[math]::Round($_.Length/1MB,2)}}
# Linux: du -ah . | sort -rh | head -10

# Count files by extension
Get-ChildItem -Recurse -File | Group-Object Extension | Sort-Object Count -Descending
# Linux: find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Export process list to CSV
Get-Process | Export-Csv processes.csv -NoTypeInformation
# Linux: ps aux > processes.txt

# Check open ports
Get-NetTCPConnection -State Listen | Sort-Object LocalPort
# Linux: ss -tlnp

# Generate random password
[System.Web.Security.Membership]::GeneratePassword(16, 4)
# Linux: openssl rand -base64 16

# Get system info
Get-ComputerInfo | Select-Object CsName, OsName, OsArchitecture, TotalPhysicalMemory
# Linux: uname -a && free -h

# Kill all processes by name
Get-Process chrome | Stop-Process -Force
# Linux: killall chrome

# Encode/decode Base64
$encoded = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("hello"))
$decoded = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($encoded))
# Linux: echo "hello" | base64; echo "aGVsbG8=" | base64 -d

# Extract zip file
Expand-Archive file.zip -DestinationPath ./output
# Linux: unzip file.zip -d ./output

# Compress to zip
Compress-Archive -Path ./folder -DestinationPath archive.zip
# Linux: zip -r archive.zip ./folder

# HTTP GET and save
Invoke-WebRequest "https://example.com/file" -OutFile "file.bin"
# Linux: curl -L -o file.bin https://example.com/file

# Show calendar
[System.Globalization.CultureInfo]::CurrentCulture.Calendar
# Linux: cal

# Hash a file
Get-FileHash file.txt -Algorithm SHA256
# Linux: sha256sum file.txt

# Find duplicate files by hash
Get-ChildItem -Recurse -File | Get-FileHash | Group-Object Hash | Where-Object { $_.Count -gt 1 }
```

---

## Quick Reference: Operator Comparison

|PowerShell|bash|Meaning|
|---|---|---|
|`-eq`|`==` or `-eq`|Equal (string or int)|
|`-ne`|`!=` or `-ne`|Not equal|
|`-gt`|`-gt` or `>`|Greater than|
|`-lt`|`-lt` or `<`|Less than|
|`-ge`|`-ge` or `>=`|Greater or equal|
|`-le`|`-le` or `<=`|Less or equal|
|`-like "a*"`|`[[ $x == a* ]]`|Wildcard match|
|`-notlike "a*"`|`[[ $x != a* ]]`|Wildcard not match|
|`-match "regex"`|`[[ $x =~ regex ]]`|Regex match|
|`-notmatch "re"`|`! [[ $x =~ re ]]`|Regex not match|
|`-contains`|`grep -q`/ `[[ " ${arr[@]} " =~ " $v " ]]`|Array contains value|
|`-in`|—|Value in array|
|`-and`|`&&`|Logical AND|
|`-or`|`\|`|Logical OR|
|`-not` / `!`|`!`|Logical NOT|
|`-band`|`&`|Bitwise AND|
|`-bor`|`\|`|Bitwise OR|
|`-bxor`|`^`|Bitwise XOR|

---

# `Where-Object` Cheat Sheet

## Syntax Forms

```powershell
# Script block (most flexible)
... | Where-Object { <condition> }

# Comparison (simple property checks, no $_ needed)
... | Where-Object <Property> <Operator> <Value>

# Alias: ? or where
... | ? { $_.Name -eq "foo" }
```

---

## Comparison Operators

|Operator|Meaning|Example|
|---|---|---|
|`-eq`|Equal|`{ $_.Status -eq "Running" }`|
|`-ne`|Not equal|`{ $_.Status -ne "Stopped" }`|
|`-gt`|Greater than|`{ $_.CPU -gt 100 }`|
|`-ge`|Greater than or equal|`{ $_.CPU -ge 50 }`|
|`-lt`|Less than|`{ $_.Count -lt 10 }`|
|`-le`|Less than or equal|`{ $_.Count -le 5 }`|
|`-like`|Wildcard match|`{ $_.Name -like "*ssh*" }`|
|`-notlike`|Wildcard non-match|`{ $_.Name -notlike "*tmp*" }`|
|`-match`|Regex match|`{ $_.Name -match "^win" }`|
|`-notmatch`|Regex non-match|`{ $_.Name -notmatch "test" }`|
|`-contains`|Collection contains value|`{ $_.Tags -contains "web" }`|
|`-notcontains`|Collection doesn't contain value|`{ $_.Tags -notcontains "old" }`|
|`-in`|Value in collection|`{ $_.Name -in @("svc1","svc2") }`|
|`-notin`|Value not in collection|`{ $_.Name -notin @("svc1","svc2") }`|

> Prefix any operator with `i` for case-insensitive (default) or `c` for case-sensitive: `-ceq`, `-clike`, `-cmatch`, etc.

---

## Logical Operators

```powershell
# AND
... | Where-Object { $_.Status -eq "Running" -and $_.CPU -gt 10 }

# OR
... | Where-Object { $_.Name -like "*sql*" -or $_.Name -like "*db*" }

# NOT
... | Where-Object { -not ($_.Name -like "*tmp*") }
# or
... | Where-Object { !($_.Name -like "*tmp*") }
```

---

## Null / Empty Checks

```powershell
# Is null
... | Where-Object { $_.Description -eq $null }
... | Where-Object { $null -eq $_.Description }   # safer form

# Is not null
... | Where-Object { $_.Description -ne $null }

# Is null or empty string
... | Where-Object { [string]::IsNullOrEmpty($_.Description) }

# Property exists and has a value
... | Where-Object { $_.PSObject.Properties['Name'] -and $_.Name }
```

---

## Type / Boolean Checks

```powershell
# Boolean true
... | Where-Object { $_.Enabled }
... | Where-Object Enabled                        # shorthand

# Boolean false
... | Where-Object { -not $_.Enabled }

# Type check
... | Where-Object { $_.Value -is [int] }
... | Where-Object { $_.Value -isnot [string] }
```

---

## Common Examples

```powershell
# Running services
Get-Service | Where-Object { $_.Status -eq "Running" }

# Processes using more than 200MB
Get-Process | Where-Object { $_.WorkingSet -gt 200MB }

# Files modified in the last 7 days
Get-ChildItem | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) }

# Event logs containing "OpenSSH"
Get-WinEvent -ListLog * | Where-Object { $_.LogName -like "*OpenSSH*" }

# Users in a specific OU (AD)
Get-ADUser -Filter * | Where-Object { $_.DistinguishedName -like "*OU=IT*" }

# Multiple conditions
Get-Process | Where-Object { $_.CPU -gt 10 -and $_.Name -notlike "idle" }

# Filter on calculated expression
Get-ChildItem | Where-Object { ($_.Length / 1MB) -gt 10 }
```

---

## Script Block vs Comparison Mode

```powershell
# Script block — supports complex logic
Get-Service | Where-Object { $_.Status -eq "Running" -and $_.StartType -eq "Automatic" }

# Comparison mode — clean for single conditions (PowerShell 3+)
Get-Service | Where-Object Status -eq "Running"

# ⚠ Comparison mode does NOT support -and / -or
# Use script block for multiple conditions
```

---

## Performance Tips

```powershell
# Filter EARLY in the pipeline — don't pipe everything then filter
# ✅ Good
Get-ChildItem -Path C:\ -Recurse -Filter "*.log"

# ⚠ Less efficient
Get-ChildItem -Path C:\ -Recurse | Where-Object { $_.Extension -eq ".log" }

# Use cmdlet-native parameters when available (-Filter, -Identity, etc.)
# Where-Object is best when no native filter exists
```

---

## Aliases Quick Reference

|Alias|Full Cmdlet|
|---|---|
|`?`|`Where-Object`|
|`where`|`Where-Object`|
|`%`|`ForEach-Object`|
|`$_`|Current pipeline object|

_PowerShell version: 5.1 (Windows built-in) and 7+ (cross-platform). Most examples work on both; PowerShell 7+ adds ternary operators, `ForEach-Object -Parallel`, null coalescing (`??`), and more._
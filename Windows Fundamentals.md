File system: [[NTFS]]

The Windows folder ( `C:\Windows` ) is traditionally known as the folder which contains the Windows operating system.

The system  environment variable for the Windows directory is `%windir%` 

# System32

The System32 folder holds the important files that are critical for the operating system.

![[Pasted image 20260512125716.png]]

# User Accounts

Can be one of two types:
1. Administrator - can make changes to the system: add users, delete users, modify groups, modify settings on the system, etc.
2. Standard User - can only make changes to folders/files attributed to the user & can't perform system-level changes, such as install programs.

## How to add/edit/delete users
1.  Click `Start` menu
2. Type `other user`
3. Click on the shortcut to the settings
4. Click on `Add someone else to this PC`

Alternatively: Run `lusrmgr.msc` on the command prompt

## User Account Control  (UAC)

Windows uses UAC to protect the system from unauthorised users gaining privileged access to the system.

How does UAC work? When a user with an account type of administrator logs into a system, the current session doesn't run with elevated permissions. When an operation requiring higher-level privileges needs to execute, the user will be prompted to confirm if they permit the operation to run.

![[Pasted image 20260512131655.png]]

# Settings and Control Panel

The primary locations to make changes

## Settings

![[Pasted image 20260512131756.png]]

## Control Panel

![[Pasted image 20260512131810.png]]

# Task Manager

Shortcut: `ctrl+shift+esc`

[Complete Guide to the Task Manager](https://www.howtogeek.com/405806/windows-task-manager-the-complete-guide/)

# System Configuration

`MSConfig` for advanced troubleshooting

![[Pasted image 20260512132830.png]]

## MSConfig Tabs:
- General - we can select what devices and services for Windows to load upon boot. The options are: **Normal**, **Diagnostic**, or **Selective**.
  ![[Pasted image 20260512133511.png]]
- Boot - we can define various boot options for the Operating System.
  ![[Pasted image 20260512133520.png]]
- Services - lists all services configured for the system regardless of their state (running or stopped).
  ![[Pasted image 20260512133526.png]]
- Startup - Microsoft advises using **Task Manager (**`taskmgr`**)** to manage (enable/disable) startup items. The System Configuration utility is **NOT** a startup management program.*
- Tools - Place where various Windows utilities are located
  ![[Pasted image 20260512133545.png]]

> *\*Windows Servers handle startup differently than Windows Clients. You will not see startup settings on a VM. The only reliable way to view user-level startup items is through the Startup folder itself. You can access it by pressing `Win + R`, which opens the Run Dialog, typing **`shell:startup`**, and then pressing Enter.

# Advanced System Settings

To access, search for `View advanced system settings` in your search bar and open it.

Windows uses a page file as an extra virtual memory space when the physical RAM becomes full. This helps to prevent slowdowns or application crashes when the system runs out of memory. You can view or modify the page file by navigating to the `Advanced` option at the top and clicking `Settings` under the `Performance` tab.

![[Pasted image 20260512133753.png]]

![[Pasted image 20260512133855.png]]

## Startup and Recovery

![[Pasted image 20260512134017.png]]
![[Pasted image 20260512134023.png]]


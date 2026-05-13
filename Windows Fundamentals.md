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

## Change UAC Settings

![[Pasted image 20260513184353.png]] 

- **Always notify:** This is the highest security. Windows notifies you whenever any apps or you yourself try to make changes, and the desktop dims (Secure Desktop). 
- **Notify for apps**: Windows notifies only when _apps_ try to make changes, but not when you change Windows settings. This option is enabled by default.
- **Notify without dimming:** Same as above (Notify for apps), but this time the screen does not dim. 
- **Never notify:** Notifications are turned off. Windows won’t warn you about any changes made by you or any apps.

## Computer Management

`compmgmt` utility has three primary sections: System Tools, Storage, and Services and Applications.

### System Tools
- Task Scheduler - create and manage common tasks that our computer will carry out automatically at the times we specify
- Event Viewer - allows us to view events that have occurred on the computer
  ![[Pasted image 20260513184738.png]]
- ![[Pasted image 20260513184743.png]]
- Shared Folders - where you will see a complete list of shares and folders shared that others can connect to
- Local Users and Groups - `lusrmgr.msc`
- Performance - `perfmon`
- Device Manager - allows us to view and configure the hardware, such as disabling any hardware attached to the computer.
### Storage
- Windows Server Backup
- Disk Management
### Services and Applications
- Routing and Remote Access
- Services
- WMI Control - controls the **Windows Management Instrumentation** service which allows scripting languages (such as VBScript or Windows Powershell) to manage Microsoft Windows personal computers and servers, both locally and remotely. Microsoft also provides a command-line interface to WMI called Windows Management Instrumentation Command-line (WMIC)

## System Information
System information or `msinfo32`gathers information about your computer and displays a comprehensive view of your hardware, system components, and software environment, which you can use to diagnose computer issues.

There are 3 sections:
- Hardware Resources
- Components
- Software Environment
	- Environment Variables

## Resource Monitor
Resource Monitor or `resmon` displays per-process and aggregate , memory, disk, and network usage information, in addition to providing details about which processes are using individual file handles and modules. Advanced filtering allows users to isolate the data related to one or more processes (either applications or services), start, stop, pause, and resume services, and close unresponsive applications from the user interface. It also includes a process analysis feature that can help identify deadlocked processes and file locking conflicts so that the user can attempt to resolve the conflict instead of closing an application and potentially losing data.

There are 4 sections:
- CPU
- Disk
- Network
- Memory

## Command Prompt
Command Prompt or `cmd`

[[command prompt |List of common command prompt commands]]

## Registry Editor

Registry editor or `regedit` is a central hierarchical database used to store information necessary to configure the system for one or more users, applications, and hardware devices.

# Windows Update

Windows Update is a service provided by Microsoft to provide security updates, feature enhancements, and patches for the Windows operating system and other Microsoft products, such as Microsoft Defender.

Updates are typically released on the 2nd Tuesday of each month. This day is called **Patch Tuesday**.

To access Windows Update, run `control /name Microsoft.WindowsUpdate` in command prompt. It can also be accessed in the Settings menu
![[Pasted image 20260513194430.png]]

# Windows Security

![[Pasted image 20260513194554.png]]

Protection Areas:
- Virus and threat protection
	- Current threats
	- Virus and threat protection settings
- Firewall and network protection
	- **Domain** - _The domain profile applies to networks where the host system can authenticate to a domain controller._ 
	- **Private** - _The private profile is a user-assigned profile and is used to designate private or home networks._
	- **Public** - _The default profile is the public profile, used to designate public networks such as Wi-Fi hotspots at coffee shops, airports, and other locations._
- App and browser control
- Device Security

# Volume Shadow Copy Service
Shadow Copy is a snapshot or point-in-time copy of the data

If VSS is enabled (**System Protection** turned on), you can perform the following tasks from within **advanced system settings**:
- **Create a restore point**
- **Perform system restore**
- **Configure restore settings**
- **Delete restore points**

![[Pasted image 20260513195702.png]]
![[Pasted image 20260513195706.png]]
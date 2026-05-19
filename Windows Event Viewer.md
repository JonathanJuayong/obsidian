Logs in WIndows are in `.evt` or `.evtx` extension

They typically can be found in `C:\Windows\System32\winevt\Logs`

# Elements of a Windows Event Log

- **System Logs:** Records events associated with the Operating System segments. They may include information about hardware changes, device drivers, system changes, and other activities related to the device.
- **Security Logs:** Records events connected to logon and logoff activities on a device. The system's audit policy specifies the events. The logs are an excellent source for analysts to investigate attempted or successful unauthorized activity.
- **Application Logs**: Records events related to applications installed on a system. The main pieces of information include application errors, events, and warnings.
- **Directory Service Events:** Active Directory changes and activities are recorded in these logs, mainly on domain controllers.
- **File Replication Service Events:** Records events associated with Windows Servers during the sharing of Group Policies and logon scripts to domain controllers, from where they may be accessed by the users through the client servers.
- **DNS Event Logs:** DNS servers use these logs to record domain events and to map out
- **Custom Logs:** Events are logged by applications that require custom data storage. This allows applications to control the log size or attach other parameters, such as ACLs, for security purposes.

More info can be found on [docs.microsoft.com (opens in new tab)](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-types)

There are three main ways of accessing these event logs within a Windows system:

1. **Event Viewer** (GUI-based application) - can be launched using `eventvwr.msc`
2. [[Wevtutil.exe]] (command-line tool)
3. [[Get-WinEvent]] (PowerShell cmdlet)




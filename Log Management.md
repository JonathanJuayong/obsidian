# Linux

Most distributions store the log files and directories in `/var/log`.

```shell-session
root@TryHackMe# sudo tree /var/log -d -L 2
/var/log
├── akmods
├── anaconda
├── audit
├── blivet-gui
├── chrony
├── cups
├── displaylink
├── gdm
├── glusterfs
├── httpd
├── journal
│   └── f29b4ed41359484da9b7d3bf3ec279ac
├── libvirt
│   ├── libxl
│   ├── lxc
│   └── qemu
├── ppp
├── private
├── qemu-ga
├── samba
│   └── old
├── speech-dispatcher
├── sssd
├── swtpm
│   └── libvirt
└── vmware
```

## Log Types

- **System logs**: These logs contain information about the general health and operation of the system.
- **Application logs**: These logs contain information about the specific applications running on the system.
- **Security logs**: These logs contain information about security-related events, such as login and failed authentication attempts.

`aureport --summary`
`aureport --failed`
`ausearch --message USER_LOGIN --success yes --interpret`
`ausearch --message USER_LOGIN --success no --interpret`

# Windows

- **System Logs**: This records activity associated with the system components, such as driver failure, resource conflict, and hardware issues. For IT professionals, they serve as sources of critical diagnostics information.
- **Application Logs**: This type concerns individual software living upon the system. When issues manifest around a specific application, such as failing to connect to a database or process-related bottlenecks, these logs come in handy to determine why the failure occurred.
- **Security Logs**: Specialised logs designed to track security events. They touch on events such as logon and logoff actions, user rights assignments, policy changes, and security-related aberrations. For security professionals, this often represents their first check when investigating a security incident.
- **Forwarded Events Logs**: These logs receive collected from other tertiary-tertiary computing environments. They act as collated reports, pulling from multiple sources into a centralised file. They are ideal for monitoring tasks and analysis in a networked environment, where you may need to assemble data from various places into a cohesive analysis.

# Windows vs Linux

| Feature                | Linux Logs                                    | Windows Logs                                     |
| ---------------------- | --------------------------------------------- | ------------------------------------------------ |
| Location               | `/var/log`                                    | `%SystemRoot%\System32\Logfiles`                 |
| Format                 | Syslog                                        | EventLog                                         |
| Logging levels         | Debug, Info, Notice, Warning, Error, Critical | Debug, Information, Warning, Error, Critical     |
| Tools for viewing logs | `tail`, `grep`, `less`                        | Event Viewer                                     |
| Advantages             | More flexible, easier to parse                | More user-friendly, more integrated with Windows |
| Disadvantages          | Can be less intuitive, less centralized       | Can be more difficult to troubleshoot            |

# Logging vs Monitoring

|                                               | Logging                                                                         | Monitoring                                                                                                        |
| --------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Primary Function**                          | To record system activities for later review                                    | To deliver real-time observation of system status                                                                 |
| **Error Detection**                           | After it hits, provides a data trail to backtrack the issue                     | Identify and notify irregularities as they occur                                                                  |
| **Process**                                   | A constant, passive process recording activities and system changes             | An active, ongoing process receiving alerts or warnings based on predefined triggers                              |
| **Typical Uses**                              | Error diagnostics post-issue, compliance proof, audit trails, forensic analysis | Daily operational tracking, preventative maintenance, bottlenecks detection, real-time functionality and security |
| **Timeliness of Notification and Inspection** | Used primarily for retrospective analysis of problems                           | Real-time reporting of potential issues                                                                           |
| **Key Objectives**                            | Error diagnosis, accountability and providing detailed context                  | Eradicate small issues from escalating and becoming larger problem                                                |

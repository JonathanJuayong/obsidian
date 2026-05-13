Logs are a record of events within a system. 

![[Pasted image 20260513202612.png]]
Logs can answer critical questions about an event, such as:
- **What** happened?
- **When** did it happen?
- **Where** did it happen?
- **Who** is responsible?
- **Were** their actions **successful**?
- **What** was the result of their action?

# Log Types

Most common log types:
- **Application Logs:** Messages about specific applications, including status, errors, warnings, etc.
- **Audit Logs:** Activities related to operational procedures crucial for regulatory compliance.
- **Security Logs:** Security events such as logins, permissions changes, activity, etc.
- **Server Logs:** Various logs a server generates, including system, event, error, and access logs.
- **System Logs:** Kernel activities, system errors, boot sequences, and hardware status.
- **Network Logs:** Network traffic, connections, and other network-related events.
- **Database Logs:** Activities within a database system, such as queries and updates.
- **Web Server Logs:** Requests processed by a web server, including URLs, response codes, etc.

# Log Formats

## 1. Semi-structured logs

May contain structured and unstructured data, with predictable components accommodating free-form text.

### Examples:

**Syslog
```shell-session
damianhall@WEBSRV-02:~/logs$ cat syslog.txt
May 31 12:34:56 WEBSRV-02 CRON[2342593]: (root) CMD ([ -x /etc/init.d/anacron ] && if [ ! -d /run/systemd/system ]; then /usr/sbin/invoke-rc.d anacron start >/dev/null; fi)
```


**Windows Event Log:
```shell-session
PS C:\WINDOWS\system32> Get-WinEvent -Path "C:\Windows\System32\winevt\Logs\Application.evtx"


   ProviderName: Microsoft-Windows-Security-SPP

TimeCreated                      Id LevelDisplayName Message
-----------                      -- ---------------- -------
31/05/2023 17:18:24           16384 Information      Successfully scheduled Software Protection service for re-start
31/05/2023 17:17:53           16394 Information      Offline downlevel migration succeeded.
```

## 2. Structured Logs

Following a strict and standardised format, these logs are conducive to parsing and analysis.

### Examples:

**Field Delimited Formats**
CSV and TSV are commonly used for this format

```shell-session
damianhall@WEBSRV-02:~/logs$ cat log.csv
"time","user","action","status","ip","uri"
"2023-05-31T12:34:56Z","adversary","GET",200,"34.253.159.159","http://gitlab.swiftspend.finance:80/"
```

**JavaScript Object Notation (JSON)**
```shell-session
damianhall@WEBSRV-02:~/logs$ cat log.json
{"time": "2023-05-31T12:34:56Z", "user": "adversary", "action": "GET", "status": 200, "ip": "34.253.159.159", "uri": "http://gitlab.swiftspend.finance:80/"}
```

**W3C Extended Log Format**
Typically used by Microsoft Internet Information Services (IIS) Web Server.
```shell-session
damianhall@WEBSRV-02:~/logs$ cat elf.log
#Version: 1.0
#Fields: date time c-ip c-username s-ip s-port cs-method cs-uri-stem sc-status
31-May-2023 13:55:36 34.253.159.159 adversary 34.253.127.157 80 GET /explore 200
```

**eXtensible Markup Language (XML)**
```shell-session
damianhall@WEBSRV-02:~/logs$ cat log.xml
<log><time>2023-05-31T12:34:56Z</time><user>adversary</user><action>GET</action><status>200</status><ip>34.253.159.159</ip><url>https://gitlab.swiftspend.finance/</url></log>
```

## 3. Unstructured Logs

Comprising free-form text.

## Examples:

**NCSA Common Log Format (CLF)**
Typically used by the Apache HTTP Server by default.
```shell-session
damianhall@WEBSRV-02:~/logs$ cat clf.log
34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886
```

**NCSA Combined Log Format (Combined)**
An extension of CLF, adding fields like referrer and user agent. It is typically used by Nginx HTTP Server by default.
```shell-session
damianhall@WEBSRV-02:~/logs$ cat combined.log
34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886 "http://gitlab.swiftspend.finance/" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0"
```

# Log Standards

- **[**Common Event Expression (CEE):** (opens in new tab)](https://cee.mitre.org/)** This standard, developed by , provides a common structure for log data, making it easier to generate, transmit, store, and analyse logs.
- **[OWASP Logging Cheat Sheet: (opens in new tab)](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)** A guide for developers on building application logging mechanisms, especially related to security logging.
- **[Syslog Protocol: (opens in new tab)](https://datatracker.ietf.org/doc/html/rfc5424)** Syslog is a standard for message logging, allowing separation of the software that generates messages from the system that stores them and the software that reports and analyses them.
- **[NIST  Special Publication 800-92: (opens in new tab)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-92.pdf)** This publication guides computer security log management.
- **[Azure Monitor Logs: (opens in new tab)](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-platform-logs)** Guidelines for log monitoring on Microsoft Azure.
- **[Google Cloud Logging: (opens in new tab)](https://cloud.google.com/logging/docs)** Guidelines for logging on the Google Cloud Platform (GCP).
- **[Oracle Cloud Infrastructure Logging: (opens in new tab)](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)** Guidelines for logging on the Oracle Cloud Infrastructure (OCI).
- **[Virginia Tech - Standard for Information Technology Logging: (opens in new tab)](https://it.vt.edu/content/dam/it_vt_edu/policies/Standard_for_Information_Technology_Logging.pdf)** Sample log review and compliance guideline.

# Log Collection

## Process of collecting logs:

- **Identify Sources:** List all potential log sources, such as servers, databases, applications, and network devices.
- **Choose a Log Collector:** Opt for a suitable log collector tool or software that aligns with your infrastructure.
- **Configure Collection Parameters:** Ensure that time synchronisation is enabled through [[NTP]] to maintain accurate timelines, adjust settings to determine which events to log at what intervals, and prioritise based on importance.
- **Test Collection:** Once configured, run a test to ensure logs are appropriately collected from all sources.

# Log Management

- **Storage:** Decide on a secure storage solution, considering factors like retention period and accessibility.
- **Organisation:** Classify logs based on their source, type, or other criteria for easier access later.
- **Backup:** Regularly back up your logs to prevent data loss.
- **Review:** Periodically review logs to ensure they are correctly stored and categorised.

# Log Centralisation

- **Choose a Centralised System:** Opt for a system that consolidates logs from all sources, such as the Elastic Stack or Splunk.
- **Integrate Sources:** Connect all your log sources to this centralised system.
- **Set Up Monitoring:** Utilise tools that provide real-time monitoring and alerts for specified events.
- **Integration with Incident Management:** Ensure that your centralised system can integrate seamlessly with any incident management tools or protocols you have in place.

# Log Storage

Logs can be stored in various locations, such as the local system that generates them, a centralised repository, or cloud-based storage.

Things to consider:
- **Security Requirements:** Ensuring that logs are stored in compliance with organisational or regulatory security protocols.
- **Accessibility Needs:** How quickly and by whom the logs need to be accessed can influence the choice of storage.
- **Storage Capacity:** The volume of logs generated may require significant storage space, influencing the choice of storage solution.
- **Cost Considerations:** The budget for log storage may dictate the choice between cloud-based or local solutions.
- **Compliance Regulations:** Specific industry regulations governing log storage can affect the choice of storage.
- **Retention Policies:** The required retention time and ease of retrieval can affect the decision-making process.
- **Disaster Recovery Plans:** Ensuring the availability of logs even in system failure may require specific storage solutions.

# Log Retention

i.e. how long should you keep your logs.

- **Hot Storage:** Logs from the past **3-6 months** that are most accessible. Query speed should be near real-time, depending on the complexity of the query.
- **Warm Storage:** Logs from **six months to 2 years**, acting as a data lake, easily accessible but not as immediate as Hot storage.
- **Cold Storage:** Archived or compressed logs from **2-5 years**. These logs are not easily accessible and are usually used for retroactive analysis or scoping purposes.

# Log Deletion

Log deletion helps to:

- Maintain a manageable size of logs for analysis.
- Comply with privacy regulations, such as GDPR, which require unnecessary data to be deleted.
- Keep storage costs in balance.

# Best Practices:

- Determine the storage, retention, and deletion policy based on both business needs and legal requirements.
- Regularly review and update the guidelines per changing conditions and regulations.
- Automate the storage, retention, and deletion processes to ensure consistency and avoid human errors.
- Encrypt sensitive logs to protect data.
- Regular backups should be made, especially before deletion.

# Log Analysis Process
|                                                                                                                                                                                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![Image Representation of a Data Source](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/bb3dac37904b48660e1ad524832a6383.svg)      | Data Sources<br><br>Data Sources are the systems or applications configured to log system events or user activities. These are the origin of logs.                                                                                                                                                                                                                                                                                                                                                                                                               |
| ![Image Representation of Log Parsing](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/b92ca5757d85ab49ee32cb0217b84b3a.svg)        | Parsing<br><br>Parsing is breaking down the log data into more manageable and understandable components. Since logs come in various formats depending on the source, it's essential to parse these logs to extract valuable information.                                                                                                                                                                                                                                                                                                                         |
| ![Image Representation of Log Normalisation](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/6d50d78e6c09b878bc4bf6aebf73b7a9.svg)  | Normalisation<br><br>Normalisation is standardising parsed data. It involves bringing the various log data into a standard format, making comparing and analysing data from different sources easier. It is imperative in environments with multiple systems and applications, where each might generate logs in another format.                                                                                                                                                                                                                                 |
| ![Image Representation of Log Sorting](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/26eb3e15a848f9249e2951ef44bbcedb.svg)        | Sorting<br><br>Sorting is a vital aspect of log analysis, as it allows for efficient data retrieval and identification of patterns. Logs can be sorted by time, source, event type, severity, and any other parameter present in the data. Proper sorting is critical in identifying trends and anomalies that signal operational issues or security incidents.                                                                                                                                                                                                  |
| ![Image Representation of Log Classification](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/907a1ff6f5a4c8fd0c0250282a23568f.svg) | Classification<br><br>Classification involves assigning categories to the logs based on their characteristics. By classifying log files, you can quickly filter and focus on those logs that matter most to your analysis. For instance, classification can be based on the severity level, event type, or source. Automated classification using machine learning can significantly enhance this process, helping to identify potential issues or threats that could be overlooked.                                                                             |
| ![Image Representation of Log Enrichment](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/2f5c66bfdad44c7f2fd5dd211a7541fc.svg)     | Enrichment<br><br>Log enrichment adds context to logs to make them more meaningful and easier to analyse. It could involve adding information like geographical data, user details, threat intelligence, or even data from other sources that can provide a complete picture of the event.<br><br>Enrichment makes logs more valuable, enabling analysts to make better decisions and more accurately respond to incidents. Like classification, log enrichment can be automated using machine learning, reducing the time and effort required for log analysis. |
| ![Image Representation of Log Correlation](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/0a909d3f679c458f65ef08d5770389fc.svg)    | Correlation<br><br>Correlation involves linking related records and identifying connections between log entries. This process helps detect patterns and trends, making understanding complex relationships between various log events easier. Correlation is critical in determining security threats or system performance issues that might remain unnoticed.                                                                                                                                                                                                  |
| ![Image Representation of Log Visualisation](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/f19a2dcb7e6af13d489590b4b13c6c3f.svg)  | Visualisation<br><br>Visualisation represents log data in graphical formats like charts, graphs, or heat maps. Visually presenting data makes recognising patterns, trends, and anomalies easier. Visualisation tools provide an intuitive way to interpret large volumes of log data, making complex information more accessible and understandable.                                                                                                                                                                                                            |
| ![Image Representation of Log Reporting](https://tryhackme-images.s3.amazonaws.com/user-uploads/63da722f2d207d0049da10b1/room-content/315f0456ea5f0130ac6a05a7d17153f3.svg)      | Reporting<br><br>Reporting summarises log data into structured formats to provide insights, support decision-making, or meet compliance requirements. Effective reporting includes creating clear and concise log data summaries catering to stakeholders' needs, such as management, security teams, or auditors. Regular reports can be vital in monitoring system health, security posture, and operational efficiency.                                                                                                                                       |

# Tools:
- [[rsyslog]] - For collecting logs
- [[logrotate]] - automates log file rotation, compression, and management
- [[Splunk]] - Log Analysis
- [[Elastic Search]] - Log Analysis
- 
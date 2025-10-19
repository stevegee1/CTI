# Real-time Analysis: ACCOUNT COMPROMISE
- **Rule Name:** Identity - Admin Account Logged In
- **Rule Description:**
This alert is triggered when a login event is detected using the administrator
account. These accounts have elevated access and are often targetted by
attackers to gain control over systems or the domain. Authorized IPs have been
whitelisted.

## Incident Response
1. Analyse the query result generated in Microsoft Defender. This showed that
   the Domain Controller, `MTS-DC.MTS-local`, was logged on with administrative
   access.
2. Important details from the query:
Category: Initial Access
MITRE ATT&CK Techniques: `T1078: Valid Accounts`
Service Source: Microsoft Sentinel
Generated on: x.x.x UTC
First Activity: x.x.x UTC
Last Activity: x.x.x UTC
EventID: 4624
IP Address of the threat actor: 45.227.254.154


**Severity**: Critical
This is critical because with admin privilege, a threat actor can alter the
Active Directory. This implies that the actor has full control over accounts,
permissions, policies of that domain.

3. Any public information about the threat actor IP?
using [AbuseIPDB](https://www.abuseipdb.com/):
`This IP was reported 765 times. Confidence of Abuse is 100%`

We have sysmon logs forwarded to Microsoft Sentinel.
## Write and run custom queries for further analysis
```sql
SecurityEvent
|where EventID == "4624"
|where IpAddress = "45.227.254.154"
|sort by TimeGenerated desc
```
Output
```
Threat actor device: B_114
```
We can further improve our query with the threat actor device filter. In this
case, we might capture event where threat actor is using different IP addresses
(VPN).

```sql
SecurityEvent
|where EventID == "4624"
|where IpAddress in "45.227.254.154"
|where WorkstationName == "B_114"
|sort by TimeGenerated desc
```
The above query uncovered a new IpAddress we didn't account for by adding the
WorkstationName filter. This shows that the actor is using different IPs

**Note:** Timeline is necessary for any findings so as to follow chain of events

```sql
SecurityEvent
|where EventID == "4624"
|where IpAddress in "45.227.254.154"
|where WorkstationName == "B_114"
|sort by TimeGenerated desc
|summarize by Computer
```
We wanna determine the scope of this threat actor in our environment by checking
for any logs other than our Domain Controller. Luckily, no lateral movement yet.

```sql
WindowsEvent
|where TimeGenerated between (datetime(x.x.x) .. datetime(x.x.x))
|where EventData.User endswith "administrator"
|where eventID == "1"
|project TimeGenerated, EventData.Image, EventData.CommandLine, EventData.User, EventData.ParentImage, EventData.FilePath
|sort by TimeGenerated in desc
```
#### Investigation
- At x.x.x UTC, the administrator launched powershell.exe
- Firewall rule was modified to enable a group called "Network Discovery"
- Installed Anydesk
- Create guest account and add to admin local group

Check other eventID detected for sysmon
- 29: FileExecutableDetected
- 26: FileDeleteDetected (File Delete logged)
- 22: DNSEvent (DNS query)
- 17: PipeEvent (Pipe Created)
- 13: RegistryEvent (Value Set)
- 12: RegistryEvent (Object create and delete)
- 11: FileCreate
- 7: Image loaded
- 3: Network Connection
- 1: Process Creation
[Learn_more](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

At x.x.x Utc admin used powershell to create a file under "C:\windows':
priviledge test

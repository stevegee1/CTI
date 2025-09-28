# Scenario
The Account Executive called the SOC earlier, sounding very frustrated and angry. He said he can’t access any files on his computer and keeps getting a pop-up saying his files have been encrypted. You disconnected the computer from the network, extracted a memory dump from his machine, and started analyzing it with Volatility. Continue your investigation to uncover how the ransomware works and how to stop it!

# Walkthrough

### Preferred Lab Env and Tools
- Kali Linux env
- Volatility tool

### Configure Your Env
#### Prerequisites:
- Git installed
- Python3 installed (we’ll be using volatility3)
- Python3-pip installed
- Memory dump file downloaded

#### Install volatility3
I’m using the latest dev version of Volatility 3.

But you can also use the stable, easy-to-install version via pip:

```bash
pip install volatility3
```

### Understand the Problem
When you run into a case where files are inaccessible, and the user keeps getting weird encryption popups, ransomware should be your first suspect.

Ransomware is malware that steals data and holds it hostage. This breaks the **availability** of sensitive data for the rightful user or owner. Criminals typically demand payment before releasing the data—but sometimes, even paying doesn’t guarantee recovery.

Sensitive data could be PII (Personally Identifiable Information), financial data, intellectual property, etc.

The rise of cryptocurrency has made ransomware even more appealing, since payments are easy and harder to trace.

There are two main types of ransomware:
- **Encrypting (crypto) ransomware**: Encrypts files and demands payment for the decryption key.
- **Screen-locking ransomware**: Locks the entire device by blocking OS access
  and demands payment on a ransom screen.

#### References:
* [Fortinet Reference](https://www.fortinet.com/resources/cyberglossary/ransomware)
* [IBM Reference](https://www.ibm.com/think/topics/ransomware)

### Solve the Problem

#### Why Memory Analysis?
If we rely only on listing processes on the isolated machine with something like `tasklist`, we’ll only see standard processes. Malware can easily evade detection that way.

Memory analysis gives us a deeper look. From a memory dump, we can analyze:
- Running processes at the time of the dump
- Dead processes (if their EPROCESS blocks weren’t overwritten yet)
- Hidden or unlinked processes (e.g., rootkits that unhook themselves from normal process lists but still exist in memory)

---

## First Goal: Find a Suspicious Process

#### Memory Environment
This dump is from a Windows environment, so we’ll use Windows-specific plugins:

```bash
python3 vol.py -f /home/kali/Desktop/btlos/infected.vmem windows.psscan >> mem_analysis_psscan.txt
```

#### Common Pitfall: Don’t judge a book by its cover
At first glance, you might be tempted to flag obvious names like `hack.exe` or `malware.exe`. But attackers are smarter than that.

For example: `dii.exe` vs `d1i.exe`—easy to miss! Malware authors often camouflage names to look legit.

Still, let’s flag these two as suspicious for now:

```
3968    2732    @WanaDecryptor  0x1fcc6800      1       59      1       False   2021-01-31 18:02:48.000000 UTC  N/A     Disabled
2732    1456    or4qtckT.exe    0x1fcd4350      8       79      1       False   2021-01-31 18:02:16.000000 UTC  N/A     Disabled
```

---

### Go In-Depth

#### Check exact process arguments
```bash
python3 vol.py -f /home/kali/Desktop/btlos/infected.vmem windows.cmdline >> mem_analysis_cmdline.txt
```

Looking at arguments for the suspicious processes:

```
3968    @WanaDecryptor  @WanaDecryptor@.exe
2732    or4qtckT.exe    "C:\Users\hacker\Desktop\or4qtckT.exe"
```

Both definitely look malicious.

#### pstree – see who spawned who
```bash
python3 vol.py -f /home/kali/Desktop/btlos/infected.vmem windows.pstree
```

**Output:**
```bash
* 2732  1456    or4qtckT.exe    0x83ed4350      8       79      1       False   2021-01-31 18:02:16.000000 UTC  N/A     \Device\HarddiskVolume1\Users\hacker\Desktop\or4qtckT.exe "C:\Users\hacker\Desktop\or4qtckT.exe"  C:\Users\hacker\Desktop\or4qtckT.exe
** 3968 2732    @WanaDecryptor  0x83ec6800      1       59      1       False   2021-01-31 18:02:48.000000 UTC  N/A     \D
```

#### Indicators of process status:
- **No star**: Normal, active process
- **One star (*)**: Suspicious, possibly hidden/unlinked
- **Double star (**)**: Strong red flag—partially corrupted or inconsistent

Here, `@WanaDecryptor` (PID 3968) was spawned by `or4qtckT.exe` (PID 2732).

---

### Digging into the Malicious PID 3968

Let’s dump its executable:

```bash
python3 vol.py -f /home/kali/Desktop/btlos/infected.vmem windows.psscan --pid 2732 --dump
```

This creates a dump file—a forensic snapshot of the suspicious process. With it, you can:
- Analyze exactly what was running
- Recover decrypted or unpacked payloads
- Compute hashes for VirusTotal submission

Example hash:

```bash
sha256sum 2732.or4qtckT.exe.0x400000.dmp
7ff3e3a98398c0ebd4b3cd4951396ec40229112a9dd387a868f2f443ea9b7874  2732.or4qtckT.exe.0x400000.dmp
```

Upload the hash to [VirusTotal](https://www.virustotal.com/gui/home/upload).

The [detection results](https://www.virustotal.com/gui/file/7ff3e3a98398c0ebd4b3cd4951396ec40229112a9dd387a868f2f443ea9b7874/detection) confirm this is **WannaCry ransomware**.

---

### What is WannaCry Ransomware?
According to [Kaspersky](https://www.kaspersky.com/resource-center/threats/ransomware-wannacry), WannaCry is crypto ransomware used to extort money by encrypting files and demanding ransom.

**Indicators of Compromise (IoCs)** for WannaCry are well-documented. A quick search turns up [Google’s Threat Intelligence profile](https://cloud.google.com/blog/topics/threat-intelligence/wannacry-malware-profile) for deeper analysis.

---

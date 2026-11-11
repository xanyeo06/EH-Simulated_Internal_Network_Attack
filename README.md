# EH — Simulated Internal Network Attack

An internal penetration test against a fictional Singapore software firm, beginning at a spear-phished HR workstation and ending with Domain Admin rights on the domain controller and exfiltrated file server data. The exercise ran on five virtual machines in an isolated subnet and recorded each step, the weakness it relied on and the control that would have stopped it.

## Contents

| File | Description |
| --- | --- |
| `EH-Report.docx` | Full assignment report with methodology, evidence and findings |

## Scenario

Synergy Dynamics is a Singapore-based software firm delivering enterprise systems to public sector agencies and financial institutions across ASEAN. The pretext is a spear-phishing campaign by a fictional group posing as GovSuite, a vendor contracted under a government digitalisation programme; the lure is a fake SGTS compliance patch sent to an HR executive at the firm's Yishun office. The assumed starting condition is no credentials and no network position, only a user willing to open an attachment; everything after that is gained from the first workstation. The engagement was a lab exercise on authorised virtual machines in an isolated network.

## Lab environment

| Host | OS | Address | Role |
| --- | --- | --- | --- |
| Attacker | Parrot OS | 192.168.8.100 | Attack platform: payload generation, Metasploit handler, SMB client, hash cracking |
| ALICE | Windows 10 (build 14393) | 192.168.8.200 | HR executive workstation; initial foothold |
| BOB | Windows 10 | 192.168.8.250 | CEO workstation; second target |
| File Server | Windows Server 2019 | 192.168.8.20 | Finance share holding payroll and departmental documents |
| Domain Controller | Windows Server 2022 | 192.168.8.30 | Active Directory domain `synergy.local` |

All five hosts share one VMware VMnet8 subnet, 192.168.8.0/24, with no internal segmentation.

## Attack path

### Initial access

A Meterpreter reverse TCP payload was generated with msfvenom, encoded to hinder signature detection and compressed into a zip archive to improve its chances of passing mail filtering. It was delivered as the promised compliance patch and executed by the HR user, returning a session as `SYNERGY\alicetan`. The step worked because executable attachments from external senders were accepted and the recipient had no practical way to verify the sender.

### Discovery

Host and network detail came from the session itself — account, build, interfaces, connections — then a Metasploit ARP scan enumerated nine live hosts on the /24. An nmap scan of ports 139 and 445 characterised the file server and other live hosts, and an SMB version scan run from the compromised host found a second workstation at .250 running SMB 3.1.1 in the same domain. The flat subnet meant one session exposed every peer.

### Credential access

The session was migrated into `svchost.exe` running as SYSTEM and the hash dump module recovered local NTLM hashes for Administrator, Jason, Martin and Shiela, all of which John the Ripper returned from `rockyou.txt`. On the second host a Mimikatz-style dump produced cleartext passwords, hashes and Kerberos tickets, and a plaintext password was also found in a text file on the desktop. Weak and reused passwords made offline recovery trivial.

### Lateral movement

The cracked local Administrator password opened the Finance share over SMB and, through psexec, produced a SYSTEM session on the CEO's workstation; the same credentials were reused against the file server, where payroll data was located and downloaded. Because the password was identical on both hosts, one cracked hash covered them both.

### Privilege escalation

winPEAS on the first workstation reported writable profile and Public folders, PATH injection points in the Winlogon Shell and SafeBoot AlternateShell values, and saved browser credentials. The exploit suggester identified MS16-075, which raised the session to NT AUTHORITY\SYSTEM. The host was unpatched, and the writable paths offered a second route.

### Domain compromise

On the CEO's workstation the credential dump exposed that user's NTLM hash, and name resolution returned the domain controller. The hash alone authenticated to the controller through impacket's psexec — pass-the-hash — after which domain accounts were enumerated and a new account added to Domain Admins. Only an account with domain administrative rights can authenticate to a controller this way, so a workstation compromise was equivalent to a domain compromise.

### Persistence

A registry Run key and an onlogon scheduled task, both named to resemble a national digital service, launched a second payload at logon and at boot with SYSTEM rights. Both were removed and the Run key re-read to confirm removal. The naming is the part a defender would have to notice.

## Findings

The report assigns no severity ratings; weaknesses are listed in the order the chain reached them.

| Weakness | Impact |
| --- | --- |
| Executable attachment accepted from an external sender; no sandboxing or attachment policy | Code execution on an HR workstation as a standard domain user |
| Weak local passwords, reused: `Pa$$w0rd` for Administrator and Shiela, `qwerty` and `apple` for two others | All local hashes recovered offline; the same Administrator password also opened the file server |
| Plaintext password stored in a file on the CEO's desktop | Credential theft with no cracking required |
| Credentials left in LSASS memory; NTLM hash accepted for network authentication | Pass-the-hash from a workstation to the domain controller |
| Domain Admin rights held by an ordinary user account | A single workstation compromise became a full domain compromise |
| Finance share accessible to a standard HR account | Payroll and other departmental data reachable from the first foothold |
| Unpatched Windows 10 build 14393, vulnerable to MS16-075; writable profile folders and PATH injection points | Local escalation to SYSTEM by two independent routes |
| Single flat subnet with SMB exposed on every host | Nine hosts reachable from one session; remote execution usable across the subnet |
| No endpoint detection; Meterpreter, hash dumping and psexec ran unalerted | The scenario narrative places discovery four days after initial access, by outbound traffic monitoring alone |

## Countermeasures

**Identity**

- Remove Domain Admin rights from user accounts and separate administrative from day-to-day identities, so a workstation session cannot reach the directory.
- Enforce password complexity and screen new passwords against breached-password lists; the recovered passwords were all in a public wordlist.
- Give each host a unique local Administrator password through a managed solution, and keep secrets in a vault rather than in files on endpoints.
- Require MFA for privileged and remote logons, and restrict credential exposure in memory to blunt hash-based authentication.

**Endpoint and email**

- Deploy EDR with behavioural detection and application control, which is what the Meterpreter sessions, hash dumps and psexec service creation would have triggered.
- Maintain a patch cycle; build 14393 predates the MS16-075 fix.
- Correct folder ACLs on profile and Public directories, remove writable PATH entries and fix unquoted service paths, closing the escalation route winPEAS found.
- Block executable and script attachments at the gateway, sandbox unknown files, and run phishing simulations with training on urgency and sender-domain mismatch — the indicators present in this lure.

**Network**

- Segment workstations, servers and directory services into separate zones, and restrict SMB to the flows that require it rather than exposing 139 and 445 subnet-wide.
- Filter egress so a reverse shell cannot reach an arbitrary external address, which is also the control the scenario credits with eventual discovery.

**Monitoring**

- Alert on Run key and scheduled task creation, service installation and LSASS access, covering both the persistence mechanism and the credential theft.
- Centralise directory logs in a SIEM, audit additions to Domain Admins, and baseline SMB read volume and outbound connections to surface bulk file access.

## Key takeaways

- An email attachment and a short chain of credential abuse reached Domain Admin, each stage reusing trust granted by the one before; password reuse and standing Domain Admin rights on a user account did more for the attacker than any vulnerability.
- The lab had no endpoint telemetry, so every stage ran silently; detection depended on network egress, which is the last place to look.
- Tiered administration and network segmentation break the chain at lateral movement, before credential material and directory rights become reachable.

## Notes

- Academic exercise conducted on authorised virtual machines in an isolated network; no real systems, organisations or individuals were targeted.
- The organisation, its staff and the threat group are fictional; all credentials in the report are lab values.
- No command lines, payload code or step-by-step procedure is reproduced here. The report documents the method for assessment purposes.

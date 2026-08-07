# Windows Active Directory Security Baseline (GPO)

> A curated collection of commonly recommended Group Policy Objects (GPOs) that help improve the security of an Active Directory environment. Each section contains a short explanation, the policy path, and the recommended configuration.

> **Note**
>
> These recommendations are intended as a baseline. Always validate them in a test environment before deploying them to production.

---

# Contents

1. Password Policy
2. Automatic Screen Lock
3. Block Control Panel
4. Disable LLMNR
5. Disable AutoRun
6. Disable SMBv1
7. PowerShell Logging
8. Advanced Audit Policy
9. Microsoft Defender Hardening

---

# 1. Password Policy

**Why?**

Weak passwords remain one of the most common attack vectors. A strong password policy helps reduce the risk of password spraying, brute-force attacks, and credential reuse.

**GPO Path**

`Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`

| Setting | Recommended value |
|---------|-------------------|
| Enforce password history | 24 passwords |
| Maximum password age | 60 days |
| Minimum password length | 12 characters |
| Password must meet complexity requirements | Enabled |
| Store passwords using reversible encryption | Disabled |

---

# 2. Automatic Screen Lock

**Why?**

Prevents unauthorized access to unattended workstations.

**GPO Path**

`User Configuration → Policies → Administrative Templates → Control Panel → Personalization`

- Screen saver timeout → **300 seconds**
- Password protect the screen saver → **Enabled**

---

# 3. Block Control Panel

**Why?**

Standard users should not be able to modify system configuration.

**GPO Path**

`User Configuration → Policies → Administrative Templates → Control Panel`

- Prohibit access to Control Panel and PC settings → **Enabled**

---

# 4. Disable LLMNR

**Why?**

LLMNR can be abused by tools such as Responder to capture NTLM authentication attempts.

**GPO Path**

`Computer Configuration → Policies → Administrative Templates → Network → DNS Client`

- Turn off Multicast Name Resolution → **Enabled**

---

# 5. Disable AutoRun

**Why?**

Prevents automatic execution of software from removable media.

**GPO Path**

`Computer Configuration → Policies → Administrative Templates → Windows Components → AutoPlay Policies`

- Turn off AutoPlay → **Enabled**
- Turn off AutoPlay on → **All drives**
- Default AutoRun behavior → **Don't execute any autorun commands**

---

# 6. Disable SMBv1

**Why?**

SMBv1 is deprecated and vulnerable. Only SMBv2/SMBv3 should be used.

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol -NoRestart
```

or

```powershell
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
```

Verify:

```powershell
Get-SmbServerConfiguration | Select EnableSMB1Protocol
```

Expected result:

```
False
```

---

# 7. Enable PowerShell Logging

**Why?**

Improves visibility into PowerShell activity for incident response and SIEM.

**GPO Path**

`Computer Configuration → Policies → Administrative Templates → Windows Components → Windows PowerShell`

- Script Block Logging → **Enabled**
- Module Logging → **Enabled** (`*`)
- PowerShell Transcription → **Enabled**
- Include invocation headers → **Enabled**

---

# 8. Advanced Audit Policy Configuration

By default, the Windows Event Logs collected and displayed in Event Viewer are not sufficient for effective security monitoring and incident investigation. Expanding the audit policy provides significantly more visibility into authentication events, account changes, privilege usage and system modifications. These logs are also essential for SIEM platforms such as Wazuh, Microsoft Sentinel or Splunk, allowing them to detect suspicious activity and alert administrators about potential attacks.

## GPO Path


`Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Advanced Audit Policy Configuration -> Audit Policies`


## Recommended Configuration

| Category | Policy | Recommendation |
|-----------|--------|----------------|
| Account Logon | Credential Validation | Success + Failure |
| Logon/Logoff | Logon | Success + Failure |
| Logon/Logoff | Logoff | Success |
| Account Management | User Account Management | Success |
| Account Management | Computer Account Management | Success |
| Account Management | Security Group Management | Success |
| Policy Change | Audit Policy Change | Success + Failure |
| Privilege Use | Sensitive Privilege Use | Success + Failure |
| System | Security State Change | Success |
| System | Security System Extension | Success |
| System | System Integrity | Success + Failure |

### Domain Controllers Only

| Category | Policy | Recommendation |
|-----------|--------|----------------|
| DS Access | Directory Service Changes | Success |
| DS Access | Directory Service Access | Success + Failure *(Optional - generates a high volume of logs)* |

---

## What does each audit policy provide?

### Credential Validation

Logs successful and failed authentication attempts.

Useful for detecting:

- Brute-force attacks
- Password spraying
- Invalid credential usage

---

### Logon

Records every successful and failed user logon.

Provides information such as:

- Who logged in
- When they logged in
- Which logon type was used (interactive, RDP, network, etc.)

---

### Logoff

Logs when users sign out of their sessions.

These events are useful for reconstructing user activity timelines during incident investigations.

---

### User Account Management

Monitors all changes related to user accounts.

Examples include:

- Password changes
- User creation
- User deletion
- Account enable/disable

---

### Computer Account Management

Tracks modifications to computer accounts within Active Directory.

Examples include:

- Computer creation
- Computer deletion
- Changes to computer account properties

---

### Security Group Management

Monitors changes made to security groups.

Examples include:

- Adding users to **Domain Admins**
- Removing users from privileged groups
- Creating or deleting security groups

---

### Audit Policy Change

Tracks modifications to Windows auditing and security policies.

Examples include:

- GPO changes
- Audit policy modifications
- Logging being disabled

Attackers frequently attempt to disable or modify auditing in order to reduce their visibility. Monitoring these events helps detect such activity.

---

### Sensitive Privilege Use

Logs the use of privileged Windows permissions by administrators, services and applications.

This helps detect:

- Privilege escalation
- Administrative abuse
- Malware attempting to abuse high-privilege rights

---

### Security State Change

Monitors important security-related changes within Windows.

Examples include:

- System startup
- Security subsystem startup
- Changes to the Windows security state

---

### Security System Extension

Tracks the installation or loading of security-related Windows components.

Examples include:

- LSA extensions
- Security providers
- Authentication packages

---

### System Integrity

Monitors integrity-related issues within Windows.

Examples include:

- Code Integrity violations
- Driver signature verification failures
- Security component integrity issues

These events are particularly useful during malware investigations.

---

### Directory Service Changes *(Domain Controllers)*

Tracks modifications made to Active Directory objects.

Examples include:

- Organizational Unit (OU) changes
- User attribute modifications
- Group attribute modifications

---

### Directory Service Access *(Optional)*

Logs access attempts to Active Directory objects.

Examples include:

- Reading user objects
- Reading security groups
- Accessing other AD objects

> **Note:** This policy generates a very large number of events and is usually enabled only temporarily during incident response or forensic investigations.

---

# 9. Microsoft Defender Hardening

Microsoft Defender Antivirus can be centrally managed using Group Policy. Enabling additional security features significantly improves endpoint protection by increasing visibility into malicious activity, inspecting scripts before execution and detecting threats hidden inside archives or packed executables.

## GPO Path


`Computer Configuration -> Policies -> Administrative Templates -> Windows Component -> Microsoft Defender Antivirus`


## Recommended Configuration

| Category | Policy | Recommendation |
|-----------|--------|----------------|
| Real-Time Protection | Turn on real-time protection | Enabled |
| Real-Time Protection | Turn on behavior monitoring | Enabled |
| Real-Time Protection | Scan all downloaded files and attachments | Enabled |
| Real-Time Protection | Turn on script scanning | Enabled |
| Real-Time Protection | Turn on process scanning whenever real-time protection is enabled | Enabled |
| Scan | Scan archive files | Enabled |
| Scan | Scan packed executables | Enabled |
| Scan | Check for the latest virus and spyware definitions before running a scheduled scan | Optional |
| Scan | Scan removable drives | Optional |

---

## What does each setting do?

### Turn on real-time protection

Provides continuous protection by scanning files whenever they are downloaded, opened or executed.

---

### Turn on behavior monitoring

Analyzes the behavior of running processes instead of relying solely on malware signatures.

This allows Defender to detect suspicious activity and previously unknown threats.

---

### Scan all downloaded files and attachments

Automatically scans every downloaded file and email attachment before it can be used.

---

### Turn on script scanning

Scans scripts before they are executed.

This includes PowerShell, JavaScript, VBScript and other supported scripting engines.

---

### Turn on process scanning whenever real-time protection is enabled

In addition to scanning files, Defender continuously monitors running processes in memory.

This helps detect attacks that only become malicious after execution, including:

- Fileless malware
- Memory injection
- Process hollowing
- In-memory payloads

---

### Scan archive files

Scans the contents of compressed files before they are extracted, helping detect malware hidden inside archives.

---

### Scan packed executables

Detects executables that have been packed or obfuscated.

Defender attempts to unpack these files and analyze their actual contents, making it easier to detect malware hidden inside packed executables.

---

### Check for the latest virus and spyware definitions before running a scheduled scan *(Optional)*

Before starting a scheduled scan, Defender checks whether newer security signatures are available.

If updates are found, they are downloaded before the scan begins, ensuring the latest threats can be detected.

This setting is optional because signature updates are often delivered automatically through Windows Update.

---

### Scan removable drives *(Optional)*

Scans USB flash drives and external storage devices for malware.

This setting does **not** block removable media-it simply scans the files stored on those devices to help prevent malware infections.
# Penetration Testing Report - Target 10.201.64.74

## Executive Summary

This report documents the complete penetration testing process of target system 10.201.64.74, from initial reconnaissance through privilege escalation to root access. The exploitation involved bypassing file upload restrictions, deploying a PHP reverse shell, and escalating privileges using SUID binaries.

## Target Information

- **Target IP**: 10.201.64.74
- **Testing Platform**: TryHackMe
- **Exploitation Type**: Web Application + Privilege Escalation

## Methodology

### 1. Reconnaissance and Enumeration

#### Initial Nmap Scan

The penetration test began with a standard nmap scan to identify open ports and running services.
```bash
nmap 10.201.64.74
```

![Initial Nmap Scan](Pasted%20image%2020250806203408.png)

#### Service Version Detection

Following the initial scan, a more detailed service enumeration was performed to identify specific versions and potential vulnerabilities.
```bash
nmap -sV 10.201.64.74
```

![Service Version Scan](Pasted%20image%2020250806203758.png)

#### Web Application Discovery

Initial web application reconnaissance revealed a basic web interface.

![Web Interface](Pasted%20image%2020250806203652.png)

### 2. Directory Enumeration

Gobuster was utilized for directory discovery while service scans were running in parallel.
```bash
gobuster dir -u http://10.201.64.74/ \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt 
```

![Gobuster Results](Pasted%20image%2020250806212549.png)

Key directories discovered:
- `/panel` - File upload functionality
- `/uploads` - File storage location

### 3. Vulnerability Assessment

#### Service Vulnerability Scan

A comprehensive vulnerability scan was performed to identify outdated services or known CVEs.
```bash
nmap --script vuln 10.201.64.74
```

![Vulnerability Scan](Pasted%20image%2020250806212418.png)

#### Web Application Analysis

Two critical endpoints were identified:

**Panel Directory** - File Upload Interface:
![Panel Interface](Pasted%20image%2020250806212758.png)

**Uploads Directory** - File Access Point:
![Uploads Directory](Pasted%20image%2020250806212845.png)

*Note: Screenshots were captured after successful exploitation*

## Exploitation Process

### 4. File Upload Bypass

Based on TryHackMe hints, the attack vector involved PHP reverse shell upload with file sanitization bypass.

![TryHackMe Hint](Pasted%20image%2020250806213258.png)

#### Reverse Shell Generation

Multiple PHP reverse shells were tested using the online generator at [RevShells.com](https://www.revshells.com/).

**Successful Payload**: The second PHP reverse shell from PentestMonkey successfully bypassed upload restrictions.

#### Initial Access

A netcat listener was established to catch the reverse shell connection, providing initial system access with limited privileges.

### 5. Privilege Escalation

#### SUID Binary Discovery

Following the room's hint about SUID binaries, enumeration revealed potential escalation vectors.

![SUID Hint](Pasted%20image%2020250806213741.png)

![SUID Binaries](Pasted%20image%2020250806213940.png)

#### GTFOBins Research

Research using [GTFOBins](https://gtfobins.github.io/) identified Python as a viable SUID escalation method.

![GTFOBins Python](Pasted%20image%2020250806214132.png)

#### Shell Stabilization

Initial attempts with standard Python SUID exploitation failed. Shell stabilization was achieved using:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

#### Root Access

Within the stabilized shell, the following command successfully escalated to root privileges:

```bash
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![Root Shell Achievement](Pasted%20image%2020250806215055.png)

## Attack Chain Summary

1. **Reconnaissance**: Port scanning and service enumeration
2. **Discovery**: Directory brute-forcing revealed upload/panel functionality
3. **Exploitation**: PHP reverse shell upload bypassing file restrictions
4. **Access**: Initial shell with limited privileges
5. **Escalation**: SUID Python binary exploitation for root access

## Key Findings

### Vulnerabilities Identified

1. **Inadequate File Upload Validation**: The panel directory allows malicious file uploads
2. **Accessible Upload Directory**: Uploaded files are directly accessible via web interface
3. **SUID Binary Misconfiguration**: Python binary with SUID bit enables privilege escalation


## Tools Used

- **Nmap**: Network reconnaissance and service enumeration
- **Gobuster**: Directory and file discovery
- **RevShells.com**: Reverse shell payload generation
- **Netcat**: Reverse shell listener
- **GTFOBins**: SUID binary exploitation research


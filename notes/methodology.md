# Testing Methodology

## 1. Purpose

This document describes the methodology used during an authorized internal penetration test against a deliberately vulnerable Metasploitable 2 virtual machine.

The objective was to practice a structured penetration-testing workflow, including:

- Lab setup
- Port discovery
- Service enumeration
- Manual vulnerability validation
- Evidence collection
- Risk classification
- Cleanup
- Reporting

The assessment was performed only within an isolated lab environment.

---

## 2. Lab Environment

| Component | Details |
|---|---|
| Attacker machine | Kali Linux |
| Target machine | Metasploitable 2 |
| Virtualization platform | Oracle VirtualBox |
| Network type | Host-only Adapter |
| Target IP address | `192.168.56.3` |
| Assessment type | Internal network penetration test |

The host-only network prevented the target from being exposed directly to external systems.

No production, public, or third-party systems were tested.

---

## 3. Rules of Engagement

The following boundaries were applied during the assessment.

### Allowed Activities

- Host connectivity validation
- TCP port scanning
- Service and version detection
- Default-script enumeration
- Manual authentication testing
- Manual validation of identified vulnerabilities
- Sensitive file access validation
- Evidence collection
- Risk and impact analysis

### Excluded Activities

- Denial-of-service testing
- Destructive modification of system files
- Persistence installation
- Malware deployment
- Testing outside the isolated lab network

---

## 4. Host Discovery and Connectivity Validation

Connectivity to the target was confirmed from Kali Linux using:

```bash
ping 192.168.56.3
```

A successful response confirmed that the target was reachable through the host-only network.

---

## 5. Full TCP Port Discovery

A full TCP port scan was performed to identify all exposed TCP services:

```bash
sudo nmap -p- --min-rate 5000 \
-oN nmap-full-port-scan.txt \
192.168.56.3
```

### Command Explanation

- `-p-` scans all 65,535 TCP ports.
- `--min-rate 5000` increases the minimum packet transmission rate.
- `-oN` saves the output in normal text format.

The scan identified multiple exposed services, including FTP, SSH, Telnet, SMTP, HTTP, SMB, NFS, databases, VNC, IRC, Apache Tomcat, and a bind shell.

The full scan output is stored in:

```text
../scans/nmap-full-port-scan.txt
```

---

## 6. Service and Version Enumeration

A focused service scan was performed against the ports discovered during the full TCP scan:

```bash
sudo nmap -sC -sV \
-p21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,36323,48726,54312,57985 \
-oN nmap-service-scan.txt \
192.168.56.3
```

### Command Explanation

- `-sC` runs Nmap's default scripts.
- `-sV` performs service and version detection.
- `-p` limits the scan to discovered ports.
- `-oN` saves the output in normal text format.

The service scan was used to:

- Identify service names and versions
- Detect default configurations
- Identify exposed administrative interfaces
- Prioritize services for manual validation

The service scan output is stored in:

```text
../scans/nmap-service-scan.txt
```

---

## 7. Manual Vulnerability Validation

Potential vulnerabilities were manually tested before being recorded as confirmed findings.

Automated scan results were not treated as confirmed vulnerabilities unless the behavior could be reproduced manually.

### 7.1 Root Bind Shell Validation

TCP port `1524` was tested using Netcat:

```bash
nc -nv 192.168.56.3 1524
```

The following commands were used to confirm the privilege level and target identity:

```bash
whoami
id
hostname
uname -a
pwd
```

The shell returned `uid=0(root)`, confirming unauthenticated root-level access.

---

### 7.2 NFS Export Validation

Available NFS exports were enumerated using:

```bash
showmount -e 192.168.56.3
```

The target returned:

```text
/ *
```

This indicated that the complete root filesystem was exported to all hosts.

A local mount point was created:

```bash
sudo mkdir -p /mnt/msf2-nfs
```

The share was mounted using:

```bash
sudo mount -t nfs -o vers=3,nolock \
192.168.56.3:/ \
/mnt/msf2-nfs
```

Sensitive file access was validated using:

```bash
sudo head /mnt/msf2-nfs/etc/shadow
```

The command returned password-hash entries from the target system.

No password cracking was required to confirm the exposure.

---

### 7.3 Tomcat Manager Validation

The Apache Tomcat service was accessed at:

```text
http://192.168.56.3:8180
```

The manager interface was tested at:

```text
http://192.168.56.3:8180/manager/html
```

Authentication succeeded using the default credentials:

```text
Username: tomcat
Password: tomcat
```

Successful authentication provided access to application-management functionality, including WAR deployment controls.

No malicious WAR file was uploaded or deployed.

---

### 7.4 Anonymous FTP Validation

The FTP service was tested using:

```bash
ftp 192.168.56.3
```

Anonymous authentication was accepted.

Testing confirmed:

```text
Files exposed: No
Anonymous upload: Blocked
Upload response: 553 Could not create file
```

Because no sensitive files were available and anonymous write access was denied, this issue was recorded as a low-risk observation rather than a primary finding.

---

## 8. Tools Used

| Tool | Purpose |
|---|---|
| Nmap | Port discovery, service detection, and default-script enumeration |
| Netcat | Manual validation of the exposed bind shell |
| `showmount` | NFS export enumeration |
| NFS client utilities | Mounting and inspecting the exported filesystem |
| FTP client | Anonymous FTP validation |
| Firefox | Accessing and validating the Tomcat Manager interface |
| cURL | Optional HTTP response validation |
| Linux command-line utilities | File inspection, identity validation, and evidence collection |

---

## 9. Evidence Collection

Evidence was collected for each confirmed finding.

The following evidence files were prepared:

```text
../evidence/finding-01-root-bind-shell.png
../evidence/finding-02-nfs-export.png
../evidence/finding-02-nfs-shadow-exposure.png
../evidence/finding-03-tomcat-login.png
../evidence/finding-03-tomcat-manager.png
```

Evidence collection followed these principles:

- Relevant commands remained visible.
- The target IP and service context remained visible.
- Command output demonstrated the vulnerability clearly.
- Password hashes were redacted before publication.
- No real credentials, tokens, or unrelated personal information were included.

---

## 10. Risk Classification

Findings were classified according to their demonstrated technical impact.

### Critical

A vulnerability was rated Critical when it allowed:

- Immediate full-system compromise
- Unauthenticated root access
- Exposure of highly sensitive authentication data
- Complete compromise without additional user interaction

### High

A vulnerability was rated High when it allowed:

- Administrative access
- Significant control over an exposed service
- Potential remote code execution
- Serious compromise requiring an additional action

### Low

An issue was rated Low when:

- The weakness was confirmed
- The demonstrated impact was limited
- No sensitive data exposure or unauthorized write access was observed

---

## 11. Cleanup

The mounted NFS share was removed after testing:

```bash
sudo umount /mnt/msf2-nfs
```

The temporary mount directory could then be removed using:

```bash
sudo rmdir /mnt/msf2-nfs
```

No persistence mechanisms, malicious applications, additional user accounts, or destructive changes were introduced.

---

## 12. Reporting Process

Each confirmed finding was documented using the following structure:

1. Finding title
2. Severity
3. Affected service
4. Description
5. Validation steps
6. Evidence
7. Impact
8. Remediation

The complete findings are documented in:

```text
../report/penetration-test-report-complete.md
```

---

## 13. Key Lessons Learned

This assessment reinforced several important penetration-testing practices:

- Perform a full port scan before focusing on common services.
- Follow port discovery with service and version enumeration.
- Validate scanner results manually.
- Collect clear evidence for every confirmed finding.
- Explain realistic impact rather than listing technical details only.
- Provide specific and practical remediation guidance.
- Redact sensitive information before publishing evidence.
- Remove temporary mounts and other testing artifacts after completion.

---

## 14. Ethical and Legal Notice

This methodology was used only in a controlled and authorized lab environment.

The techniques described in this document should only be used against systems that you own or have explicit permission to test.

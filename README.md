# ICDFA Lab 2 — Web Application Security Assessment

> **Controlled Penetration Testing Laboratory | File Upload & Command Injection Assessment**

---

## Project Overview

This repository contains the documentation, evidence, integrity hashes, and final assessment report produced during **ICDFA Lab 2**.

The assessment was conducted against intentionally vulnerable web applications deployed inside an isolated Docker-based laboratory environment. The objective was to identify, validate, document, and assess web application security weaknesses while maintaining controlled and non-destructive testing practices.

The assessment focused primarily on:

- File Upload Security
- File Type Validation
- Server-Side File Handling
- Command Injection
- Command Execution Context
- Evidence Preservation
- Security Impact Assessment

All testing was performed within the authorized laboratory environment.

---

## Assessment Scope

| Component | Details |
|---|---|
| Assessment Type | Controlled Web Application Security Assessment |
| Testing Platform | Kali Linux |
| Environment | Docker-based vulnerable application laboratory |
| Primary Application | Damn Vulnerable Web Application (DVWA) |
| Secondary Application | OWASP Mutillidae II |
| Network | Isolated laboratory network |
| Evidence Location | `evidence/` |
| Final Report | `report/` |
| Testing Approach | Black-box validation with controlled local verification |

---

## Repository Structure

```text
ICDFA-Lab2/
│
├── README.md
│
├── report/
│   └── ICDFA_Lab2_File_Upload_Command_Execution_Assessment.pdf
│
├── evidence/
│   │
│   ├── markers/
│   │   ├── lab2-marker.png
│   │   ├── lab2-marker.txt
│   │   └── lab2-mismatch.png
│   │
│   ├── hashes/
│   │   └── marker-sha256.txt
│   │
│   ├── requests/
│   │   └── ...
│   │
│   ├── responses/
│   │   └── ...
│   │
│   ├── screenshots/
│   │   └── ...
│   │
│   └── notes/
│       └── activity-log.txt
│
└── LICENSE
```

---

## Executive Summary

The laboratory assessment successfully demonstrated security weaknesses associated with insufficient validation of user-controlled input.

Testing against DVWA confirmed that the application accepted controlled file uploads and that uploaded content could be stored within the application's upload directory.

A separate command-injection assessment demonstrated that user-controlled input supplied to the command execution functionality could influence operating-system command execution.

The controlled validation produced the following evidence:

```text
ICDFA-LAB2-CMD-MARKER
```

The resulting execution context was:

```text
uid=33(www-data)
gid=33(www-data)
groups=33(www-data)
```

This demonstrates that operating-system command execution occurred within the privileges of the web-server account.

No destructive payloads, persistence mechanisms, real-world targets, or external callback infrastructure were used during this assessment.

---

# Finding 01 — Unrestricted / Insufficiently Validated File Upload

## Description

The file-upload functionality was tested using controlled, non-executable marker files.

Two primary test artifacts were created:

```text
lab2-marker.png
lab2-marker.txt
```

A deliberate extension/content mismatch was also created:

```text
lab2-mismatch.png
```

The mismatch file retained text content while using a `.png` extension.

The application accepted the controlled upload and stored the files in:

```text
/var/www/html/hackable/uploads/
```

The container-side verification confirmed:

```text
lab2-marker.png: PNG image data
lab2-mismatch.png: ASCII text
```

This demonstrates that filename extension and actual file content can differ and should not be treated as equivalent security controls.

## Evidence

The following artifacts support the finding:

- Controlled PNG marker
- Controlled text marker
- Extension/content mismatch file
- SHA-256 integrity hashes
- Container filesystem verification
- Upload confirmation
- Activity log

## Security Impact

Insufficient file validation can allow attackers to upload content that does not conform to the expected file type.

Depending on server configuration and execution permissions, insecure upload handling may potentially lead to:

- Malicious file storage
- Content-type bypasses
- Server-side script execution
- Stored malicious content
- Further application compromise

The actual impact depends on the web-server configuration, executable file types, storage permissions, and application architecture.

---

# Finding 02 — OS Command Injection

## Description

The DVWA command execution functionality was assessed using controlled input.

The baseline request used:

```text
ip=10.5.5.12&Submit=Submit
```

A controlled command-injection test was subsequently performed by appending a harmless marker command:

```text
10.5.5.12 && echo ICDFA-LAB2-CMD-MARKER
```

The application returned the expected ping output followed by:

```text
ICDFA-LAB2-CMD-MARKER
```

This confirms that user-controlled input was incorporated into an operating-system command without adequate separation between data and command syntax.

## Execution Context

The controlled execution produced:

```text
uid=33(www-data)
gid=33(www-data)
groups=33(www-data)
```

The command therefore executed under the web-server service account.

## Security Impact

Command injection can allow an attacker to execute operating-system commands with the privileges available to the vulnerable application.

Potential consequences include:

- Unauthorized file access
- Application data exposure
- Modification of application files
- Credential exposure
- Internal network reconnaissance
- Privilege escalation opportunities
- Further compromise of the application environment

The assessment intentionally stopped at controlled proof-of-execution and did not perform destructive actions or persistence.

---

# Evidence Integrity

Evidence integrity was established using SHA-256 hashing.

The recorded hashes include:

| Artifact | SHA-256 |
|---|---|
| `lab2-marker.png` | `431ced6916a2a21a156e38701afe55bbd7f88969fbbfc56d7fe099d47f265460` |
| `lab2-marker.txt` | `8242651417ea05c42ec91ba992d06c7cfcc99e1d3d06c56d0e38f7d88375af5d` |

The original hash record is available at:

```text
evidence/hashes/marker-sha256.txt
```

To independently verify the artifacts:

```bash
cd evidence
sha256sum markers/*
```

Compare the resulting values against:

```text
hashes/marker-sha256.txt
```

---

# Methodology

The assessment followed a controlled penetration-testing workflow.

## 1. Environment Identification

The vulnerable applications were identified within the Docker laboratory environment.

Example:

```bash
docker inspect -f '{{range $p, $conf := .NetworkSettings.Networks}}{{println $p $conf.IPAddress}}{{end}}' dvwa.pc
```

The DVWA container was identified on the laboratory network at:

```text
10.5.5.12
```

The Mutillidae container was identified at:

```text
10.5.5.11
```

## 2. Application Verification

HTTP connectivity and application responses were validated using `curl`.

Example:

```bash
curl -I http://10.5.5.12/
```

## 3. Controlled File Testing

Non-executable marker files were created and hashed before testing.

Example:

```bash
sha256sum markers/*
```

The files were subsequently uploaded through the application's file-upload functionality.

## 4. Server-Side Verification

The container filesystem was inspected to confirm whether uploaded artifacts were actually stored.

Example:

```bash
docker exec dvwa.pc ls -la /var/www/html/hackable/uploads/
```

File signatures were independently verified:

```bash
docker exec dvwa.pc file \
/var/www/html/hackable/uploads/lab2-marker.png \
/var/www/html/hackable/uploads/lab2-mismatch.png
```

## 5. Command Injection Validation

The command execution endpoint was tested using a harmless marker payload.

The test was designed to demonstrate command interpretation without modifying system files or establishing persistence.

## 6. Evidence Collection

Relevant requests, responses, screenshots, hashes, and activity notes were preserved within the repository.

---

# Safety and Testing Boundaries

This assessment was intentionally constrained to an authorized laboratory environment.

The following restrictions were observed:

- Testing was performed against intentionally vulnerable applications.
- Only controlled local targets were used.
- Non-executable marker files were used for upload testing.
- Command injection validation used harmless marker output.
- No destructive commands were executed.
- No persistence was established.
- No external systems were targeted.
- No external callback infrastructure was used.
- Testing was stopped after sufficient proof of vulnerability was obtained.

The purpose of the exercise was vulnerability identification, validation, evidence preservation, and professional security reporting.

---

# Remediation Recommendations

## File Upload

Implement defense-in-depth controls for uploaded files.

Recommended controls include:

1. Validate file content rather than relying solely on extensions.
2. Use an allowlist of permitted file types.
3. Validate MIME type and file signatures.
4. Generate server-side filenames.
5. Store uploads outside the web root where possible.
6. Disable script execution within upload directories.
7. Apply restrictive filesystem permissions.
8. Enforce reasonable file-size limits.
9. Reject double-extension and ambiguous filenames.
10. Consider malware scanning for production environments.

## Command Injection

The command execution functionality should avoid constructing operating-system commands from raw user input.

Recommended controls include:

1. Avoid shell execution where possible.
2. Use safe language/library APIs instead of shell commands.
3. Apply strict IP-address validation.
4. Use allowlists for accepted input formats.
5. Separate command arguments from command interpreters.
6. Disable unnecessary shell metacharacters.
7. Apply least-privilege service accounts.
8. Restrict network and filesystem permissions.
9. Log suspicious command-injection attempts.
10. Add automated security tests for command-injection payloads.

---

# Risk Assessment

| Finding | Severity | Primary Risk |
|---|---|---|
| File Upload Validation | High | Potential malicious file placement or execution depending on server configuration |
| Command Injection | Critical | Potential arbitrary OS command execution |

Severity should ultimately be mapped to the organization's formal risk-rating methodology and environmental controls.

---

# Evidence Register

| Evidence | Purpose |
|---|---|
| `lab2-marker.png` | Valid PNG upload marker |
| `lab2-marker.txt` | Controlled text marker |
| `lab2-mismatch.png` | Extension/content mismatch test |
| `marker-sha256.txt` | SHA-256 integrity record |
| Request captures | HTTP testing evidence |
| Response captures | Server response evidence |
| Screenshots | Visual confirmation |
| `activity-log.txt` | Testing timeline and methodology |
| Final PDF report | Formal assessment documentation |

---

# Conclusion

The ICDFA Lab 2 assessment successfully validated two significant classes of web application security weaknesses within the controlled laboratory environment.

The file-upload assessment demonstrated that uploaded files could be accepted and stored and that filename extensions could differ from the underlying file content.

The command-injection assessment provided direct proof that controlled user input could influence operating-system command execution. The resulting execution occurred under the `www-data` service account.

These findings demonstrate the importance of strict input validation, secure file-upload architecture, safe command execution practices, least privilege, and defense-in-depth controls.

The assessment was intentionally limited to controlled proof-of-concept activity and did not proceed into destructive exploitation, persistence, or unauthorized access.

---

# Reproduction Environment

The assessment was performed using:

```text
Operating System: Kali Linux
Virtualization/Container Platform: Docker
Primary Application: DVWA v1.9
Secondary Application: OWASP Mutillidae II
Testing Tools: curl, Docker CLI, Linux utilities
```

---

# Disclaimer

This repository is provided for authorized educational and laboratory security testing purposes.

The techniques documented here should only be performed against systems for which explicit authorization has been obtained.

The vulnerable applications used in this assessment are intentionally insecure and should not be exposed to untrusted networks.

---

# Author

**Perry NDZIE KOKO**

ICDFA Lab 2  
Web Application Security Assessment

---

## Report

The complete technical assessment and supporting evidence are available in:

```text
report/ICDFA_Lab2_File_Upload_Command_Execution_Assessment.pdf
```

## Evidence

Supporting artifacts are available under:

```text
evidence/
```

All evidence should be interpreted within the context of the controlled laboratory environment described in this report.
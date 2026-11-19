# LabOps Infrastructure & Application Security Assessment Report

**Date:** November 19–20, 2026  
**Target Environment:** LabOps Local Docker & WSL2 Audit Sandbox (`DESKTOP-3V9U4QU`)  
**Assessment Scope:** Phishing Resilience (`lab02`), SSH Authentication (`lab03`), Cryptographic Key Integrity (`lab04-05`), Secrets Management (`lab06`), and Software Supply Chain Verification (`lab07`)

---

## 1. Executive Summary

This report documents the findings of a controlled, authorized security assessment performed on the **LabOps** development and deployment environment. All testing was conducted strictly within an isolated local Ubuntu WSL2 and Docker container environment (`labops-project-ssh-server` bound to `127.0.0.1:2222` and a local HTTP sandbox on `localhost:8080`) using pre-authorized laboratory assets (`~/CS-Vaje-main` and `~/labops-project`).

### Summary of Key Findings

| ID | Lab Module | Vulnerability / Weakness | Severity | CVSS v3.1 | Primary Impact |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **F-01** | `lab02` | Unencrypted Phishing Login Clone (`login.html`) | **High** | 7.4 | Credential theft via lookalike login portals lacking TLS and domain validation |
| **F-02** | `lab03` | Weak SSH Password Policy & Missing Brute-Force Rate Limiting | **Critical** | 9.1 | Complete unauthorized shell access (`weakuser` compromised in **5.106 seconds**) |
| **F-03** | `lab04-05` | Unverified GPG Public Key Substitution (Identity Spoofing) | **High** | 7.5 | Silent interception and decryption of confidential data by a rogue keypair |
| **F-04** | `lab06` | Plaintext Database Credentials in Environment File (`secrets.env`) | **Critical** | 8.8 | Full database compromise (`DB_PASSWORD=VeryStrongPassword`) via disk or repository exposure |
| **F-05** | `lab07` | Decoupled Checksum & GPG Signature Verification on `install.sh` | **High** | 8.1 | Execution of tampered installation code if administrators verify only `.sha256.sig` without running `sha256sum -c` |

---

## 2. Scope, Target Assets, and Test Environment

### 2.1 Asset Inventory

| Asset Name | Type | Location / Endpoint | Role & Criticality |
| :--- | :--- | :--- | :--- |
| **LabOps SSH Container** | Docker Service (`labops-project-ssh-server`) | `127.0.0.1:2222->2222/tcp` | Remote administration endpoint hosting `weakuser` and `stronguser` accounts |
| **Web Login Portal (`login.html`)** | HTML Web Interface | `http://localhost:8080/login.html` | Authentication frontend evaluated for phishing clone indicators |
| **GPG Keyring & Release Identities** | OpenPGP Trust Store | `~/.gnupg` (`LabOps Team <team@labops.local>`) | Asymmetric encryption and release signature verification infrastructure |
| **Application Environment Config** | Configuration File (`secrets.env`) | `~/CS-Vaje-main/lab06/secrets.env` | Stores runtime database credentials (`DB_PASSWORD`) |
| **Deployment Bootstrapper** | Shell Script (`install.sh`) | `~/CS-Vaje-main/lab07/install.sh` | Automated installation script distributed with `.sha256` and `.sha256.sig` files |

### 2.2 Environment Baseline

```text
=== SYSTEM & DOCKER BASELINE (evidence/01-system-status.txt) ===
Container ID: d1b2caad795d
Image:        labops-project-ssh-server
Command:      "/entrypoint.sh"
Status:       Up 25 hours
Ports:        127.0.0.1:2222->2222/tcp
Name:         ssh_lab

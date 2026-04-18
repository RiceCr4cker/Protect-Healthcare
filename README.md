# Protect-Healthcare

This repository contains the comprehensive technical and governance documentation for a multi-layered cybersecurity "Protect" strategy. The project simulates a targeted attack on a medical clinic and implements advanced defense-in-depth countermeasures.

## 🏥 Context: Polyclinique Vandervelde (PVDV)
PVDV is a fictitious Belgian medical center member of a larger hospital group called `Shirocks`. The project includes:
* **Governance & Risk:** A risk assessment using the **CyFun 2025** (NIS2 alignment) framework, evaluating the clinic from "Small" to "Large" organization perspectives.
* **Scenario:** A "COVID-26" vaccination agenda sabotage attempt by a group of hacktivists.
* **Physical lab:** Implementation on a physical IT architecture composed of 2x Raspberry Pi 4 (the victim's desktop with `Raspbian OS` & a headless ubuntu server to host a `MinIO S3 database`), a hardware switch, and a router to mimic a production environment.

## 🛡️ The Attack & Defense scenario

### Step 1: Physical security & social engineering
* **The Attack:** Break into the "server room" with a plastic card (door bypass) and social engineering for RFID badge cloning using a `Proxmark` based tool.
* **The Defense:** Implementation of physical hardening (anti-intrusion brackets) and a new security policy focused on badge management and personnel awareness.

### Step 2: BadUSB & automated response
* **The Attack:** Injection of a custom ransomware payload (`LockedHealth 3.0`) via an unauthorized HID device (PicoUSB/Flipper Zero/Rubber Ducky/Bash Bunny).
* **The Defense:** 
    * **USBGuard:** Enforces strict hardware whitelisting at the kernel level.
    * **Custom Parser:** A Python script (`usbguard_to_json.py`) was developed to bridge USBGuard logs with the `Wazuh SIEM`.
    * **Wazuh Active Response:** Configured to trigger an immediate `force-shutdown.sh` via a specific sudoers rule when unauthorized hardware activity is detected.

### Step 3: Immutable data survival (ransomware defense)
* **The Attack:** Execution of the "LockedHealth 3.0" ransomware to test data resilience.
* **The Defense:**
    * **Data at rest:** Full disk encryption via **LUKS** (AES-256) with keyfile-based persistence for headless booting.
    * **Data in transit:** Segmented traffic through a **Tailscale** VPN tunnel, allowing only specific ports (9000 for MinIO, 1514-1515 for Wazuh).
    * **Immutability:** Use of **Restic** to push encrypted snapshots to a **MinIO** server configured with **S3 Object Locking** in **Compliance mode** (30-day retention).
    * **Recovery Logic:** A dedicated `bucket-recovery.sh` script using `jq` to identify and remove S3 DeleteMarkers, allowing restoration even after a manual "worst-case" deletion attempt.

## 📁 Repository contents
* **`full_report.pdf`:** Complete project documentation (46 pages). This document is in **French** as required by my academic curriculum.
* **`TheCutestRansomware.txt`:** Malicious script in Flipper Zero/DuckyScript syntax (English). It injects keystrokes to write the ransomware and execute it.
* **Demos:**
    * **`physical_breach.mp4`:** Demonstration of social engineering and physical bypass (English).
    * **`bad-usb_ransomware_execution.mp4`:** Live capture of the ransomware payload (English).
    * **`usbguard_and_disaster-recovery.mp4`:** Quick demo of a defense with USBGuard + Wazuh Active Response and technical walk-through of the recovery process. Narrated in **French** for academic purposes.

## 🛠️ Technical Stack
* **EDR/SIEM:** `Wazuh`
* **Backup & Storage:** `MinIO` & `Restic`
* **Encryption:** `LUKS` & `Openssl`
* **Network & Management:** `Tailscale` & `Docker Compose`

# Automated Infrastructure, Identity, and Compliance Strategy

## 1. Executive Summary

**Current State:** Our environment has historically been managed via manual, "band-aid" fixes. Changes are reactive, and testing is performed on live production systems due to the lack of a high-fidelity lab. This "luck-based" engineering poses significant risk to mission readiness.

**Future State:** We are implementing a Software Defined Datacenter (SDDC) strategy. This project automates the creation of two distinct environments:

- **The Mirror:** A safe, air-gapped replica of Production for destructive testing.
- **The Fortress:** A "Gold Standard" environment pre-hardened to DISA STIGs for future migration.

---

## 2. The Toolchain Strategy

> We are moving from "Click-Ops" to "DevSecOps."

| Tool | Role | The "Why" |
|------|------|-----------|
| **Packer** | The Mold | Guarantees every server starts identical. No more "configuration drift." |
| **Terraform** | The Builder | Builds the network and VMs. Allows us to destroy/rebuild labs in minutes, not days. |
| **Vault** | The Vault | Security. Stores passwords/keys safely. Replaces dangerous hardcoded credentials in scripts. |
| **DSC / M365DSC** | The Brain | Defines the Identity. It "hydrates" the VMs with our Users, OUs, and GPOs automatically. |
| **Ansible** | The Hands | Manages software and updates. Replaces manual RDP sessions and inconsistent patching. |
| **Vagrant** | The Sim | Allows staff to simulate the network on a laptop for safe training before touching the main cluster. |

---

## 3. The Dual-Pipeline Architecture

### Pipeline A: "The Mirror" (Operational Assurance)

**Objective:** Test patches and scripts against real data without risking the real network.

**Method (Metadata Extraction):**

- We do **not** clone heavy, risky hard drives.
- We use *Reverse Engineering* scripts to extract the structure of our domain (Users, Groups, OUs) into code.
- We inject this code into a clean lab, effectively cloning the "Soul" of the domain without the "Body."

### Pipeline B: "The Fortress" (Compliance Baseline)

**Objective:** Create the perfect future state.

**Method (STIG-as-Code):**

- We define DISA STIG standards (Password Complexity, Banners, Auditing) as code.
- We build a fresh domain that enforces these rules from the very first second of boot time.

---

## 4. Implementation Plan

| Phase | Focus | Description |
|-------|-------|-------------|
| **Phase 1** | Foundation | Establish the "Golden Image" pipeline and Vagrant training simulator. |
| **Phase 2** | Infrastructure | Deploy Terraform to handle networking and VM provisioning. |
| **Phase 3** | Security | Integrate Vault to secure all automation credentials. |
| **Phase 4** | Identity | Implement the "Reverse DSC" logic to clone production data safely. |
| **Phase 5** | Operations | Deploy Ansible for automated software management and patching. |
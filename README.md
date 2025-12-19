# Iron-Sandbox: The SDDC "Warpath" Strategy

## 1. Executive Summary

**Objective:** To transition from reactive "Click-Ops" administration to a proactive **Software Defined Datacenter (SDDC)**.

This project implements a dual-pipeline strategy to automate the creation of Active Directory environments. It empowers the team to test destructive changes safely and migrate to a secure, STIG-compliant future state without risking mission-critical production systems.

### The Two Pipelines

| Feature | **Fork 1: The Mirror** (Ops Assurance) | **Fork 2: The Fortress** (Compliance) |
| :--- | :--- | :--- |
| **Goal** | Exact replica of Production for testing patches/scripts. | "Greenfield" build for STIG/Migration planning. |
| **Data Source** | **Reverse Extraction** (Metadata only). No disk cloning. | **Infrastructure as Code** (STIG Baselines). |
| **Security** | Sanitized. No Prod passwords leave the network. | Hardened from boot. 100% Compliance. |

---

## 2. Public Repo Strategy: "The Hollow Shell"

This repository utilizes a **Hollow Shell Architecture** to ensure security while maintaining open-source compatibility.

* **The Code (Public):** The Terraform logic in this repo is generic. It contains **no** hardcoded IP addresses, domain names, or credentials. It serves as the "Engine."
* **The Config (Private):** All environment-specific data (IP schemes, Naming conventions) is stored in a private **HashiCorp Vault** instance on-premises.
* **The Execution:** At runtime, Terraform connects to Vault to "fill in the blanks," injecting the private data into the public logic.

**Benefit:** We can showcase our automation logic publicly on GitHub without exposing internal network topology.

---

## 3. The Integrated Toolchain

We utilize an "Integrate, Not Replace" philosophy. We wrap modern automation around our existing VMware & Windows ecosystem.

* **Packer (The Mold):** Builds a single "Golden Image" (Server 2022) compatible with both **Nutanix/ESXi** (Prod) and **VMware Workstation** (Laptops).
* **Terraform (The Builder):** Defines the network isolation and VM hardware specs. It asks **Vault** for configuration data.
* **Vault (The Security):** Central storage for Secrets (Passwords) AND Configuration (IPs, Hostnames).
* **DSC + M365DSC (The Brain):** "Hydrates" the OS. It reads the extracted production data and rebuilds Users, Groups, and OUs automatically.
* **Ansible (The Hands):** Manages "Day 2" operations (Software installation, Patching, IIS Config) via agentless SSH/WinRM.
* **Vagrant (The Simulator):** Allows staff to spin up a mini-version of the domain on their **VMware Workstation** laptops for safe training.

---

## 4. Architecture Diagram

```text
[Public GitHub Repo]     [Private On-Prem Vault]      [Production/Lab Cluster]
       |                         |                               |
   (Generic Code) + (Config Data & Secrets) --> (Terraform Engine)
       |                                                 |
[Operator Laptop]                                [Nutanix / ESXi]
       |                                                 |
   (Vagrant) <-------------------------------------> (Real Lab)
                                                         |
                           +------------------- (Ansible) -------------+
                                     Config Management & Patching
```

# Iron-Sandbox Implementation Checklist

## Phase 1: The "Flight Simulator" (Local Dev)

*Goal: Enable rookie to test safely on VMware Workstation.*

- [ ] **Packer:** Configure `vmware-iso` builder in `server-2022.pkr.hcl`.
    - [ ] Inject `VMware Tools` drivers.
    - [ ] Enable WinRM for Ansible/Vagrant connection.
- [ ] **Vagrant:** Install `vagrant-vmware-desktop` plugin.
    - [ ] `vagrant plugin install vagrant-vmware-desktop`
    - [ ] `vagrant plugin install vagrant-winrm`
- [ ] **Box Creation:** Output Packer build to a `.box` file.
    - [ ] Run `vagrant box add iron-server-2022 ./output-vmware/package.box`

---

## Phase 2: The Vault (Security Layer)

*Goal: Remove all hardcoded passwords from scripts.*

- [ ] **Deploy:** Spin up HashiCorp Vault (Docker or VM).
- [ ] **Secrets:**
    - [ ] Store Nutanix/vSphere API User.
    - [ ] Store Domain Admin Credentials.
- [ ] **Integration:** Configure Terraform `provider "vault"` to read secrets dynamically.

---

## Phase 3: Infrastructure (Terraform)

*Goal: Define the "Skeleton" of the lab.*

- [ ] **Network:** Define Isolated vSwitch (Air-Gapped).
- [ ] **Compute:** Define DC hardware specs (4 vCPU, 16GB RAM).
- [ ] **Toggles:** Create `var.environment` input to switch between "Mirror" and "Clean" builds.

---

## Phase 4: Identity & Data (The "Brains")

*Goal: Define the content of the Active Directory.*

- [ ] **Fork 1 (Mirror):**
    - [ ] Run `Export-M365DSCConfiguration` on Prod to get `prod-export.ps1`.
    - [ ] Script the "IP Address Sanitizer" (Regex replace Prod IPs -> Lab IPs).
- [ ] **Fork 2 (Clean):**
    - [ ] Define STIG Baseline using `PowerSTIG` DSC module.
    - [ ] Define "Golden State" OU structure in code.

---

## Phase 5: Configuration (Ansible & DSC)

*Goal: Software management.*

- [ ] **DSC:** Write the "Hydration" script that runs on first boot (Promote DC, Create OUs).
- [ ] **Ansible:** Write `site.yml` playbook.
    - [ ] Task: Install Chrome, Firefox, Notepad++.
    - [ ] Task: Apply latest Windows Updates.
- [ ] **Glue:** Configure Terraform `local-exec` to trigger Ansible once VM is up.
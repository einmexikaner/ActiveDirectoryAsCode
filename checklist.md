# Iron-Sandbox Implementation Checklist

## Phase 0: Research & Reference Materials

*Official vendor documentation for the "Iron-Sandbox" toolchain.*

### Infrastructure (The "Skeleton")

* **Packer (VMware ISO Builder):** [HashiCorp Developer - VMware ISO](https://developer.hashicorp.com/packer/integrations/hashicorp/vmware/latest/components/builder/iso)
  * *Docs for building the initial VM from a Windows ISO.*
* **Terraform (vSphere/ESXi Provider):** [Terraform Registry - vSphere](https://registry.terraform.io/providers/hashicorp/vsphere/latest/docs)
  * *Docs for managing VMs on your production cluster.*
* **Vagrant (VMware Provider):** [Vagrant Docs - VMware Desktop](https://developer.hashicorp.com/vagrant/docs/providers/vmware)
  * *Required for running the lab on Workstation Pro.*

### Security & Secrets (The "Vault")

* **Vault (KV Secrets Engine):** [Vault Docs - KV Engine](https://developer.hashicorp.com/vault/docs/secrets/kv)
  * *The specific storage engine we are using for Config/Passwords.*
* **Terraform (Vault Provider):** [Terraform Registry - Vault](https://registry.terraform.io/providers/hashicorp/vault/latest/docs)
  * *How to make Terraform talk to Vault.*

### Identity & Configuration (The "Brain")

* **Microsoft365DSC (Extraction):** [Official M365DSC Documentation](https://microsoft365dsc.com/)
  * *Refer to the "ReverseDSC" or "Export" capabilities.*
* **PowerSTIG (Compliance):** [PowerSTIG GitHub Wiki](https://github.com/microsoft/PowerSTIG/wiki)
  * *Documentation on generating DSC configs from DISA XCCDF.*
* **Ansible (Windows Guide):** [Ansible Docs - Windows Setup](https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html)
  * *Specifics on setting up WinRM and writing Playbooks for Windows.*

---

## Phase 1: The "Flight Simulator" (Local Dev)

*Goal: Enable the rookie to test safely on VMware Workstation.*

* [ ] **Packer:** Configure `vmware-iso` builder in `server-2022.pkr.hcl`.
  * [ ] Inject `VMware Tools` drivers.
  * [ ] Enable WinRM for Ansible/Vagrant connection.
* [ ] **Vagrant:** Install `vagrant-vmware-desktop` plugin.
  * [ ] `vagrant plugin install vagrant-vmware-desktop`
  * [ ] `vagrant plugin install vagrant-winrm`
* [ ] **Box Creation:** Output Packer build to a `.box` file.
  * [ ] Run `vagrant box add iron-server-2022 ./output-vmware/package.box`

---

## Phase 2: The Vault (The "Hollow Shell")

*Goal: Abstract all data so the code can be public.*

* [ ] **Deploy:** Spin up HashiCorp Vault (Docker or VM).
* [ ] **Secrets (The Keys):**
  * [ ] Store Nutanix/vSphere API User & Pass.
  * [ ] Store Domain Admin Credentials.
* [ ] **Configuration (The Data):**
  * [ ] Define the JSON Schema (IPs, Subnets, Domain Names, Gateways).
  * [ ] Populate Vault Path: `secret/iron-sandbox/config`.
* [ ] **Integration:** Configure Terraform `provider "vault"` to authenticate and read these paths.

---

## Phase 3: Infrastructure (Terraform)

*Goal: Define the "Skeleton" of the lab using Vault data.*

* [ ] **Refactor:** Replace hardcoded strings in `.tf` files with `data.vault_generic_secret` lookups.
* [ ] **Network:** Define Isolated vSwitch (Air-Gapped).
* [ ] **Compute:** Define DC hardware specs (4 vCPU, 16GB RAM).
* [ ] **Toggles:** Create `var.environment` input to switch between "Mirror" and "Clean" builds.

---

## Phase 4: Identity & Data (The "Brains")

*Goal: Define the content of the Active Directory.*

* [ ] **Fork 1 (Mirror):**
  * [ ] Run `Export-M365DSCConfiguration` on Prod to get `prod-export.ps1`.
  * [ ] Script the "IP Address Sanitizer" (Regex replace Prod IPs -> Lab IPs).
* [ ] **Fork 2 (Clean):**
  * [ ] Define STIG Baseline using `PowerSTIG` DSC module.
  * [ ] Define "Golden State" OU structure in code.

---

## Phase 5: Configuration (Ansible & DSC)

*Goal: Software management.*

* [ ] **DSC:** Write the "Hydration" script that runs on first boot (Promote DC, Create OUs).
* [ ] **Ansible:** Write `site.yml` playbook.
  * [ ] Task: Install Chrome, Firefox, Notepad++.
  * [ ] Task: Apply latest Windows Updates.
* [ ] **Glue:** Configure Terraform `local-exec` to trigger Ansible once VM is up.

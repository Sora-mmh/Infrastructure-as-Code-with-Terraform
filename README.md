# Infrastructure-as-Code-with-Terraform

The listed projects in this module cover:

---

### **1. Full AWS Stack**
- **VPC, subnets, route tables, security groups** – all in HCL.  
- **EC2 instance** – dynamic AMI lookup, auto-generated SSH key, remote-exec to install **Docker + nginx**.  
- **Outputs** printed the public DNS for instant browser access.

---

### **2. Provisioners**
- Used `remote-exec`, `file`, and `local-exec` to bootstrap instances.  
- **Best-practice warning**: prefer **Ansible / cloud-init** for real-world config management.

---

### **3. Re-usable Modules**
- Extracted **network** and **webserver** configs into **modules**.  
- **Root module** now calls `module "vpc"` and `module "web"`—clean, DRY, shareable.

---

### **4. EKS in One Shot**
- **VPC module + EKS module** from the **Terraform Registry**.  
- **Kubernetes provider** wired to the new EKS cluster.  
- Deployed a simple **nginx pod**; destroyed with `terraform destroy`.

---

### **5. CI/CD-Driven Infra**
- **Jenkins pipeline** installs Terraform inside the agent container.  
- Pipeline stages:  
  1. **Provision EC2** via Terraform  
  2. **Remote-exec script** installs Docker & Docker-Compose  
  3. **Deploy app** with `docker-compose up -d`  
- Secrets (SSH key, DockerHub creds) stored as **Jenkins credentials**.

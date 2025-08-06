# Project: Automated & Secure Server Provisioning with Ansible

This project demonstrates a complete Infrastructure as Code (IaC) workflow for automatically provisioning a bare Ubuntu server into a fully configured and secured web server. It utilizes **Ansible** to perform configuration management, ensuring the process is repeatable, reliable, and idempotent.

The project is structured using **Ansible Roles**, a best practice for creating modular and reusable automation code. It goes beyond basic setup to include key security hardening measures like **SSH key-only authentication** and **automated daily security updates**.

A significant part of this project involved debugging real-world challenges, such as managing Docker container capabilities (`NET_ADMIN`) and resolving cross-platform issues between Windows, WSL, and Ansible.

---

![Ansible](https://img.shields.io/badge/ansible-automation-blue) ![Docker](https://img.shields.io/badge/docker-provisioning-lightblue) ![License: MIT](https://img.shields.io/badge/license-MIT-green)

## Core Concepts & Skills Demonstrated

- **Configuration Management:** Used Ansible, an industry-standard tool, to manage the state of a server from code.
- **Security Hardening:**
  - Implemented **SSH key-only authentication**, disabling password-based logins for improved security.
  - Configured the **UFW (Uncomplicated Firewall)** to deny all traffic by default and only allow specific ports.
  - Set up a **cron job** for automatic, unattended daily security updates.
  - Created a non-root user with passwordless `sudo` privileges for running administrative tasks.
- **Idempotency:** The Ansible playbook is designed to be idempotent. It can be run repeatedly, but will only make changes if the server's configuration has drifted from the desired state.
- **Ansible Roles:** The automation logic is encapsulated in a reusable `webserver` role, demonstrating best practices for scalable and maintainable Ansible projects.
- **Infrastructure as Code (IaC):** The entire server setup is defined in version-controlled YAML files.
- **Advanced Debugging:**
  - Diagnosed and solved a container permission error by granting the `NET_ADMIN` capability to manage the firewall.
  - Resolved SSH `Permission denied` errors by ensuring correct file permissions and ownership for SSH keys inside the container.
  - Adapted the workflow for a WSL development environment, including managing dependencies (`sshpass`) and command-line flags.

---

## Technologies Used

- **Configuration Management:** Ansible
- **Containerization:** Docker, Docker Compose
- **Development Environment:** WSL (Windows Subsystem for Linux)
- **Operating System:** Ubuntu 20.04
- **Web Server:** Nginx
- **Firewall:** UFW
- **System Tools:** SSH, Cron

---

## How to Run This Project

### Prerequisites

- A Windows machine with **WSL2** installed and a Linux distribution (e.g., Ubuntu).
- Docker Desktop installed on Windows, with the WSL2 integration enabled.
- Ansible and `sshpass` installed **inside your WSL environment**:

```bash
# Run these commands inside your WSL terminal
sudo apt-get update
sudo apt-get install -y ansible sshpass
```

---

#### Step 1: Generate and Prepare SSH Key

_If you don't have an SSH key in WSL, create one. Then, copy the public key into this project's directory._

```bash
# 1. Generate the key (press Enter for all prompts)
ssh-keygen -t rsa -b 4096
```

```bash
# 2. Then copy your public key into the project directory.
cp ~/.ssh/id_rsa.pub .
```

_This builds the container, embedding your public key and granting it the necessary `NET_ADMIN` capability._

---

#### Step 2: Build and Start the Target Server

The container is built with:

- Your public SSH key
- The necessary Docker capabilities (e.g., `NET_ADMIN` for UFW)

Start it with:

```bash
docker-compose up -d
```

---

#### Step 3: Run Ansible to Configure the Server

This command executes the playbook, connecting via your SSH key.

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini playbook.yml
```

---

#### Step 4: Verify the Setup

Open your web browser and navigate to:
`http://localhost:8080`
_You should see a welcome page that says "Server Configured by Ansible!"_

---

#### Step 5: Clean Up

To stop and remove the container, run:

```bash
docker-compose down
```

---

## Troubleshooting

❌ Ansible SSH Fails: Permission denied (publickey)

_This means the SSH key used by Ansible doesn't match the one embedded in the container._

### Fix:

Confirm your WSL public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy it into the project folder:

```bash
cp ~/.ssh/id_rsa.pub .
```

Rebuild the container to re-embed the correct key:

```bash
docker-compose down
docker-compose up --build -d
```

Manually test SSH:

```bash
ssh -i ~/.ssh/id_rsa ansible@127.0.0.1 -p 2222
```

---

❌ ansible.cfg ignored: "world-writable directory" warning

Ansible may show in commandline:
_"Ansible is being run in a world writable directory ... ignoring it as an ansible.cfg source."_

Why: You're running Ansible inside a `/mnt/d/...` folder from Windows, which WSL mounts as world-writable for compatibility.

#### Fix (optional):

> You can ignore this for personal use.

Or, move the project into your WSL home directory (~/projects/...) for tighter control.

---

❌ Port 8080 not working in browser

Ensure:
The container is running:

```bash
docker ps
```

The Nginx service is active inside the container:

```bash
docker exec -it ansible-target-server systemctl status nginx
```

Or try:

```bash
docker exec -it ansible-target-server curl localhost
```

---

❌ UFW Not Allowing Connections

If UFW is blocking access:

```bash
docker exec -it ansible-target-server ufw status
```

Expected output:

```vbnet
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
If not, re-run your Ansible playbook or add UFW rules manually.
```

---

❌ Cannot SSH into Container (manual test fails)
Symptoms:

```bash
ssh -i ~/.ssh/id_rsa ansible@127.0.0.1 -p 2222
```

Fails with:

Permission denied (publickey)

### Fix

- You forgot to copy your public key into the project directory
- Or it's outdated (regenerated keys)

Re-copy and rebuild the container as described above

---

### Clean Reset (when everything’s broken)

To start fresh:

```bash
docker-compose down -v
docker system prune -af
```

Then rebuild:

```bash
docker-compose up --build -d
```

---

## 📌 Reminder

> **This is a local-use setup — not intended for production-level internet exposure. If exposing externally, implement:**

- Fail2Ban or SSH rate limiting
- HTTPS (Let’s Encrypt)
- Ansible Vault for any secrets

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

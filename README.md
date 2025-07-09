# Project: Automated & Secure Server Provisioning with Ansible

This project demonstrates a complete Infrastructure as Code (IaC) workflow for automatically provisioning a bare Ubuntu server into a fully configured and secured web server. It utilizes **Ansible** to perform configuration management, ensuring the process is repeatable, reliable, and idempotent.

The project is structured using **Ansible Roles**, a best practice for creating modular and reusable automation code. It goes beyond basic setup to include key security hardening measures like **SSH key-only authentication** and **automated daily security updates**.

A significant part of this project involved debugging real-world challenges, such as managing Docker container capabilities (`NET_ADMIN`) and resolving cross-platform issues between Windows, WSL, and Ansible.

---

## Core Concepts & Skills Demonstrated

*   **Configuration Management:** Used Ansible, an industry-standard tool, to manage the state of a server from code.
*   **Security Hardening:**
    *   Implemented **SSH key-only authentication**, disabling password-based logins for improved security.
    *   Configured the **UFW (Uncomplicated Firewall)** to deny all traffic by default and only allow specific ports.
    *   Set up a **cron job** for automatic, unattended daily security updates.
    *   Created a non-root user with passwordless `sudo` privileges for running administrative tasks.
*   **Idempotency:** The Ansible playbook is designed to be idempotent. It can be run repeatedly, but will only make changes if the server's configuration has drifted from the desired state.
*   **Ansible Roles:** The automation logic is encapsulated in a reusable `webserver` role, demonstrating best practices for scalable and maintainable Ansible projects.
*   **Infrastructure as Code (IaC):** The entire server setup is defined in version-controlled YAML files.
*   **Advanced Debugging:**
    *   Diagnosed and solved a container permission error by granting the `NET_ADMIN` capability to manage the firewall.
    *   Resolved SSH `Permission denied` errors by ensuring correct file permissions and ownership for SSH keys inside the container.
    *   Adapted the workflow for a WSL development environment, including managing dependencies (`sshpass`) and command-line flags.

---

## Technologies Used

*   **Configuration Management:** Ansible
*   **Containerization:** Docker, Docker Compose
*   **Development Environment:** WSL (Windows Subsystem for Linux)
*   **Operating System:** Ubuntu 20.04
*   **Web Server:** Nginx
*   **Firewall:** UFW
*   **System Tools:** SSH, Cron

---

## How to Run This Project

**Prerequisites:**
*   A Windows machine with **WSL2** installed and a Linux distribution (e.g., Ubuntu).
*   Docker Desktop installed on Windows, with the WSL2 integration enabled.
*   Ansible and `sshpass` installed **inside your WSL environment**:
    ```bash
    # Run these commands inside your WSL terminal
    sudo apt-get update
    sudo apt-get install -y ansible sshpass
    ```

**Step 1: Generate and Prepare SSH Key**
If you don't have an SSH key in WSL, create one. Then, copy the public key into this project's directory.

```bash
# 1. Generate the key (press Enter for all prompts)
ssh-keygen -t rsa -b 4096
```
```bash
# 2. Copy the public key to this folder
cp ~/.ssh/id_rsa.pub.
```
This builds the container, embedding your public key and granting it the necessary `NET_ADMIN` capability.

**Step 3: Provision the Server with Ansible**

This command executes the playbook, connecting via your SSH key.

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini playbook.yml
```

**Step 4: Verify the Result**
Open your web browser and navigate to:
`http://localhost:8080`

You should see a welcome page that says "Server Configured by Ansible!"

**Step 5: Clean Up**

To stop and remove the container, run:
```bash
docker-compose down
```
## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
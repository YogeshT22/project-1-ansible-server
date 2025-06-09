# Project: Automated Server Provisioning with Ansible

This project demonstrates a complete Infrastructure as Code (IaC) workflow for automatically provisioning a bare Ubuntu server into a fully functional web server. It utilizes **Ansible** to perform configuration management, ensuring the process is repeatable, reliable, and idempotent. The "server" is a Docker container with an SSH daemon, allowing for safe, local, and disposable practice of real-world server automation techniques.

The project is structured using **Ansible Roles**, a best practice for creating modular, reusable, and maintainable automation code. It also documents solutions to real-world challenges encountered when running Ansible in a Windows/WSL development environment.

---

## Core Concepts & Skills Demonstrated

*   **Configuration Management:** Used Ansible, an industry-standard tool, to manage the state of a server from code.
*   **Idempotency:** The Ansible playbook is designed to be idempotent. It can be run repeatedly, but will only make changes if the server's configuration has drifted from the desired state.
*   **Infrastructure as Code (IaC):** The entire server setup—from package installation to firewall configuration—is defined in version-controlled YAML files.
*   **Ansible Roles:** The automation logic is encapsulated in a reusable `webserver` role, demonstrating best practices for scalable and maintainable Ansible projects. This includes separating tasks, handlers, templates, and variables.
*   **Server Hardening & Docker Capabilities:** Performed security hardening with **UFW**, which required diagnosing a container permission error and solving it by granting the `NET_ADMIN` capability to the container in Docker Compose.
*   **Environment-Specific Debugging:** Solved common issues when using Ansible on Windows, including moving to **WSL (Windows Subsystem for Linux)**, installing `sshpass`, and working around WSL file permission issues that affected `ansible.cfg`.
*   **Templating & Service Management:** Utilized Jinja2 templates for configuration and managed system services (`nginx`, `ufw`) to ensure they are started and enabled on boot.

---

## Technologies Used

*   **Configuration Management:** Ansible
*   **Containerization:** Docker, Docker Compose
*   **Development Environment:** WSL (Windows Subsystem for Linux)
*   **Operating System:** Ubuntu 20.04
*   **Web Server:** Nginx
*   **Firewall:** UFW

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

**Step 1: Start the Target Server**

From your WSL terminal, navigate to the project directory and run:

```bash
docker-compose up --build -d
```
This builds and runs the container with the necessary NET_ADMIN capability to manage the firewall.

**Step 2: Provision the Server with Ansible:**

The ```ansible.cfg``` file may be ignored by Ansible due to WSL's default file permissions. Therefore, we provide the configuration directly in the command.

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini playbook.yml --ask-pass
```
You will be prompted for the SSH password. The password is ```ansible```.

**Step 3: Verify the Result:**

The playbook installs Nginx, and the Docker Compose file maps port 8080 on your local machine to the container's port 80.

Open your web browser and navigate to:
```http://localhost:8080```

You should see a welcome page that says "Server Configured by Ansible!"

**Step 4: Clean Up:**

To stop and remove the container, run:

```bash
docker-compose down
```

**Architecture Diagram**
```mermaid
graph LR
    subgraph "Your Local Machine (Control Node)"
        A[Ansible Playbook] -- "Runs on" --> U[You in WSL]
        U -- "Executes `ansible-playbook`" --> P[playbook.yml]
    end

    P -- "Connects via SSH on localhost:2222" --> S[Docker Container: Server]

    subgraph "Docker Container (Managed Node with NET_ADMIN)"
        style S fill:#f9f,stroke:#333,stroke-width:2px
        S -- "1. Installs Nginx, UFW" --> P1[Packages]
        S -- "2. Creates index.html" --> F1[Files]
        S -- "3. Configures nginx.conf from Template" --> F2[Templates]
        S -- "4. Sets up Firewall Rules" --> F3[Firewall]
        S -- "5. Starts & Enables Services" --> P2[Services]
    end

    U -- "Verifies at http://localhost:8080" --> S
    ```

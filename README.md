# Ansible Lab Environment

A self-contained Docker-based lab to practice Ansible automation across Linux (SSH) and Windows (WinRM) targets.

## Overview

This project provisions three containers via Docker Compose:

- **ansible-controller**: Control node with Ansible installed; mounts the `ansible/` project directory and private SSH key.
- **linux-client**: Ubuntu-based target host configured for key-based SSH access.
- **windows-client**: Windows Server Core 2022 target reachable over WinRM.

All services share an isolated bridge network `labnet` so they can reach each other by container name.

## Repository Layout

```
ansible/
  ansible.cfg            # Core Ansible configuration
  inventory/hosts        # Inventory defining linux-client & windows-client
  group_vars/            # Group variable files (all, linux, windows)
  host_vars/             # (Optional) per-host variable directory
  playbooks/             # Playbooks for Linux & Windows automation tasks
  files/                 # Static files (e.g., test.txt) for copy modules
  templates/             # Jinja2 templates (add as needed)
controller-shell.sh      # Helper script to open a shell in controller
docker-compose.yaml      # Service & network definitions
ansible-controller/      # Dockerfile for controller image
linux-client/            # Dockerfile for Linux target image
windows-client/          # Dockerfile for Windows target image
keys/                    # SSH key pair used for Linux auth (lab only)
```

## Prerequisites

- Docker & Docker Compose installed on host (macOS compatible)
- Internet access (for pulling base images / Chocolatey packages)

## Quick Start

From the project root:

```sh
docker-compose up -d
./controller-shell.sh   # or: docker exec -it ansible-controller bash
```

Validate connectivity:

```sh
cd /root/ansible
ansible all -m ping
```

Expected: Linux host responds via `ping` (SSH), Windows host via WinRM.

## Ansible Configuration Highlights (`ansible/ansible.cfg`)

- Sets inventory path to `inventory/hosts`
- Disables host key checking for lab convenience
- Uses private key at `/root/.ssh/id_rsa` inside controller
- Enables privilege escalation if configured

## Inventory (`ansible/inventory/hosts`)

Defines two groups/hosts:

- `linux-client` (SSH, key-based)
- `windows-client` (WinRM)

Customize host vars or add additional hosts by extending this file and adding matching containers.

## Variables

Group variable files:

- `group_vars/all.yml` – Shared defaults (e.g., common users/packages)
- `group_vars/linux.yml` – Linux-specific settings (package lists, service names)
- `group_vars/windows.yml` – Windows-specific settings (Chocolatey packages, services)


## Playbooks

### Linux

- `linux_copy_file.yml` – Copies test file to target.
- `linux_create_users.yml` – Creates users with generated passwords.
- `linux_install_nginx.yml` – Installs and configures Nginx.
- `linux_install_packages.yml` – Installs base utilities (curl, htop, tree, vim, etc.).
- `linux_run_bash_debug.yml` – Executes shell commands and demonstrates debug output.
- `linux_sshd_config_handler.yml` – Adjusts SSH daemon config with handlers.

### Windows

- `windows_copy_file_debug.yml` – Copies files and uses debug output.
- `windows_create_users_loop.yml` – Creates multiple local users via loops.
- `windows_install_software_handler.yml` – Installs software (Chocolatey) with handlers.
- `windows_powershell_debug.yml` – Runs PowerShell commands and captures output.
- `windows_restart_services_loop.yml` – Restarts services iteratively.

Run any playbook from the controller:

```sh
ansible-playbook playbooks/<playbook-name>.yml
```

Example:

```sh
ansible-playbook playbooks/linux_install_nginx.yml
```

## Authentication

### Linux (`linux-client`)

- SSH key: `keys/id_rsa` mounted into controller at `/root/.ssh/id_rsa`
- Public key mounted into Linux container as authorized key

### Windows (`windows-client`)

- WinRM over HTTP (port 5985) inside Docker network
- Credentials defined in `group_vars/windows.yml` (lab defaults)

## Adding New Automation

1. Add variables: create or modify files under `group_vars/` or `host_vars/`.
2. Add templates: store Jinja2 templates in `templates/`.
3. Write a new playbook in `playbooks/` referencing inventory groups.
4. Run with `ansible-playbook` from controller.

## Troubleshooting

Check container status:

```sh
docker-compose ps
```

Logs for a specific service:

```sh
docker-compose logs ansible-controller
```

Restart a service:

```sh
docker-compose restart linux-client
```

Rebuild environment:

```sh
docker-compose down
docker-compose up -d --build
```

Connectivity test details:

```sh
ansible linux-client -m ping -vvv
ansible windows-client -m win_ping -vvv
```

## Cleaning Up

Stop and remove containers:

```sh
docker-compose down
```

Remove with volumes/network:

```sh
docker-compose down -v
```

## Security Disclaimer

This setup is **for lab and learning only**:

- Hardcoded credentials & unencrypted WinRM (HTTP)
- Private SSH key stored in repository
- Host key checking disabled

Do **not** reuse these settings in production. For real deployments: encrypt secrets (e.g., Ansible Vault), enforce TLS for WinRM, rotate keys, enable host key checking.

---

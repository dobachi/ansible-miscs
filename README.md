# ansible-miscs

Comprehensive Ansible playbooks and roles for automating system configuration, development environment setup, and infrastructure management across Linux, WSL, and Windows platforms.

## Overview

This repository contains:
- **49 playbooks** for system configuration and operations
- **78 roles** covering development tools, programming languages, infrastructure, and utilities
- Support for multiple platforms: Linux, WSL (Windows Subsystem for Linux), Raspberry Pi, and Windows

## Quick Start

```bash
# Basic WSL configuration
ansible-playbook -i hosts playbooks/conf/wsl/wsl.yml

# Install Docker (requires Administrator privileges)
ansible-playbook -i hosts playbooks/conf/wsl/docker.yml

# Configure a specific host
ansible-playbook -i hosts playbooks/conf/linux/basic.yml -e server=hostname
```

## Project Structure

```
ansible-miscs/
├── playbooks/
│   ├── conf/          # Configuration playbooks (40 total)
│   │   ├── fileserver/    # File server setup
│   │   ├── kafka_pseudo/  # Kafka pseudo-distributed mode
│   │   ├── linux/         # Linux system configurations (27 playbooks)
│   │   ├── raspberrypi/   # Raspberry Pi specific
│   │   ├── win/           # Windows configurations
│   │   └── wsl/           # WSL configurations (8 playbooks)
│   └── operation/     # Operational playbooks (9 total)
│       ├── kafka_pseudo/  # Kafka service management
│       └── mxd/           # MXD service operations
├── roles/            # Reusable Ansible roles (78 total)
├── hosts            # Linux/Unix inventory
├── hosts.win        # Windows inventory
└── ansible.cfg      # Ansible configuration
```

## Key Playbooks

### WSL Configuration
- `playbooks/conf/wsl/wsl.yml` - Base WSL setup
- `playbooks/conf/wsl/docker.yml` - Docker installation (requires Administrator)
- `playbooks/conf/wsl/git_clone_sources.yml` - Clone development repositories
- `playbooks/conf/wsl/japanese.yml` - Japanese language support

### Linux Development Environment
- `playbooks/conf/linux/basic.yml` - Essential system packages
- `playbooks/conf/linux/intellij.yml` - IntelliJ IDEA installation
- `playbooks/conf/linux/pycharm.yml` - PyCharm installation
- `playbooks/conf/linux/docker.yml` - Docker setup
- `playbooks/conf/linux/kind.yml` - Kubernetes in Docker
- `playbooks/conf/linux/terraform.yml` - Infrastructure as Code

### Infrastructure Services
- `playbooks/conf/fileserver/fileserver.yml` - File server configuration
- `playbooks/conf/kafka_pseudo/kafka_broker.yml` - Kafka broker setup
- `playbooks/operation/kafka_pseudo/start_all.yml` - Start Kafka cluster

## Available Roles

### Development Tools & IDEs (12 roles)
- IDEs: `intellij`, `pycharm`, `android_studio`, `clion`, `goland`, `vscode`
- Editors: `vim`, `vim_config`, `gvim`, `zed`

### Programming Languages (11 roles)
- Java: `jdk`, `maven`, `sbt`, `gradle`
- Python: `pyenv`, `pip3`, `pipenv`, `anaconda`
- Go: `go`, `golangci`, `goreturns`
- Node.js: `nodejs`

### Container & Orchestration (6 roles)
- `docker`, `kind`, `minikube`, `helm`, `stern`

### Infrastructure & DevOps (8 roles)
- `terraform`, `vagrant`, `jenkins`, `kafka_pseudo`, `spark`

### System & Utilities
- Base system: `commons`, `bash`, `tmux`
- WSL specific: `wsl_common`, `wsl_gui`
- Browsers: `chrome`, `chromium`, `firefox`
- Japanese support: `japanese`, `ipafont`

## Usage Examples

### Running Playbooks

```bash
# Default inventory group
ansible-playbook -i hosts playbooks/conf/linux/docker.yml

# Specific host
ansible-playbook -i hosts playbooks/conf/linux/terraform.yml -e server=myhost

# Windows hosts
ansible-playbook -i hosts.win playbooks/conf/win/win.yml
```

### Testing

```bash
# Syntax check
ansible-playbook --syntax-check playbooks/conf/wsl/wsl.yml

# Lint check
ansible-lint playbooks/conf/wsl/wsl.yml

# Test specific role
cd roles/docker
ansible-playbook -i tests/inventory tests/test.yml
```

## Requirements

- Ansible 2.9+
- Python 3.6+
- SSH access to target hosts
- WinRM configured for Windows hosts

## License

This project is licensed under the MIT License - see the LICENSE file for details.

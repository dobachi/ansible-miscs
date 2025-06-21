# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an Ansible repository for automating system configuration, particularly focused on WSL (Windows Subsystem for Linux) and various development tools. The repository contains playbooks and roles for installing and configuring software packages like Docker, IDEs, programming languages, and infrastructure tools.

## Common Commands

### Running Playbooks

```bash
# Execute a playbook with default inventory
ansible-playbook -i hosts playbooks/conf/wsl/wsl.yml

# Execute with specific server target
ansible-playbook -i hosts playbooks/conf/wsl/docker.yml -e server=hostname

# Windows hosts (requires Administrator for some tasks)
ansible-playbook -i hosts.win playbooks/conf/win/basic.yml
```

### Testing and Validation

```bash
# Syntax check
ansible-playbook --syntax-check playbooks/conf/wsl/wsl.yml

# Lint check (if ansible-lint is installed)
ansible-lint playbooks/conf/wsl/wsl.yml

# Test a specific role
cd roles/role_name
ansible-playbook -i tests/inventory tests/test.yml
```

## Architecture

### Directory Structure
- `playbooks/conf/` - Configuration playbooks organized by target environment (wsl/, linux/, win/, etc.)
- `playbooks/operation/` - Operational playbooks for service management
- `roles/` - Reusable Ansible roles following Galaxy structure
- `hosts` - Inventory file for Linux/Unix hosts
- `hosts.win` - Inventory file for Windows hosts

### Key Patterns
1. **Host targeting**: Playbooks use `"{{ server | default('group_name') }}"` pattern for flexible host targeting
2. **Role-based design**: Most functionality is encapsulated in roles for reusability
3. **Distribution handling**: Roles include distribution-specific tasks via conditional includes
4. **Version management**: Some roles (like kind, terraform) use alternatives for version management

### Common Playbook Groups
- **wsl** - Default group for WSL configurations
- **fileservers** - File server configurations
- **localhost** - Local machine operations
- **thk** - Windows hosts group
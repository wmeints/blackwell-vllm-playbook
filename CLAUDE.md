# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible playbook for provisioning and configuring vLLM on Dell Pro Max GB10 machines (NVIDIA Blackwell GPUs). The goal is to automate the full setup so developers can quickly get a vLLM inference server running and accessible from laptops on the same internal network.

Key features:
- Installs vLLM and its dependencies (NVIDIA drivers and CUDA are preinstalled on the GB10)
- Configures vLLM as a systemd service exposing an OpenAI-compatible API
- Supports user-configurable API key for authentication
- Binds to the machine's network interface for internal network access

## Architecture

This is an Ansible project. The expected layout follows standard Ansible conventions:

- `inventory/` — host definitions for target GB10 machines
- `roles/` — modular roles (e.g., vllm, firewall)
- `group_vars/` / `host_vars/` — variable files including API key configuration
- `playbooks/` or root-level `.yml` — main playbook entrypoints
- `templates/` — Jinja2 templates for config files and systemd units

## Common Commands

```bash
# Run the full playbook against inventory
ansible-playbook -i inventory/hosts playbook.yml

# Run with a specific API key
ansible-playbook -i inventory/hosts playbook.yml -e "vllm_api_key=YOUR_KEY"

# Limit to a single host
ansible-playbook -i inventory/hosts playbook.yml --limit hostname

# Check mode (dry run)
ansible-playbook -i inventory/hosts playbook.yml --check

# Lint the playbook
ansible-lint

# Syntax check
ansible-playbook playbook.yml --syntax-check
```

## Key Design Decisions

- Target platform: Ubuntu (Dell Pro Max GB10 ships with Ubuntu)
- vLLM runs as a systemd service for automatic restarts and log management
- API key is passed as an Ansible variable (`vllm_api_key`) and injected into the vLLM service configuration
- The server binds to `0.0.0.0` to allow internal network access; firewall rules should restrict to the local subnet

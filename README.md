# Blackwell vLLM Setup

Ansible playbook for provisioning and configuring [vLLM](https://docs.vllm.ai/)
on Dell Pro Max GB10 machines equipped with NVIDIA Blackwell GPUs. Automates the
full setup so you can quickly get a vLLM inference server running and accessible
from other machines on your internal network.

## Prerequisites

- **Ubuntu** — the Dell Pro Max GB10 ships with Ubuntu preinstalled
- **NVIDIA drivers and CUDA** — preinstalled on the GB10, no manual setup needed
- **Ansible** — installed locally on the machine
  (see [installation docs](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))

## Quick Start

Clone the repository on the GB10 machine:

```bash
git clone https://github.com/wmeints/blackwell-vllm-playbook
cd blackwell-vllm-playbook
```

Run the playbook locally:

```bash
ansible-playbook -i inventory/hosts playbook.yml --connection=local
```

To set a custom API key for the vLLM server:

```bash
ansible-playbook -i inventory/hosts playbook.yml --connection=local -e "vllm_api_key=YOUR_KEY"
```

### Other useful commands

```bash
# Dry run (check mode)
ansible-playbook -i inventory/hosts playbook.yml --connection=local --check

# Syntax check
ansible-playbook playbook.yml --syntax-check

# Lint
ansible-lint
```

## What the Playbook Does

1. **Installs uv and vLLM** — installs uv, sets up a Python environment, and
   installs vLLM with its dependencies
2. **Configures a systemd service** — runs vLLM as a managed service with
   automatic restarts and log management, exposing an OpenAI-compatible API
3. **Sets up firewall rules** — restricts access to the vLLM API to the local subnet

## Usage

Once the playbook has run, the vLLM server exposes an OpenAI-compatible API at
`http://<gb10-ip>:8000`. You can verify it's running with:

```bash
curl http://<gb10-ip>:8000/v1/models
```

### Connecting opencode

[opencode](https://github.com/opencode-ai/opencode) can use the vLLM server as
a backend via its OpenAI-compatible API. Add the following to your
`opencode.json` configuration file:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "vllm": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "vllm",
      "options": {
        "baseURL": "http://<gb10-ip>:8000/v1",
        "apiKey": "<configured-api-key>"
      },
      "models": {
        "Qwen/Qwen3-Coder-Next-FP8": {
          "name": "qwen3-coder-next"
        }
      }
    }
  }
}
```

Replace `<gb10-ip>` with the IP address of your GB10 machine.

## Playbook Configuration

Key variables that can be customized (via `-e` flag or in variable files under `group_vars`/`host_vars`):

| Variable       | Description                                            | Default   |
| -------------- | ------------------------------------------------------ | --------- |
| `vllm_api_key` | API key for authenticating requests to the vLLM server | —         |
| `vllm_model`   | Model to serve (e.g., `Qwen/Qwen3-Coder-Next-FP8`)     | —         |
| `vllm_host`    | Address the server binds to                            | `0.0.0.0` |
| `vllm_port`    | Port the server listens on                             | `8000`    |

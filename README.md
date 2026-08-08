# Raspberry Pi 5 AI Cluster Automation with Ansible

## Description
This repository provides Ansible automation for a Raspberry Pi 5 AI cluster with a `llama.cpp` RPC stack and a Hermes Agent node.

## Prerequisites
- Ansible 2.10 or higher.
- Python 3 with `pip`.
- SSH access to all target nodes for the configured `ansible_user`.
- Git.
- `ansible-lint`.
- `yamllint`.
- `pre-commit`.
- `ansible-vault` for encrypted secret management.

## Project Structure
- `site.yml`
- `cluster_nodes.yml`
- `hermes_node.yml`
- `inventories/dev/inventory.yml`
- `roles/`

## Role Breakdown
- `preflight`: validates host mapping and architecture.
- `common`: installs baseline OS packages and locale settings.
- `ntp`: configures chrony time sync.
- `motd`: installs the J.A.R.V.I.S. MOTD.
- `docker_engine`: installs Docker and hardens the daemon config.
- `llama_cluster`: deploys the `llama.cpp` RPC stack.
- `hermes_agent`: installs and runs Hermes Agent.
- `wifi`: enables or disables Wi-Fi.
- `performance`: applies CPU, sysctl, and file limit tuning.
- `hardware_tuning`: applies Raspberry Pi hardware tuning.

## Usage
1. Install the required collection.

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

2. Validate syntax.

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml --syntax-check
```

3. Check reachability.

```bash
ansible -i inventories/dev/inventory.yml all -m ping
```

4. Deploy the full stack.

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml
```

5. Deploy the Hermes node only.

```bash
ansible-playbook -i inventories/dev/inventory.yml hermes_node.yml
```

6. Deploy the cluster nodes only.

```bash
ansible-playbook -i inventories/dev/inventory.yml cluster_nodes.yml
```

7. Adjust environment variables in `inventories/dev/inventory.yml`.

- Shared defaults live in `roles/llama_cluster/defaults/main.yml`.
- Avoid duplicating variable values unless the environment requires it.
- Set `llama_enforce_image_digests: true` if you want digest pinning.
- Store Hermes Telegram secrets in `inventories/dev/group_vars/hermes_agent_nodes/vault.yml` and encrypt that file with `ansible-vault`.

## Web UI

This section covers the Hermes web dashboard and the Open WebUI integration points.

### Hermes Web Dashboard

- `hermes_web_dashboard`: deploys the Hermes Web Dashboard through Docker Compose.
- Docker Engine and the Compose v2 plugin are required for the dashboard container.
- The dashboard runs as a Docker Compose service, binds to `0.0.0.0:9119` by default, and shares the Hermes home volume with the Hermes Agent.

### Copy Dashboard Links

```text
http://<host-ip>:9119/
http://<host-ip>:9119/login
http://<host-ip>:9119/api/status
```

### Deployment

```bash
ansible-playbook -i inventories/dev/inventory.yml hermes_node.yml
```

### Open WebUI

Open WebUI connects to the Hermes gateway API, not directly to the dashboard.

```text
http://<host-ip>:8642/v1
```

- Open WebUI setup: https://docs.openwebui.com/
- Hermes gateway endpoint: use the Hermes host IP and port `8642`
- If you run Open WebUI separately, point its `OPENAI_API_BASE_URL` at the Hermes gateway endpoint above.

## Important

- Hermes Agent was intentionally installed locally on the host rather than in a Docker container.
- The Hermes Agent service runs with root privileges on the target machine.

## Official Documentation References
- Ansible playbooks: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html
- Ansible inventory: https://docs.ansible.com/ansible/latest/inventory_guide/index.html
- Ansible Vault: https://docs.ansible.com/ansible/latest/vault_guide/index.html
- `ansible.builtin.assert`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/assert_module.html
- `ansible.builtin.apt`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
- `ansible.builtin.command`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html
- `ansible.builtin.copy`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
- `ansible.builtin.lineinfile`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html
- `ansible.builtin.service`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
- `ansible.builtin.set_fact`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/set_fact_module.html
- `ansible.builtin.template`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html
- `ansible.builtin.pip`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html
- `ansible.builtin.systemd`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html
- `community.docker.docker_compose_v2`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_v2_module.html
- `community.docker.docker_image`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html

## Notes
- Any `config.txt` change requires a reboot.
- Raspberry Pi 5 PCIe Gen 3 may be unstable under some thermal and power conditions.
- Use a 5A USB-C supply for best USB and PCIe stability.

Author: Andrii Sadovskyi <andrii.sadovskyi@gmail.com>
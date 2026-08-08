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
- `community.docker.docker_compose_v2`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_v2_module.html
- `community.docker.docker_image`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html

## Notes
- Any `config.txt` change requires a reboot.
- Raspberry Pi 5 PCIe Gen 3 may be unstable under some thermal and power conditions.
- Use a 5A USB-C supply for best USB and PCIe stability.

Author: Andrii Sadovskyi <andrii.sadovskyi@gmail.com>
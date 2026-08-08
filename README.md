# Raspberry Pi 5 AI Cluster Automation with Ansible

## Project Name
Raspberry Pi 5 AI Cluster Automation with Ansible.

## Description
Ansible automation for a Raspberry Pi 5 cluster with a `llama.cpp` RPC stack and a Hermes Agent node.

## Prerequisites
- Ansible 2.10 or later.
- Python 3 with `pip`.
- SSH access to all managed hosts.
- Git.
- `ansible-lint`.
- `yamllint`.
- `pre-commit`.
- `ansible-vault`.

## Usage
1. Install the required collection.

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

2. Validate syntax.

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml --syntax-check
```

3. Check connectivity.

```bash
ansible -i inventories/dev/inventory.yml all -m ping
```

4. Deploy everything.

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml
```

5. Deploy only the cluster.

```bash
ansible-playbook -i inventories/dev/inventory.yml cluster_nodes.yml
```

6. Deploy only the Hermes node.

```bash
ansible-playbook -i inventories/dev/inventory.yml hermes_node.yml
```

## Notes
- Shared defaults live in `roles/llama_cluster/defaults/main.yml`.
- Keep secrets in `inventories/dev/group_vars/hermes_agent_nodes/vault.yml` and encrypt that file with `ansible-vault`.
- The Hermes Agent runs locally on the host with root privileges.
- Hermes dashboard: `http://<host-ip>:9119/`.
- Open WebUI is available at `http://<host-ip>:3000/`.

## Official Documentation References
- Ansible playbooks: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html
- Ansible inventory: https://docs.ansible.com/ansible/latest/inventory_guide/index.html
- Ansible Vault: https://docs.ansible.com/ansible/latest/vault_guide/index.html
- `ansible.builtin.assert`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/assert_module.html
- `ansible.builtin.apt`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
- `ansible.builtin.command`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html
- `ansible.builtin.copy`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
- `ansible.builtin.file`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
- `ansible.builtin.lineinfile`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html
- `ansible.builtin.pip`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html
- `ansible.builtin.service`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
- `ansible.builtin.set_fact`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/set_fact_module.html
- `ansible.builtin.stat`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html
- `ansible.builtin.systemd`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html
- `ansible.builtin.template`: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html
- Open WebUI: https://docs.openwebui.com/
- `community.docker.docker_compose_v2`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_v2_module.html
- `community.docker.docker_image`: https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html

Author: Andrii Sadovskyi <andrii.sadovskyi@gmail.com>

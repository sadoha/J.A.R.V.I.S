# Raspberry Pi 5 AI Cluster Automation with Ansible

## Description:
  * Ansible-based deployment for a Raspberry Pi 5 AI cluster.
  * Includes llama.cpp RPC cluster with Docker and Hermes Agent.
  * Provides inventory-driven role modules for system tuning.

This repository provides an Ansible-based deployment for:

- 1 Raspberry Pi 5 llama-server-head node and 2 Raspberry Pi 5 llama-rpc-worker nodes running a local llama.cpp RPC cluster with Docker.
- Inventory-driven role modules for Wi-Fi, performance, and hardware tuning.

## Prerequisites

-   **Ansible**: Version 2.10 or higher.
-   **Python 3**: With `pip` for Ansible dependencies.
-   **SSH access**: To all Raspberry Pi nodes with the specified `ansible_user`.
-   **Git**: For cloning the repository.
-   **ansible-lint**: Recommended for linting Ansible playbooks.
-   **yamllint**: Recommended for linting YAML files.
-   **pre-commit**: Recommended for managing pre-commit hooks for code quality.

## Usage

### 1. Install required Ansible collections

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

### 2. Validate syntax

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml --syntax-check
```

### 3. Validate host reachability

```bash
ansible -i inventories/dev/inventory.yml all -m ping
```

## Project Structure

This repository follows the official Ansible sample setup pattern, using role-based organization.

- site.yml
- cluster_nodes.yml
- inventories/dev/inventory.yml
- roles/*

## Role Breakdown

- preflight: validates host count, IP mapping, architecture, and key safety assumptions.
- common: baseline operating system and unattended security updates.
- ntp: configures chrony so all Raspberry Pi hosts use the same NTP sources and remain synchronized.
- motd: installs a J.A.R.V.I.S. banner and host configuration summary in /etc/motd.
- docker_engine: official Docker apt repository setup and hardened daemon logging.
- llama_cluster: deploys llama-server-head on the head node and a single llama-rpc-worker on each worker node.
- wifi: explicit Wi-Fi enable or disable through inventory variable.
- performance: CPU governor, sysctl, and file descriptor tuning.
- hardware_tuning: PCIe, PCIe Gen 3, PoE fan probe, and active-cooling fan thresholds.

## Security Controls Included

- Preflight assertions for host mapping and architecture.
- Docker daemon log limits to avoid uncontrolled log growth.
- Container hardening in Compose: no-new-privileges, dropped Linux capabilities, read-only root filesystem, and tmpfs for ephemeral writes.
- Loopback service binding by default for llama head API (`127.0.0.1`).
- Optional strict container digest policy for llama and Open WebUI images.

## Configure Variables

Edit environment values in inventories/dev/inventory.yml.

Variable ownership convention:
- Keep shared defaults in roles/llama_cluster/defaults/main.yml.
- Override only deployment-specific values in inventories/dev/inventory.yml, such as the cluster role, host endpoints, and any host-specific tuning.
- Avoid duplicating the same value in both places unless the inventory needs to intentionally override the default for a specific environment.

Important variables to set before first deployment:

- ansible_user
- llama_model_path

If your container images are pinned by digest, set:

- llama_enforce_image_digests: true

### 4. Deploy full stack

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml
```

5. Deploy only cluster nodes:

```bash
ansible-playbook -i inventories/dev/inventory.yml cluster_nodes.yml
```

6. Deploy only Hermes node:

```bash
ansible-playbook -i inventories/dev/inventory.yml hermes_node.yml
```

## Open WebUI on the head node

When the head-node deployment is complete, Open WebUI is available at http://127.0.0.1:3000/ by default. The service uses the official Open WebUI container image from https://docs.openwebui.com/ and connects to the local llama.cpp head API through the Compose network defined in the deployment.

Official references:
- https://docs.openwebui.com/
- https://docs.docker.com/compose/compose-file/

## Testing llama-rpc-worker and llama-server-head with curl

Use the following checks after deployment to verify that each component is alive and that the head can be reached as an HTTP API endpoint.

### 1. Verify the head server is healthy

Run this on the head node or from any machine that can reach the head host:

```bash
curl -i http://127.0.0.1:8080/health
```

What this does:
- Sends an HTTP GET request to the head service.
- Confirms that the container is up and listening on the expected port.
- A successful response should return HTTP 200.

If you are testing from another machine, replace `127.0.0.1` with the head node IP from your inventory, for example `192.168.50.201`.

### 2. Verify the RPC worker is listening on its port

Run this on each worker node, or from a machine that can reach that node:

```bash
curl -i http://127.0.0.1:50052/
```

What this does:
- Attempts to open a connection to the RPC worker port.
- Confirms that the worker service is listening and reachable over the network.
- The response may be an empty reply or a connection-close depending on the binary implementation, but a successful TCP connection is the important signal here.

If the worker is not reachable, you will see a connection error and should inspect the container logs.

### 3. Verify the head can serve a real chat request

This is the most important end-to-end smoke test. It exercises the head service as an API endpoint and confirms that the stack is actually usable for inference:

```bash
curl -i -X POST http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "messages": [
      {"role": "user", "content": "Say hello in one short sentence."}
    ],
    "temperature": 0.2,
    "max_tokens": 16
  }'
```

What this does:
- Sends a minimal chat-completions request to the head server.
- Verifies that the HTTP API is working and that the model stack is ready to generate output.
- A successful response should contain a JSON payload with a completion message.

If you are testing from another host, replace `127.0.0.1` with the head node IP.

### 4. Inspect logs if a test fails

If either the health check or the chat completion request fails, inspect the relevant container logs:

```bash
docker compose -f /opt/llama-cluster/docker-compose.yml ps
docker compose -f /opt/llama-cluster/docker-compose.yml logs llama-server-head
docker compose -f /opt/llama-cluster/docker-compose.yml logs llama-rpc-worker
```

These commands help you distinguish between:
- the head service not starting,
- the worker service not binding to its port,
- or the head failing to connect to the worker endpoints.

## Useful Tags

Run only selected modules with tags:

```bash
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags preflight
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags docker,llama
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags hermes
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags ntp,time
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags motd,branding
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags wifi
ansible-playbook -i inventories/dev/inventory.yml site.yml --tags performance,hardware
```

## Notes

- Any config.txt changes require a reboot to become active.
- Raspberry Pi documentation warns that PCIe Gen 3 on Raspberry Pi 5 may be unstable. Keep this enabled only after validation under your thermal and power conditions.
- For best USB and PCIe stability on Raspberry Pi 5, use a 5A USB-C power supply.
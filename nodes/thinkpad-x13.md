# ThinkPad X13 Gen 2 — Worker Node Build Doc

**Role:** k3s agent + CPU inference worker
**Hardware:** i5-1145G7, 16 GB RAM, Intel Iris Xe (CPU inference — no discrete GPU)
**OS:** Ubuntu Server 26.04 LTS, 64-bit ("Intel or AMD 64-bit architecture" build), headless

## 1. Flash the installer

- Download: Ubuntu Server 26.04.1 LTS, 64-bit from ubuntu.com/download/server
- Flash with Popsicle on the rig. Boot menu is **F12** on ThinkPads; **Enter** at
  power-on then F1 gets you into BIOS setup if you need it.

## 2. Install Ubuntu Server

- Language/keyboard: defaults.
- Network: DHCP with a router-side reservation — stable address
  (suggested hostname: `thinkpad-worker`).
- Storage: "Use an entire disk".
- Profile: username of your choice, strong password.
- **SSH: check "Install OpenSSH server".**
- No extra snaps needed.

## 3. First boot

```bash
sudo apt update && sudo apt upgrade -y
```

No NVIDIA drivers needed on this node — Iris Xe is fine for display-out on the
rare occasion you plug in a monitor, and Ollama will run CPU-only here.

## 4. Join the mesh network

Join your NordVPN Meshnet and note the mesh IP:

- Mesh IP: _______________

## 5. Install the k3s agent

```bash
K3S_URL=https://<CONTROL_PLANE_MESH_IP>:6443 K3S_TOKEN=<NODE_TOKEN> ./k3s/install-agent.sh
```

Get `<NODE_TOKEN>` from the rig: `sudo cat /var/lib/rancher/k3s/server/node-token`

## 6. Verify

From the rig: `kubectl get nodes` — `thinkpad-worker` should appear as Ready.

## 7. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

CPU-only inference. 16 GB RAM comfortably fits 7–8B Q4 models
(see `ollama/models.md`). Slower than the GPU nodes, but fine for background and
overflow workloads.

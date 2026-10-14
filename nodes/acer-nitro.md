# Acer Nitro AN515-55 — Worker Node Build Doc

**Role:** k3s agent + GPU inference worker
**Hardware:** i5-10300H, 8 GB RAM, NVIDIA RTX 3050 Laptop GPU 4 GB
(confirmed via Device Manager, Oct 14, 2026 — earlier notes wrongly said GTX 1650)
**OS:** Ubuntu Server 26.04 LTS, 64-bit ("Intel or AMD 64-bit architecture" build), headless

## 1. Flash the installer

- Download: Ubuntu Server 26.04.1 LTS, 64-bit from ubuntu.com/download/server
  (the main download button — "Intel or AMD 64-bit architecture" is the AMD64 build)
- Flash with Popsicle on the rig. Boot the Acer, hammer **F2** for BIOS —
  confirm UEFI boot mode, disable Secure Boot, then **F12** for the boot menu
  to pick the USB.

## 2. Install Ubuntu Server

- Language/keyboard: defaults are fine.
- Network: DHCP is fine if your router does DHCP reservations — give this node a
  stable address (suggested hostname: `acer-worker`).
- Storage: "Use an entire disk". LVM optional — skip it, keep it simple.
- Profile: username of your choice, strong password.
- **SSH: check "Install OpenSSH server".** This is how you'll reach the node
  headless from the rig afterward. Don't import SSH keys from GitHub at install
  time — add your key manually later.
- No extra snaps needed.

## 3. First boot

```bash
sudo apt update && sudo apt upgrade -y
```

## 4. NVIDIA drivers (Acer only — the important part)

```bash
ubuntu-drivers devices        # see what's recommended
sudo ubuntu-drivers autoinstall
sudo reboot
nvidia-smi                    # should show the GPU, driver version, VRAM
```

**Secure Boot gotcha:** if Secure Boot is enabled in BIOS, the driver install
will ask you to set a MOK (Machine Owner Key) password. On the next reboot a
blue screen prompts you to "Enroll MOK" — select it, enter the password you set,
reboot. Alternatively, disable Secure Boot in BIOS before installing drivers and
skip this entirely.

Record the exact GPU model and VRAM from `nvidia-smi` output here:

- GPU model: _______________
- VRAM: _______________
- Driver version: _______________

## 5. Join the mesh network

Join your NordVPN Meshnet (per the network plan) and note the mesh IP:

- Mesh IP: _______________

## 6. Install the k3s agent

From the repo root on this machine:

```bash
K3S_URL=https://<CONTROL_PLANE_MESH_IP>:6443 K3S_TOKEN=<NODE_TOKEN> ./k3s/install-agent.sh
```

Get `<NODE_TOKEN>` from the rig: `sudo cat /var/lib/rancher/k3s/server/node-token`

## 7. Verify

From the rig: `kubectl get nodes` — `acer-worker` should appear as Ready.

## 8. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ollama picks up the NVIDIA GPU automatically once `nvidia-smi` works — no
container toolkit needed for host-level installs. Model assignment per
`ollama/models.md` (sized for the 4 GB VRAM — see that file).

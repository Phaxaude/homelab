# Acer Nitro AN515-55 — Worker Node Build Doc

**Role:** k3s agent + GPU inference worker
**Hardware:** i5-10300H, 8 GB RAM, NVIDIA RTX 3050 Laptop GPU 4 GB
(confirmed via Device Manager, Oct 14, 2026 — earlier notes wrongly said GTX 1650)
**OS:** Ubuntu Server 26.04 LTS, 64-bit ("Intel or AMD 64-bit architecture" build), headless

## 1. BIOS prep (Acer quirks — read this first)

- Boot the Acer, hammer **F2** for BIOS. Note: a supervisor password may be set
  on the machine — you'll need it to change any settings.
- **SATA Mode is hidden by default.** On the **Main** tab, press **Ctrl+S** to
  reveal it, then change it from RAID/Optane to **AHCI**. Without this, the
  Ubuntu installer cannot see the NVMe drive at all
  ("block probing did not discover any disks").
- **Fast Boot:** disable it — it makes USB booting flaky.
- Leave Intel VTX/VTD enabled (good for virtualization later).
- **Boot tab:** confirm UEFI boot mode, Secure Boot disabled. The
  Popsicle-flashed USB shows up as "Linpus lite" — that's normal.
- **F12 boot menu:** enabled, so you can pick the USB at boot with **F12**.

## 2. Install Ubuntu Server

- Language/keyboard: defaults are fine.
- Network: DHCP is fine if your router does DHCP reservations — give this node a
  stable address (hostname: `acer-nitro`).
- Storage: "Use an entire disk". LVM is fine — leave "Encrypt the LVM group
  with LUKS" unchecked. A headless cluster node shouldn't need a passphrase
  typed in at every reboot.
- Profile: hostname `acer-nitro`, username `phaxaude`, strong password.
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
sudo ubuntu-drivers install   # (autoinstall was retired — install is the replacement)
sudo reboot
nvidia-smi                    # should show the GPU, driver version, VRAM
```

**Secure Boot gotcha:** if Secure Boot is enabled in BIOS, the driver install
will ask you to set a MOK (Machine Owner Key) password. On the next reboot a
blue screen prompts you to "Enroll MOK" — select it, enter the password you set,
reboot. Alternatively, disable Secure Boot in BIOS before installing drivers and
skip this entirely.

Record the exact GPU model and VRAM from `nvidia-smi` output here
(verified Oct 14, 2026):

- GPU model: NVIDIA GeForce RTX 3050 (laptop)
- VRAM: 4 GB (4096 MiB)
- Driver version: 595.99.02
- CUDA version: 13.2

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

From the rig: `kubectl get nodes` — `acer-nitro` should appear as Ready.

## 8. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Ollama picks up the NVIDIA GPU automatically once `nvidia-smi` works — no
container toolkit needed for host-level installs. Model assignment per
`ollama/models.md` (sized for the 4 GB VRAM — see that file).

# ThinkPad X13 Gen 2 — Worker Node Build Doc

**Role:** k3s agent + CPU worker node
**Hardware:** i5-1145G7, 16 GB RAM, Intel Iris Xe (integrated — no discrete GPU,
  no dedicated VRAM; shares system RAM), 512 GB Toshiba NVMe (KXG6AZNV512G)
**OS:** Ubuntu Server 26.04.1 LTS, 64-bit ("Intel or AMD 64-bit architecture"
  build), headless
**Status (2026-10-14):** installed, joined to the cluster as `lenovo-thinkpad`

## 1. Flash the installer

- Download: Ubuntu Server 26.04.1 LTS, 64-bit from ubuntu.com/download/server
- Flash with Popsicle on the rig. Boot menu is **F12** on ThinkPads; **Enter** at
  power-on then F1 gets you into BIOS setup if you need it.
- This machine previously ran Kali Linux (wiped). The boot menu showed a
  leftover Windows Boot Manager entry from an older install — ignored, the whole
  disk was wiped.

## 2. Install Ubuntu Server

- Language/keyboard: defaults.
- Network: installed over WiFi (Intel AX201). See networking notes below.
- Storage: "Use an entire disk" (512 GB Toshiba NVMe), LVM on, no LUKS
  (headless node — no passphrase at boot).
- Profile: hostname `lenovo-thinkpad`, username `phaxaude`, strong password.
- **SSH: check "Install OpenSSH server".**
- No extra snaps needed.

### Networking notes

- The X13 has **no built-in RJ45 port**. There is a Lenovo proprietary
  **ethernet extension port** on the left side (Intel I219-V onboard) — it needs
  Lenovo's proprietary dongle, which we don't have, so it's unused.
- Tried a j5create USB-C dock's ethernet (ASIX AX88179): the chip enumerated
  in the installer but DHCP never completed (link lights were on at both ends).
  Parked for later troubleshooting from the installed OS — possibly a USB-C
  power-delivery quirk with the dock.
- A Plugable USBC-E1000 USB-C-to-Gigabit adapter is on order (arriving late
  October 2026) for a dedicated wired connection. Until then: WiFi.

## 3. First boot

```bash
sudo apt update && sudo apt upgrade -y
```

No proprietary drivers needed on this node — everything is Intel and in-kernel
(Iris Xe graphics, AX201 WiFi, chipset). Ollama runs CPU-only here.

Gotcha: the rig's `~/.ssh/known_hosts` still had this machine's old Kali host
key, so the first SSH attempt was refused. Fix:

```bash
ssh-keygen -R <thinkpad-ip>
```

then reconnect and accept the new key.

## 4. Join the mesh network

Join your NordVPN Meshnet and note the mesh IP:

- Mesh IP: _______________

## 5. Install the k3s agent

On the node (k3s v1.36.5+k3s1):

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<CONTROL_PLANE_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
```

Get `<NODE_TOKEN>` from the rig: `sudo cat /var/lib/rancher/k3s/server/node-token`.
The control plane must allow the agent through its firewall: TCP 6443 (k3s
API), UDP 8472 (flannel VXLAN), TCP 10250 (kubelet).

## 6. Verify

From the rig: `sudo kubectl get nodes` — `lenovo-thinkpad` should appear as
Ready alongside the control plane and `acer-nitro`.

## 7. Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

CPU-only inference (no discrete GPU). Small models stay snappy — e.g.
`ollama pull qwen2.5:1.5b`. The 16 GB of RAM comfortably fits 7–8B Q4 models
too, just slower (see `ollama/models.md`). Fine for background and overflow
workloads; the GPU node (acer-nitro) handles the heavy inference.

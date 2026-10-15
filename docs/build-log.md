# Build Log

Running record of the cluster build. Newest entries at the top.

## 2026-10-14 — ThinkPad X13 installed, 3-node cluster complete

- ThinkPad X13 Gen 2 wiped (was running Kali; ignored a leftover Windows Boot
  Manager entry in the F12 menu) and installed with Ubuntu Server 26.04.1 LTS:
  entire 512 GB Toshiba NVMe (KXG6AZNV512G), LVM, no LUKS, hostname
  `lenovo-thinkpad`, user `phaxaude`, OpenSSH server.
- Networking: no RJ45 on this model — only a Lenovo proprietary ethernet
  extension port (Intel I219-V onboard; dongle not purchased). Tried the
  j5create USB-C dock's ethernet (ASIX AX88179): enumerated in the installer
  but DHCP never completed despite link lights — parked for later. Installed
  over WiFi (Intel AX201). A Plugable USBC-E1000 adapter is on order for a
  dedicated wired link.
- No proprietary drivers needed — all Intel, in-kernel. No discrete GPU
  (Iris Xe shares system RAM, no dedicated VRAM).
- SSH gotcha: rig's known_hosts still had the old Kali host key —
  `ssh-keygen -R <ip>` fixed it.
- Joined k3s as agent (v1.36.5+k3s1). Installed Ollama (CPU-only),
  pulled `qwen2.5:1.5b` — snappy for short prompts.
- **Cluster is now 3 nodes, all Ready:** system76 (control-plane),
  acer-nitro (GPU worker, RTX 3050), lenovo-thinkpad (CPU worker).

## 2026-10-14 — Acer Nitro install complete, k3s agent joined

- Ubuntu Server 26.04.1 LTS flashed with Popsicle, booted via F12.
- Hit "block probing did not discover any disks" — BIOS was in RAID/Optane mode.
  SATA Mode is **hidden** on this model: press **Ctrl+S** on the Main tab to reveal
  it, then switch to AHCI. Also disabled Fast Boot (flaky USB boot), confirmed
  UEFI + Secure Boot off, VTX/VTD left enabled.
- Storage: entire 256 GB WD NVMe, LVM, no LUKS (headless node — no passphrase at boot).
- Profile: hostname `acer-nitro`, user `phaxaude`. OpenSSH server installed.
- Wired via Cat8 through the 5-way splitter; DHCP gave it 192.168.0.13, then
  reassigned to 192.168.0.14 — set a router-side DHCP reservation.
- NVIDIA drivers via `sudo ubuntu-drivers install` (`autoinstall` is retired),
  verified with `nvidia-smi`: RTX 3050, driver 595.99.02, CUDA 13.2, 4 GB VRAM.
- k3s agent join initially failed — rig's ufw (default DROP) was blocking it.
  Opened TCP 6443, UDP 8472, TCP 10250. Then `kubectl get nodes` showed
  acer-nitro Ready.
- Installed Ollama, pulled `llama3.2:3b`. First inference was slow (~30 s,
  model load + CUDA init), then instant. `nvidia-smi` confirmed the
  llama-server process at 97% GPU utilization — the 3050 is doing real work.
- Lid-close suspend disabled via a systemd-logind drop-in
  (`HandleLidSwitch=ignore`, `HandleLidSwitchExternalPower=ignore`) so the
  node keeps running headless. Fans verified working under load (stress-ng).

## 2026-10-14 — Project kicked off

- Decided: Ubuntu Server 26.04 LTS on both workers, k3s, Ollama per node.
- Repo created. Next: flash Ubuntu Server ISOs, install on Acer + ThinkPad.
- Resume project bullet will be rewritten to match reality as the build progresses —
  no claims of finished work until it's finished.

## Pinned for later

- **ProxyDHCP netboot server on the rig** (dnsmasq in proxy mode + TFTP): lets
  future nodes install over the network without touching the household router's
  DHCP — the BE15000 keeps serving the house untouched, the rig only answers
  PXE-booting machines. Do after the cluster is up.

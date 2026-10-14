# Build Log

Running record of the cluster build. Newest entries at the top.

## 2026-10-14 — Acer Nitro install in progress

- Ubuntu Server 26.04.1 LTS flashed with Popsicle, booted via F12.
- Hit "block probing did not discover any disks" — BIOS was in RAID/Optane mode.
  SATA Mode is **hidden** on this model: press **Ctrl+S** on the Main tab to reveal
  it, then switch to AHCI. Also disabled Fast Boot (flaky USB boot), confirmed
  UEFI + Secure Boot off, VTX/VTD left enabled.
- Storage: entire 256 GB WD NVMe, LVM, no LUKS (headless node — no passphrase at boot).
- Profile: hostname `acer-nitro`, user `phaxaude`. OpenSSH server installed.
- Wired via Cat8 through the 5-way splitter, DHCP gave it 192.168.0.13.
- Next: finish install, `apt update && upgrade`, NVIDIA drivers via
  `ubuntu-drivers autoinstall`, verify with `nvidia-smi`, then k3s agent join.

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

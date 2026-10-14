# Build Log

Running record of the cluster build. Newest entries at the top.

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

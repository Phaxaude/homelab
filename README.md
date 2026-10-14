# Homelab k3s Cluster

Jacob's home Kubernetes lab: a k3s cluster doing distributed local LLM inference
across heterogeneous hardware, and the scheduling target for his in-development
agent framework.

## Hardware

| Role | Machine | CPU | RAM | GPU | OS |
|------|---------|-----|-----|-----|----|
| Control plane | System76 rig (self-built) | i7-14700K (20c/28t) | 128 GB | RTX 3060 + Intel UHD 770 | Pop!_OS 24.04 |
| Worker | Acer Nitro AN515-55 | i5-10300H | 8 GB | RTX 3050 Laptop 4 GB | Ubuntu Server 26.04 LTS |
| Worker | ThinkPad X13 Gen 2 | i5-1145G7 | 16 GB | Iris Xe | Ubuntu Server 26.04 LTS |

The ASUS Zenbook 14 (school laptop, must stay dual-boot Windows) is intentionally
excluded — a cluster node that dual-boots is a headache nobody wants.

Network: 5 Gbps fiber into the rig. Node interconnect via NordVPN Meshnet.

## Layout

- `nodes/` — per-node build docs: OS install, drivers, k3s agent join, verification
- `k3s/` — cluster install scripts and config (server on rig, agents on workers)
- `ollama/` — per-node Ollama deployment, model placement sized to each node's VRAM/RAM
- `docs/` — decisions and build log

## Decisions

- **k3s** over full Kubernetes: single binary, tiny RAM footprint (matters on the
  Acer's 8 GB), perfect for homelab scale.
- **Ubuntu Server 26.04 LTS** on workers: zero learning curve (daily-driver is
  Pop!_OS), smoothest NVIDIA story for the Acer's RTX 3050, SSH access for
  learning and debugging. Talos Linux deferred to a possible phase two — its
  NVIDIA-on-worker complexity isn't worth it while the control plane is a mutable
  desktop anyway. (Full Talos vs Ubuntu analysis in `docs/os-decision.md`.)
- **Ollama per node**, models sized to each machine: small GPU-accelerated models
  on the Acer's 4 GB 3050, larger CPU/RAM-backed models on the rig and ThinkPad.

## Status

- [ ] Ubuntu Server 26.04 on Acer + ThinkPad
- [ ] NVIDIA drivers on Acer, verify `nvidia-smi`
- [ ] k3s server on rig
- [ ] k3s agents joined (Acer, ThinkPad)
- [ ] Ollama deployed per node
- [ ] First distributed inference test

## Build log

See `docs/build-log.md`.

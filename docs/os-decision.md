# OS Decision Record — Worker Nodes

**Decided:** 2026-10-14 — Ubuntu Server 24.04 LTS (headless) on both workers.
**Deferred:** Talos Linux as a possible phase two, once the stack is understood.

## Why Ubuntu Server won for phase one

- **Zero learning curve.** Daily driver is Pop!_OS — every command, config path, and
  troubleshooting instinct transfers directly.
- **k3s's home turf.** The vast majority of k3s guides, GitHub issues, and
  "I hit this exact error" threads assume Ubuntu. Biggest possible answer pool
  for 2am debugging.
- **NVIDIA is painless.** `ubuntu-drivers autoinstall` + NVIDIA container toolkit
  via apt. This is the decisive one: the Acer's GPU is the only worker GPU, and
  GPU-accelerated Ollama there is core to the plan. On Ubuntu this is a solved
  problem with years of documentation.
- **It's a real OS.** SSH in, run htop, poke at logs, test half-built agent
  framework components directly on a node without containerizing everything
  first. That flexibility matters while iterating.
- **LTS until 2029.** Install once, forget about it.

## Why Talos lost for phase one (but stays on the table)

- **NVIDIA is the catch.** Talos supports NVIDIA via system extensions, but it's
  tied to Talos releases, less documented, and fiddlier — landing on exactly the
  wrong node (the Acer, where the GPU matters for Ollama).
- **No shell to hide in.** Debugging goes through the Talos API. Elegant once you
  know Kubernetes internals cold; a genuine handicap while still learning.
- **Smaller community.** Fewer exact-error/exact-fix threads than Ubuntu+k3s.
- **The mixed environment kills the purity case.** Talos's big win is fully
  declarative infrastructure, but the control plane is a hand-administered
  Pop!_OS desktop — immutable workers bolted to a mutable control plane means
  paying the full learning curve without the full payoff.
- **The agent framework.** Anything needing to run on a node outside a container
  during development becomes painful or impossible on Talos.

## Phase-two condition

If the cluster becomes beloved and the goal shifts to full GitOps-declarative
infrastructure, reinstalling two worker nodes on Talos takes an afternoon —
revisit then, with real k3s experience in hand.

## Related decisions

- **k3s** over full Kubernetes/k0s: single binary, tiny RAM footprint (the Acer's
  8 GB will thank us).

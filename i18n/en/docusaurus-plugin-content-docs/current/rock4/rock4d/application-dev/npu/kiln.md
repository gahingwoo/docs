---
sidebar_position: 25

doc_kind: page
source_of_truth: local
title: "Kiln: NPU (LLM + vision) on a mainline kernel"
description: "Kiln runs the vendor RKLLM/RKNN NPU stack on a mainline Linux kernel on the ROCK 4D: one command on Armbian for a local LLM chat, image recognition, and an OpenAI-compatible API."
locale: en
board: rock4d
task_type: install
last_verified: 2026-07-11
---

# Kiln: NPU (LLM + vision) on a mainline kernel

[Kiln](https://github.com/gahingwoo/kiln) is an open-source community project that runs
the vendor RKLLM / RKNN NPU stack on the ROCK 4D on a **mainline Linux kernel** (instead
of the vendor 6.1 BSP). It builds the vendor GPL `rknpu` driver out-of-tree, drives it
with the closed `librkllmrt` (LLM) and `librknnrt` (vision) runtimes, and adds a small
set of RK3576 NPU kernel patches (clock / power-domain / two-IOMMU) so the NPU works on
`linux-7.1.3`.

Once installed you get:

- **`kiln-chat`** — chat with a local LLM on the NPU;
- **`kiln-vision`** — image classification / YOLO object detection;
- **`kiln-serve`** — an OpenAI-compatible HTTP API you can point Open WebUI, LangChain, or any OpenAI client at;
- `kiln` (a menu), `kiln-config` (settings), `kiln-convert` (on-board model conversion), `kiln-doctor` (health check).

:::note
Kiln is an independent third-party open-source project, not part of official Radxa
support. Report issues on its [repository](https://github.com/gahingwoo/kiln).
:::

## Prerequisites

- A **ROCK 4D** (RK3576) running **Armbian**.
- A working network (the first install downloads the kernel and runtimes; Ethernet is most reliable).
- You supply the models (Kiln ships none): drop a `*-rk3576-w4a16.rkllm` in `/opt/models`
  for the LLM; for vision you can convert one on the board with `kiln-convert`.

## Steps

Install with one command on Armbian:

```bash
curl -fsSL https://raw.githubusercontent.com/gahingwoo/kiln/main/scripts/kiln-install.sh | bash
```

It is hands-off: it pre-downloads what it needs, installs the Kiln mainline kernel, then
**reboots itself twice (~10–15 minutes total)** to finish setup and land in a ready
system. This is normal — **don't cut power**. Onboard Wi-Fi is briefly down between the
reboots (phase 2 finishes offline and doesn't need it). You'll see a welcome message at
the next login.

To drive the reboots yourself, use manual mode:

```bash
KILN_MANUAL=1 bash kiln-install.sh
```

## Expected Output

After logging in, the health check should be largely green:

```bash
kiln-doctor
```

The key items should read `[ OK ]`: `rknpu` loaded, `/dev/dri/renderD128` present, all four
MMU banks `st=0x19/0x19/0x19/0x19`, and runtimes `librkllmrt` 1.2.0 / `librknnrt` 2.3.0.

## Validation

Vision (build a classifier on the board with `kiln-convert`, then run it):

```bash
kiln-convert mobilenet --set-active
kiln-vision /opt/models/test.jpg
```

LLM (after placing a `.rkllm` in `/opt/models`):

```bash
kiln-chat
```

OpenAI-compatible API + Open WebUI:

```bash
kiln-serve
# from another machine:
curl http://<board-ip>:8080/v1/models
```

`kiln-serve` prints a ready-to-copy connection string (with the board's IP filled in) on
startup — paste it straight into Open WebUI.

## Troubleshooting

- Run `kiln-doctor` first and paste its full output into an issue.
- For common errors (version-lock, YOLO export, Wi-Fi, etc.) see Kiln's
  [troubleshooting guide](https://github.com/gahingwoo/kiln/blob/main/docs/TROUBLESHOOTING.md).
- More detail is in Kiln's [README](https://github.com/gahingwoo/kiln/blob/main/README.md).

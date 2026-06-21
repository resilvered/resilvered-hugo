---
title: "The Debian 13 ComfyUI Gauntlet: How I Got Stable Diffusion Running Without Breaking My System"
date: 2026-06-21T00:00:00+00:00
draft: false
categories: [blog]
tags: [ai, comfyui, debian, stable-diffusion, linux, python, cuda, pytorch]
description: "What happens when your distro ships Python 3.13, your GPU driver is a year old, and modern AI frameworks want the bleeding edge."
---

**Or: what happens when your distro ships Python 3.13, your GPU driver is a year old, and modern AI frameworks want the bleeding edge.**

---

Debian 13 "Trixie" is out. It's fresh, it's stable, and it comes with Python 3.13 as the default. That's great — unless you want to run ComfyUI, the node-based Stable Diffusion workflow tool that's taken the AI art world by storm. Then it's a puzzle box of version conflicts that'll have you questioning your life choices.

I just went through this on a machine with an RTX 3060, and I'm writing this so you don't have to burn an afternoon deciphering CUDA compatibility matrices. Here's the gauntlet, and how to clear it.

## The stack of doom

Here's what Debian 13 gives you out of the box:

- **Python 3.13** — brand new, shiny, and missing wheels for quite a few packages
- **NVIDIA driver 550.163.01** — from the standard repos, supports up to CUDA 12.4
- **No official NVIDIA CUDA repo support for Debian 13 yet**

Here's what ComfyUI (mid-2026) expects:

- **PyTorch 2.9+** — which wants CUDA 12.8 or 13.0
- **CUDA 12.8** — which needs NVIDIA driver 570.65+
- **CUDA 13.0** — which needs driver 580.65+

See the problem? Debian gives you CUDA 12.4. ComfyUI wants CUDA 12.8. There's a gap, and bridging it with third-party driver repos is how you end up with a system that breaks on the next `apt upgrade`.

## The strategy: meet in the middle

The trick is CUDA's **forward compatibility**. A newer PyTorch compiled against CUDA 12.8 will happily run on a system with CUDA 12.4, as long as the driver supports the older CUDA version. This is documented, it's supported, and it's the key to staying within Debian's stable ecosystem.

The plan:

1. **Use Python 3.12 instead of 3.13** — better ecosystem support, fewer missing wheels
2. **Install PyTorch 2.5.1+cu124 first** — built against CUDA 12.4, matches your driver
3. **Let ComfyUI's requirements.txt upgrade PyTorch to 2.9.1+cu128** — forward-compatible with your CUDA 12.4 system
4. **Enable nvidia-persistenced** — fixes CUDA initialization errors on Linux

## Step by step

### 1. Install UV (the fast Python package manager)

UV is a Rust-based drop-in for pip/venv/pip-tools. It resolves dependencies in seconds instead of minutes and handles Python version management. It's become the standard tool for Python projects in 2025-2026.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
```

Add this to your shell config to make it permanent:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

### 2. Install Python 3.12 with UV

```bash
uv python install 3.12
```

This pulls down CPython 3.12.12 (or latest 3.12.x) without touching your system Python. Verify:

```bash
python3.12 --version   # Python 3.12.12
```

### 3. Create the ComfyUI virtual environment

```bash
cd ~/Source/ComfyUI       # clone ComfyUI first if you haven't
rm -rf .venv               # nuke any existing venv
uv venv --python 3.12 .venv --prompt "Comfy UI"
source .venv/bin/activate
```

**Note:** UV's venvs are minimal — pip isn't included by default. Bootstrap it:

```bash
python -m ensurepip --upgrade
```

UV only creates `pip3` and `pip3.12` symlinks, not `pip`. Use `pip3` or `python -m pip` throughout.

### 4. Install PyTorch 2.5.1 with CUDA 12.4

This is the anchor install. PyTorch 2.5.1+cu124 matches your driver's CUDA 12.4 capability.

```bash
pip3 install --upgrade pip
pip3 install torch==2.5.1+cu124 torchaudio==2.5.1 \
    --extra-index-url https://download.pytorch.org/whl/cu124
```

This pulls about 900MB of packages, mostly NVIDIA CUDA libraries. Grab a coffee.

Verify:

```python
import torch
print(torch.__version__)     # 2.5.1+cu124
print(torch.version.cuda)    # 12.4
```

### 5. Install ComfyUI dependencies (the upgrade happens here)

```bash
pip3 install -r requirements.txt
```

This is where the magic happens. `requirements.txt` pulls in PyTorch 2.9.1+cu128, which **replaces** the 2.5.1 you just installed. This is intentional. PyTorch 2.9.1 compiled against CUDA 12.8 will run on your CUDA 12.4 system thanks to forward compatibility.

You'll also get `torchvision 0.24.1`, `transformers 4.57.3`, `safetensors 0.7.0`, `kornia 0.8.2`, and about 50 other packages.

### 6. Fix the CUDA initialization issue

If you try running ComfyUI now, you'll likely see:

```
RuntimeError: CUDA unknown error - this may be due to an incorrectly set up environment
```

This is a known Linux quirk. The fix:

```bash
sudo systemctl enable nvidia-persistenced
sudo systemctl start nvidia-persistenced
sudo reboot           # this step is NOT optional
```

`nvidia-persistenced` keeps the NVIDIA kernel module loaded even when no processes are using it. Without it, CUDA initialization at Python startup can fail silently. You **must reboot** — the fix doesn't take effect until the kernel module reloads.

### 7. Run ComfyUI

After rebooting:

```bash
cd ~/Source/ComfyUI
source .venv/bin/activate
python main.py --enable-manager
```

If everything worked, you'll see:

```
Total VRAM 7966 MB, total RAM 63974 MB
pytorch version: 2.9.1+cu128
Device: cuda:0 NVIDIA GeForce RTX 3060 : cudaMallocAsync
```

ComfyUI is live at `http://127.0.0.1:8188`.

## Why this works

The whole approach hangs on two insights:

**CUDA forward compatibility is real and documented.** NVIDIA guarantees that applications built against a newer CUDA toolkit run on systems with an older CUDA driver, within the same major version. PyTorch 2.9.1 (CUDA 12.8) on a CUDA 12.4 system with driver 550 — that's within the CUDA 12.x family, so forward compatibility applies.

**You don't need to leave Debian's repos.** No `cuda-repo-debian13.deb`, no manual driver installs, no third-party PPAs. Everything comes from Debian's own `non-free` repository. Your system stays update-safe.

## What you get

With an RTX 3060 (8GB VRAM) and this setup:

- Async weight offloading with 2 streams
- ~60GB of pinned system memory for overflow
- Normal VRAM mode (not low-vram, not high-vram)
- Full GPU acceleration — no CPU fallback, no OOM at 512×512

## Troubleshooting quick reference

| Problem | Likely fix |
|---------|-----------|
| `pip: command not found` | Use `pip3` or `python -m pip` |
| PyTorch 2.9 install fails | Don't install it directly. Install 2.5.1 first, then let `requirements.txt` upgrade |
| `torch.cuda.is_available()` returns `False` | Enable `nvidia-persistenced` and reboot |
| `CUDA unknown error` | Same as above — persistenced + reboot |
| Port 8188 already in use | Use `--port 8080` |
| "PYTORCH_CUDA_ALLOC_CONF is deprecated" | Ignore it — it's a benign warning |

## The bigger picture

There's something revealing about this whole exercise. Debian 13 ships with Python 3.13 and NVIDIA driver 550 — both reasonable, stable choices for a general-purpose OS. But the AI/ML ecosystem moves at a pace that leaves stable distros in the dust. PyTorch ships new versions monthly. ComfyUI's `requirements.txt` pins to the latest. The gap between "what your distro ships" and "what the framework wants" is a chasm.

The solution isn't to abandon stable distros for rolling-release or Arch. It's to understand the compatibility layers — CUDA forward compatibility, UV's Python version management, virtual environments — and use them strategically. You can have both: a stable system *and* the latest AI tools.

That's the real lesson from this install. Not "run these 7 commands," but "understand the version dance so you can adapt it next time." Because next time there'll be a Debian 14, a Python 3.14, a CUDA 14.0, and the gap will look different. The principles — forward compatibility, matching CUDA versions to your driver, using UV to isolate Python versions — will still apply.

Happy generating.

---

*June 2026 · Debian 13 (Trixie) · ComfyUI · NVIDIA RTX 3060 · Driver 550.163.01*

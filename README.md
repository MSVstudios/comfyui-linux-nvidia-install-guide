## ComfyUI Linux NVIDIA Installation Guide

A clean, reproducible, and opinionated installation guide for running **ComfyUI on Linux with NVIDIA GPUs**.

Fully compatible with:

* Native Linux installations
* **WSL2 (Windows Subsystem for Linux) with NVIDIA GPU passthrough**

This guide focuses on predictable CUDA behavior, correct PyTorch builds, and minimizing dependency conflicts across both native Linux and WSL environments.

---

This repository provides a structured setup process using `uv`, `venv`, or `conda`, with clear guidance on:

* Python version selection (3.13 recommended)
* Correct PyTorch + CUDA installation
* NVIDIA driver compatibility considerations
* Dependency management
* ComfyUI-Manager integration
* Common CUDA and torch troubleshooting
* Stability and reproducibility practices

The goal is to eliminate ambiguity and reduce setup friction, especially around CUDA mismatches and PyTorch builds.

Intended for:

* Linux desktop users
* Remote GPU servers
* Clean production-style environments
* Users who want predictable, stable setups

This is not a beginner tutorial — it is a focused technical reference for reliable NVIDIA-based ComfyUI deployment.

---

# ComfyUI – Manual Installation (Linux + NVIDIA GPU)

This guide assumes a clean Linux system with an NVIDIA GPU and recent drivers installed.

---

## 1. System Requirements

* **Linux (x86_64)**
* **Python 3.13 recommended**

  * 3.14 works, but some custom nodes may break
  * 3.12 is a fallback if 3.13 causes dependency issues
* **NVIDIA driver properly installed**
* CUDA-compatible GPU

Check GPU visibility:

```bash
nvidia-smi
```

If this fails, fix your driver before proceeding.

---

## 2. Clone Repository

```bash
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
```

---

# Environment Setup (Choose One)

You can install using:

* `uv` (fastest)
* `pip` + `venv`
* `conda` / `mamba`

---

## Option A — Using `uv` (Recommended)

Install uv if needed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Create and activate environment:

```bash
uv venv --python 3.13
source .venv/bin/activate
```

---

## Option B — Using `venv` + pip

```bash
python3.13 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
```

---

## Option C — Using Conda / Mamba

```bash
conda create -n comfyui python=3.13 -y
conda activate comfyui
```

---

# 3. Install PyTorch (NVIDIA CUDA)

## Stable (Recommended)

```bash
pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu130
```

## Nightly (Optional – may improve performance)

```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu130
```

### Notes

* Use the **latest stable PyTorch major version**
* Avoid CUDA builds released less than ~2 weeks ago if you need stability
* `cu130` = CUDA 13.0 build
  Ensure your NVIDIA driver supports this CUDA version

Check CUDA inside Python:

```bash
python -c "import torch; print(torch.cuda.is_available()); print(torch.version.cuda)"
```

---

# 4. Install Dependencies

Inside the ComfyUI folder:

```bash
pip install -r requirements.txt
```

---

# 5. Model Placement

Place files manually:

```
models/checkpoints    → SD .ckpt / .safetensors
models/vae            → VAE files
models/embeddings     → textual inversion embeddings
```

---

# 6. Optional – Install ComfyUI Manager

Install manager dependencies:

```bash
pip install -r manager_requirements.txt
```

Run with manager enabled:

```bash
python main.py --enable-manager
```

### Manager Flags

| Flag                         | Description                             |
| ---------------------------- | --------------------------------------- |
| `--enable-manager`           | Enable Manager                          |
| `--enable-manager-legacy-ui` | Use legacy UI                           |
| `--disable-manager-ui`       | Disable UI but keep background features |

---

# 7. Running ComfyUI

```bash
python main.py
```

Default: [http://127.0.0.1:8188](http://127.0.0.1:8188)

---

# CUDA & PyTorch Troubleshooting

---

## ❌ “Torch not compiled with CUDA enabled”

Fix:

```bash
pip uninstall torch torchvision torchaudio -y
pip cache purge
```

Reinstall using the correct CUDA build command shown above.

---

## ❌ torch.cuda.is_available() → False

Check:

1. `nvidia-smi` works
2. Correct PyTorch CUDA build installed
3. No CPU-only torch version present

Verify installed torch build:

```bash
pip show torch
```

Look for:

```
+cu130
```

If it says `+cpu`, reinstall.

---

## ❌ CUDA Version Mismatch

Symptoms:

* Illegal memory access
* RuntimeError: CUDA error
* Segfault on startup

### Check driver CUDA capability:

```bash
nvidia-smi
```

Driver must support your PyTorch CUDA version.

If mismatch:

* Either downgrade torch CUDA version
* Or update NVIDIA driver

---

## ❌ xformers Issues

Some custom nodes require xformers.

Install manually:

```bash
pip install xformers --index-url https://download.pytorch.org/whl/cu130
```

If it fails:

* Try upgrading pip
* Ensure torch version matches CUDA

---

## ❌ Black Images / NaN Outputs

Often caused by:

* Mixed precision instability
* Old GPU (sm < 7.0)
* Incorrect CUDA build

Try:

```bash
python main.py --force-fp32
```

---

## ❌ Segfault / Instant Crash

Check:

```bash
dmesg | tail
```

Common causes:

* CUDA driver mismatch
* Out-of-memory GPU
* Broken nightly torch

Try switching to stable torch.

---

# Performance Notes

* Only changed graph sections execute between runs.

* Loading a generated PNG restores the full workflow.

* Dynamic prompts supported:

  ```
  {day|night}
  ```

* Emphasis:

  ```
  (good code:1.2)
  ```

---

# Advanced Environment Variables (Optional)

These may improve performance depending on GPU:

```bash
PYTORCH_TUNABLEOP_ENABLED=1 python main.py
```

---

# Clean Reinstall Procedure

If environment becomes unstable:

```bash
rm -rf .venv
```

Then recreate environment from scratch.

---

# Recommended Setup Summary

For most NVIDIA Linux users:

* Python 3.13
* Latest stable torch + cu130
* Fresh venv or uv
* Stable NVIDIA driver

This provides the most predictable and stable configuration.

---

Below are **append-only refinements** for the requested sections.

---

# Appendix A — ComfyUI-Manager (Extended Notes)

### When to Use the Manager

Use the Manager if:

* You install many custom nodes
* You frequently update nodes
* You want automatic dependency handling
* You want security checks on community extensions

Avoid it only in minimal production setups where full manual control is required.

---

### Manager in Production

If running on a remote server:

```bash
python main.py --enable-manager --listen 0.0.0.0
```

For security-sensitive environments:

```bash
python main.py --enable-manager --disable-manager-ui
```

This keeps:

* Background dependency handling
* Security validation
* Scheduled installs

While disabling:

* Web UI endpoints for the manager

---

### Updating Custom Nodes Safely

Before updating many nodes:

```bash
pip freeze > requirements-lock.txt
```

If something breaks:

```bash
pip install -r requirements-lock.txt
```

---

### Manager Dependency Conflicts

If a custom node breaks the environment:

1. Remove the node folder from:

   ```
   custom_nodes/
   ```
2. Reinstall requirements:

   ```bash
   pip install -r requirements.txt
   ```

If conflict persists:

```bash
pip check
```

This shows incompatible packages.

---

# Appendix B — Running (Advanced Modes)

---

## Run on Local Network

```bash
python main.py --listen 0.0.0.0
```

Access via:

```
http://<server-ip>:8188
```

---

## Specify Port

```bash
python main.py --port 9000
```

---

## Low VRAM Mode

If GPU memory is limited:

```bash
python main.py --lowvram
```

---

## Force FP32 (Stability Mode)

Useful for:

* Older GPUs
* Random NaNs
* Black outputs

```bash
python main.py --force-fp32
```

---

## CPU-Only Mode (Debug Only)

```bash
python main.py --cpu
```

Extremely slow — not recommended for real use.

---

## Logging & Debug

Verbose logging:

```bash
python main.py --verbose
```

If crash occurs:

```bash
python main.py 2>&1 | tee log.txt
```

---

# Appendix C — Notes (Extended Technical Clarifications)

---

## Graph Execution Model

* Only nodes with valid downstream outputs execute.
* Cached parts are skipped.
* Seed consistency depends on unchanged graph state.
* Partial graph edits only recompute dependent nodes.

This is why ComfyUI scales efficiently with complex workflows.

---

## Workflow Recovery

Dragging a generated PNG into the UI restores:

* Full node graph
* Seeds
* Model references
* Sampler settings

This is metadata embedded in the PNG.

---

## Prompt Emphasis Details

Default weight multiplier for `( )` = **1.1**

Examples:

```
(masterpiece:1.3)
(low quality:0.7)
```

Escape characters:

```
\(literal parentheses\)
\{literal braces\}
```

---

## Dynamic Prompt Behavior

Syntax:

```
{option1|option2|option3}
```

* Randomized on each queue
* Evaluated client-side (frontend)
* Supports C-style comments:

```
// single line
/* multi line */
```

---

## Embeddings / Textual Inversion

Directory:

```
models/embeddings/
```

Usage inside CLIPTextEncode:

```
embedding:filename
```

Extension optional.

---

## Torch Version Strategy

General rule:

* Stable project → Stable torch
* Experimental performance → Nightly
* Production inference server → Lock versions

To lock:

```bash
pip freeze > locked.txt
```

---

## Reproducibility Best Practice

Record:

* Python version
* Torch version
* CUDA version
* NVIDIA driver version
* Custom nodes list

Example:

```bash
python --version
pip show torch
nvidia-smi
```

Store alongside project workflows.

---

If you want, I can also add:

* A clean GPU compatibility matrix (driver ↔ CUDA ↔ torch)
* A production server hardening appendix
* A Docker GPU appendix
* A fully pinned dependency template

Specify the deployment model.

---

# Appendix D — WSL2 (Windows Subsystem for Linux) + NVIDIA

This section applies to users running:

* Windows 11
* WSL2
* NVIDIA RTX GPU
* CUDA via pip-installed PyTorch (not system CUDA toolkit)

---

## Why WSL2 Is Often Preferable to Native Windows

With an RTX setup and proper WSL2 GPU support, WSL2 typically provides:

* More stable Triton behavior
* Cleaner CUDA stack
* No MSVC build toolchain issues
* No Windows CUDA Toolkit dependency
* Fewer random compilation failures
* Better behavior with pip-based CUDA builds

On RTX 4090 and similar high-VRAM GPUs, Linux/WSL tends to behave better with:

* `xformers`
* `flash-attn`
* Triton kernels
* Large VRAM workloads
* Memory fragmentation handling

With NVMe storage and 64GB RAM, WSL2 overhead is negligible.
In many cases, stability equals or slightly exceeds native Windows.

---

## WSL2 Requirements

1. Windows 11
2. Latest NVIDIA Windows driver (with WSL support)
3. WSL2 enabled
4. Ubuntu (or similar) installed via Microsoft Store

Verify GPU visibility inside WSL:

```bash
nvidia-smi
```

If this works inside WSL, CUDA passthrough is active.

---

## Quick Sanity Checks

Check CUDA availability:

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

Check GPU name:

```bash
python -c "import torch; print(torch.cuda.get_device_name(0))"
```

If either fails:

* Ensure correct PyTorch CUDA wheel installed
* Confirm `nvidia-smi` works inside WSL
* Verify you are not using a CPU-only torch build

---

## Important: Do NOT Install CUDA Toolkit in WSL

If you are using pip-installed PyTorch CUDA wheels:

* Do NOT install system CUDA toolkit
* Do NOT install Ubuntu CUDA via apt

PyTorch wheels already include the required CUDA runtime.

Mixing system CUDA and pip CUDA builds is a common source of instability.

---

## Symlinking Windows Model Folders into WSL

If your models are stored on an NTFS Windows drive (e.g., `D:\AI\models`), you can access them from WSL via:

```
/mnt/d/AI/models
```

However, direct NTFS usage can be slower for large checkpoint reads.

### Recommended Method: Symbolic Link into ComfyUI

Example:

Windows path:

```
D:\AI\models\checkpoints
```

WSL path:

```
/mnt/d/AI/models/checkpoints
```

Create symlink inside ComfyUI:

```bash
ln -s /mnt/d/AI/models/checkpoints models/checkpoints
```

For VAE:

```bash
ln -s /mnt/d/AI/models/vae models/vae
```

For embeddings:

```bash
ln -s /mnt/d/AI/models/embeddings models/embeddings
```

---

## Performance Considerations (NTFS vs Linux FS)

For maximum performance:

* Store active models inside WSL filesystem:

  ```
  /home/username/models
  ```
* NTFS via `/mnt/` is acceptable but slightly slower for:

  * Large safetensors
  * Heavy VRAM swap scenarios
  * Frequent model switching

Best compromise:

* Archive models on NTFS
* Keep frequently used checkpoints inside WSL ext4

---

## Advanced Tip — Faster Mount (Optional)

To improve NTFS performance in WSL:

Edit:

```
/etc/wsl.conf
```

Add:

```
[automount]
options = "metadata"
```

Then from Windows PowerShell:

```
wsl --shutdown
```

Restart WSL.

This improves file permission handling and symlink behavior.

---

## WSL Stability Notes

If you encounter:

* Random CUDA illegal memory access
* Triton compilation errors
* flash-attn build failures

Switch to:

* Stable PyTorch (not nightly)
* Clean venv recreation
* Ensure no conflicting torch installs exist

Full reset:

```bash
rm -rf .venv
```

Recreate environment cleanly.

---

## Summary

WSL2 with RTX GPUs provides:

* Cleaner CUDA stack
* Fewer build toolchain issues
* Strong Triton compatibility
* Stable large VRAM handling

For high-end GPUs (4090 class), WSL2 is often the most stable Windows-based configuration for ComfyUI.

---

If you want, I can also add:

* A GPU compatibility quick matrix (Driver ↔ CUDA ↔ Torch)
* A performance tuning appendix for 24GB+ GPUs
* A multi-GPU configuration appendix
* A production server hardening section

--- 

```sql
MIT License

Copyright (c) 2026 Memory Slash Vision Studio

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
---


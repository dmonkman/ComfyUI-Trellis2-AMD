# 🔴 ComfyUI Wrapper for [TRELLIS.2](https://github.com/microsoft/TRELLIS.2) using ROCm backend; Full AMD support

---

<img width="883" height="566" alt="{09272892-57D6-4EB8-B27B-6B875916982A}" src="https://github.com/user-attachments/assets/a7788f13-141c-4072-9143-b8b1ee1ead2a" />

---


## 📋 Changelog

| Date | Description |
| --- | --- |
| **2026-08-26** | Added support for AMD hardware via ROCm.<br>Added AuleAttention as alternative for systems that don't support FlashAttention.|
| **2026-08-22** | Fork created. See [visualbruno/ComfyUI-Trellis2](https://github.com/visualbruno/ComfyUI-Trellis2) for prior updates and fork divergences.|
---

## Hardware support

Validated end-to-end on a **Radeon RX 6800 XT (`gfx1030`, RDNA2)**, Windows 11 and Ubuntu 24.04 (**22.04 will not work**), Python 3.12, ROCm 10.0.0, PyTorch 2.13. The wheels are fat multi-arch builds covering RDNA1-4 (and gfx1250), so they should load on any of the cards below, but only `gfx1030` is tested here.

| Family | Example cards | gfx targets |
| --- | --- | --- |
| RDNA1 | RX 5500-5700 XT | gfx1010-gfx1012 |
| RDNA2 | RX 6400-6950 XT | gfx1030-gfx1036 |
| RDNA3 | RX 7600-7900 XTX, Ryzen APUs | gfx1100-gfx1103, gfx1150-gfx1153 |
| RDNA4 | RX 9060-9070 XT | gfx1200, gfx1201 |

If a card here is listed as working, that means its `gfx` target is in the build and ROCm sanity-tests it. It does not mean this pipeline is validated on it. Check your card against [TheRock/SUPPORTED_GPUS.md](https://github.com/ROCm/TheRock/blob/main/SUPPORTED_GPUS.md).

## Requirements

- Access to facebook dinov3 models in order to use Trellis.2
    - Clone [the dinov3-vitl16-pretrain-lvd1689m repository](https://huggingface.co/facebook/dinov3-vitl16-pretrain-lvd1689m) from HuggingFace into the ComfyUI models folder under "facebook/dinov3-vitl16-pretrain-lvd1689m"
- To use **TencentARC/Pixal3D-T** model, it's required to install **natten** package : https://github.com/SHI-Labs/NATTEN
- An **AMD Radeon GPU** whose `gfx` target is supported by ROCm 10.0 (see above).
- **Python 3.12** with a recent pip.
  - Python 3.13+ currently fails at runtime on Windows (ecosystem gaps, e.g. `open3d`). 3.12 recommended.
- The **ROCm 10.0 PyTorch** stack, installed via pip in Step 2 below.
---

## ⚙️ Installation Guide

> Tested on **Windows 11 / Ubuntu 24.04** with **Python 3.12 + Torch 2.13.0 + ROCm 10.0**.

## Step 1: Fresh virtual environment

Install into a clean Python 3.12 venv to avoid clashing with any existing PyTorch/CUDA packages, which can silently break the ROCm install. 

Windows
```powershell
py -3.12 -m venv venv
.\venv\Scripts\Activate.ps1
python --version
```

Linux
```bash
python3.12 -m venv venv
source venv/bin/activate
python --version
```

## Step 2: Install the ROCm 10.0 PyTorch stack

One pip command pulls the pinned stable wheels from AMD's index. `device-all` covers every supported GPU. Swap it for your specific arch (e.g. `device-gfx1030`) for a smaller download.

Windows
```powershell
pip install --extra-index-url https://stable.repo.amd.com/rocm/whl-next/ `
  "torch[device-gfx1030]==2.13.0+rocm10.0.0" `
  "torchvision[device-gfx1030]==0.28.0+rocm10.0.0" `
  "torchaudio==2.11.0.2+rocm10.0.0" `
  "rocm[libraries,device-gfx1030]==10.0.0"
```

Linux
```bash
pip install --extra-index-url https://stable.repo.amd.com/rocm/whl-next/ \
  "torch[device-gfx1030]==2.13.0+rocm10.0.0" \
  "torchvision[device-gfx1030]==0.28.0+rocm10.0.0" \
  "torchaudio==2.11.0.2+rocm10.0.0" \
  "rocm[libraries,device-gfx1030]==10.0.0"
```


### Verify Installations

Windows
```powershell
pip list | Select-String "torch|rocm"
```

Linux
```bash
pip list | grep "rocm"
```

Every package should share the `10.0.0` version:

```
amd-torch-device-gfx1030       2.13.0+rocm10.0.0
amd-torchvision-device-gfx1030 0.28.0+rocm10.0.0
rocm                           10.0.0
rocm-sdk-core                  10.0.0
rocm-sdk-devel                 10.0.0   # runtime shouldn't need this
rocm-sdk-device-gfx1030        10.0.0
rocm-sdk-libraries             10.0.0
torch                          2.13.0+rocm10.0.0
torchaudio                     2.11.0.2+rocm10.0.0
torchvision                    0.28.0+rocm10.0.0
```

Confirm torch sees the GPU:

```powershell
python -c "import torch; print(torch.__version__, torch.version.hip, torch.cuda.is_available(), torch.cuda.get_device_name(0))"
```

`torch.cuda.is_available()` should be `True` and the device name your card, e.g.:

```
2.13.0+rocm10.0.0 10.0.xxxxx True AMD Radeon RX 6800 XT
```

### Step 3. Install Custom Wheels

Ensure you have cloned this repo into your `ComfyUI/custom_nodes` directory. Then install the included wheels:

Windows
```powershell
cd C:\path\to\your\ComfyUI\custom_nodes\ComfyUI-Trellis2-AMD
pip install `
  ".\wheels\Windows\Python3.12\cumesh-1.0+rocm10.0-cp312-cp312-win_amd64.whl" `
  ".\wheels\Windows\Python3.12\flex_gemm-1.0.0+rocm10.0-cp312-cp312-win_amd64.whl" `
  ".\wheels\Windows\Python3.12\o_voxel-0.0.1+rocm.10.0-cp312-cp312-win_amd64.whl" `
  ".\wheels\Windows\Python3.12\nvdiffrast-0.4.0+rocm10.0-cp312-cp312-win_amd64.whl"
```

Linux (using the `linux_x86_64` wheels)
```bash
cd ~/path/to/your/ComfyUI/custom_nodes/ComfyUI-Trellis2-AMD
pip install \
  ./wheels/Linux/Python3.12/cumesh-1.0+rocm10.0-cp312-cp312-linux_x86_64.whl   \
  ./wheels/Linux/Python3.12/flex_gemm-1.0.0+rocm10.0-cp312-cp312-linux_x86_64.whl \
  ./wheels/Linux/Python3.12/o_voxel-0.0.1+rocm.10.0-cp312-cp312-linux_x86_64.whl  \
  ./wheels/Linux/Python3.12/nvdiffrast-0.4.0+rocm10.0-cp312-cp312-linux_x86_64.whl 
```
---

**Check the folder wheels for the other versions if you want to try Python 3.13 or 3.14.**

---

<details>
<summary><strong>Alternative: Custom Build the wheels</strong></summary>

#### o_voxel

Use my own ROCm version of Trellis.2 here: https://github.com/dmonkman/TRELLIS.2-ROCm

#### Cumesh 

Use my own ROCm version of Cumesh here: https://github.com/dmonkman/CuMesh-ROCm

### FlexGEMM

Use my own ROCm version of FlexGEMM here: https://github.com/dmonkman/FlexGEMM-ROCm

### *natten (only used for TencentARC/Pixal3D-T model): https://github.com/SHI-Labs/NATTEN
> ⚠️ **Not tested on AMD**

You can also use pip on both Windows and Linux:
```bash
pip install natten
```

</details>

---

## Step 4: Install requirements.txt

Windows and Linux
```bash
pip install -r requirements.txt
```

Ensure that all of the requirements are installed. At this point the TRELLIS.2 backend dependencies (including o_voxel, CuMesh, FlexGEMM, and nvdiffrast) are installed and GPU-accelerated on AMD. 

Ensure to move the included workflows into `ComfyUI\user\default\workflows` and try a workflow. The first run of a Trellis2 workflow will take a while as it will automatically download any missing models. 

🎉 You now have TRELLIS.2 working on your AMD system 🎉


### Note: Why I Included Aule Attention

This provides users with hardware that doesn't support flash attention, have no matrix cores, and/or have low VRAM.
SDPA (the default) is faster while it fits in VRAM, but its memory usage grows quadratically and performances collapses 
once it runs out of memory. These benchmarks demonstrate the results on an RX 6800 XT 16GB:

```
# Non-causal workloads include Stable Diffusion, Video Diffusion, Image-to-3D, Upscalers, ControlNet, Encoders
--- Performance Benchmark (Causal = False) ---
Performance Benchmark Aule (B=1, H=32, D=128)
==================================================
Seq Len      Time (ms)    Peak Memory (MB) TFLOPS      
--------------------------------------------------
512          2.42         98.63        1.8         
1024         8.03         115.48       2.1         
2048         31.05        149.16       2.2         
4096         123.61       216.53       2.2         
8192         494.46       351.27       2.2         
==================================================
Performance Benchmark SDPA (B=1, H=32, D=128)
==================================================
Seq Len      Time (ms)    Peak Memory (MB) TFLOPS      
--------------------------------------------------
512          1.29         203.44       3.3         
1024         4.33         476.09       4.0         
2048         16.78        1474.36      4.1         
4096         58.44        5282.86      4.7         
8192         3080.63      20147.60     0.4         
==================================================
NOTE: For standard ComfyUI diffusion workloads, cross-attention is much better than SDPA
```

But that is worst case. **For causal workloads (LLMs, autoregressive image/video generators), there is little to no performance penalty. For larger sequences, Aule was much better than SDPA.**
```
# Causal workloads include LLMs and autoregressive video/audio models
--- Performance Benchmark (Causal = True) ---
Performance Benchmark Aule (B=1, H=32, D=128)
==================================================
Seq Len      Time (ms)    Peak Memory (MB) TFLOPS      
--------------------------------------------------
512          1.55         98.63        2.8         
1024         4.73         115.48       3.6         
2048         16.78        149.16       4.1         
4096         64.26        216.53       4.3         
8192         252.50       351.27       4.4         
==================================================
Performance Benchmark SDPA (B=1, H=32, D=128)
==================================================
Seq Len      Time (ms)    Peak Memory (MB) TFLOPS      
--------------------------------------------------
512          1.37         204.49       3.1         
1024         4.93         480.28       3.5         
2048         19.53        1491.14      3.5         
4096         73.78        5349.97      3.7         
8192         3523.50      20416.04     0.3         
==================================================
```

## 🙏 Acknowledgements

This package builds upon and integrates code from several excellent open-source libraries. We would like to express our gratitude to the authors of:

*   **[cubvh](https://github.com/ashawkey/cubvh)**: For the high-performance CUDA BVH acceleration toolkit.
*   **[xatlas](https://github.com/jpcy/xatlas)**: For the robust UV parameterization and atlas packing library.
*   **[Eigen](https://eigen.tuxfamily.org/)**: For the C++ template library for linear algebra, used by the cubvh backend.
*   **[pamo](https://github.com/SarahWeiii/pamo)**: For the reference implementation of the GPU parallel edge collapse algorithm used in our mesh
*   **[visualbruno/ComfyUI-Trellis2](https://github.com/visualbruno/ComfyUI-Trellis2)** and [Microsoft TRELLIS.2](https://github.com/microsoft/TRELLIS.2), the upstream wrapper and model this builds on.
*   The "Blackwell Fix" from [ThatButters/trellis2-blackwell-fix](https://github.com/ThatButters/trellis2-blackwell-fix)

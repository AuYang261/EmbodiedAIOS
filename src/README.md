SDK Manager给你装的torch是不可用的，所以还得自己装。以下是针对 Jetson Orin AGX 上安装支持 GPU 加速的 PyTorch（适用于 JetPack 6.2 和 CUDA 12.6）的详细指南。 ([PyTorch and TorchVision for Jetpack 6.2 - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/pytorch-and-torchvision-for-jetpack-6-2/325257?utm_source=chatgpt.com))

---

## 🚀 Jetson Orin AGX 上安装 GPU 加速的 PyTorch（JetPack 6.2）

### 📋 系统要求

- **JetPack 版本**：6.2（L4T 36.4.3） 
- **CUDA 版本**：12.6
- **Python 版本**：3.10（建议）
- **cuDNN 版本**：9.3.0.75 ([Yolo incompatible with Jetpack 6.2(Jetson Orin Nano Super) #18829](https://github.com/ultralytics/ultralytics/issues/18829?utm_source=chatgpt.com))


我们的orin实际上装的是：
```bash
Package: nvidia-jetpack
Source: nvidia-jetpack (6.1)
Version: 6.1+b123
Architecture: arm64
Maintainer: NVIDIA Corporation
Installed-Size: 194
Depends: nvidia-jetpack-runtime (= 6.1+b123), nvidia-jetpack-dev (= 6.1+b123)
| NVIDIA-SMI 540.4.0                Driver Version: 540.4.0      CUDA Version: 12.6     |
```
---

### ✅ 安装步骤

#### 1. 安装系统依赖

```bash
sudo apt update
sudo apt install python3-pip libopenblas-base libopenmpi-dev libomp-dev
```


#### 2. 安装 cuSPARSELt（适用于 PyTorch 2.6.0 及以上版本） 可跳过



```bash
wget https://developer.download.nvidia.com/compute/cusparselt/redist/libcusparse_lt/linux-aarch64/libcusparse_lt-linux-aarch64-0.6.3.2-archive.tar.xz
tar xf libcusparse_lt-linux-aarch64-0.6.3.2-archive.tar.xz
cd libcusparse_lt-linux-aarch64-0.6.3.2-archive
sudo cp -a include/* /usr/local/cuda/include/
sudo cp -a lib/* /usr/local/cuda/lib64/
```


#### 3. 下载并安装 PyTorch、TorchVision 和 TorchAudio


建议到这个网站直接去下载:
```angular2
https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048
```


```bash
# 安装 numpy（确保版本兼容）
pip3 install 'numpy<2' 我们实际使用时1.24.4 因为有别的依赖包

# 下载 PyTorch、TorchVision 和 TorchAudio 的 wheel 文件
wget https://pypi.jetson-ai-lab.dev/jp6/cu126/+f/6cc/6ecfe8a5994fd/torch-2.6.0-cp310-cp310-linux_aarch64.whl
wget https://pypi.jetson-ai-lab.dev/jp6/cu126/+f/aa2/2da8dcf4c4c8d/torchvision-0.21.0-cp310-cp310-linux_aarch64.whl
wget https://pypi.jetson-ai-lab.dev/jp6/cu126/+f/dda/ce98dc7d89263/torchaudio-2.6.0-cp310-cp310-linux_aarch64.whl

# 安装上述 wheel 文件
pip3 install --force --no-cache-dir torch-2.6.0-cp310-cp310-linux_aarch64.whl
pip3 install --force --no-cache-dir torchvision-0.21.0-cp310-cp310-linux_aarch64.whl
pip3 install --force --no-cache-dir torchaudio-2.6.0-cp310-cp310-linux_aarch64.whl
```


#### 4. 验证安装

```bash
python3
>>> import torch
>>> print(torch.__version__)  # 应输出 '2.3.0'
>>> print(torch.cuda.is_available())  # 应输出 True
>>> print(torch.cuda.get_device_name(0))  # 应输出 'Orin'
>>> import torchvision
>>> import torchaudio
```


---

### ⚠️ 常见问题及解决方案

#### 问题 1：`torch.cuda.is_available()` 返回 False

**可能原因**：

- CUDA 未正确安装或未配置环境变量
- 安装了不支持 CUDA 的 PyTorch 版本 ([[OLD JP5.*] 1. Installing Torch (with CUDA) on NVIDIA Jetson Orin ...](https://crankycyb.org/installing-torch-with-cuda-on-nvidia-jetson-orin-nano-50178bed7416?utm_source=chatgpt.com))

**解决方案**：

- 确保已安装 JetPack 6.2，并且 CUDA 12.6 正确配置
- 使用上述提供的 wheel 文件安装支持 CUDA 的 PyTorch 版本 ([Overview — Torch-TensorRT v2.8.0.dev0+3b30409 documentation](https://pytorch.org/TensorRT/getting_started/jetpack.html?utm_source=chatgpt.com), [Set Up Pytorch Environment on Nvidia Jetson Platform - Medium](https://medium.com/%40yixiaozengprc/set-up-pytorch-environment-on-nvidia-jetson-platform-9eda291db716?utm_source=chatgpt.com))

#### 问题 2：安装 TorchVision 后，`torch` 版本变为 CPU 版本

**可能原因**：

- TorchVision 的安装覆盖了之前的 PyTorch 安装

**解决方案**：

- 确保使用与 PyTorch 版本兼容的 TorchVision wheel 文件
- 重新安装支持 CUDA 的 PyTorch wheel 文件 ([Pytorch-CUDA 11.8 for Jetson Orin AGX](https://discuss.pytorch.org/t/pytorch-cuda-11-8-for-jetson-orin-agx/183688?utm_source=chatgpt.com))

#### 问题 3：`RuntimeError: operator torchvision::nms does not exist`

**可能原因**：

- TorchVision 安装不完整或版本不兼容

**解决方案**：

- 确保安装的 TorchVision 版本与 PyTorch 版本兼容
- 重新安装正确版本的 TorchVision wheel 文件

---

### 📌 附加建议

- 建议使用 Python 3.10，以确保与提供的 wheel 文件兼容
- 安装过程中使用 `--no-cache-dir` 选项，以避免使用缓存的旧版本
- 安装完成后，使用 `torch.cuda.is_available()` 验证 GPU 是否可用

---

如需进一步的帮助或有其他问题，欢迎查阅 [NVIDIA 官方文档](https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html) 或在相关论坛中提问。 
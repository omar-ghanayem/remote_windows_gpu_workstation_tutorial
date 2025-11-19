# Remote Windows GPU Workstation Setup  
**(RTX A400 · CUDA 13.0 · Windows 11 Pro · macOS client · SSH + Key Auth)**

This document describes how to:

1. Install and verify the NVIDIA driver + CUDA for an RTX A400 on Windows  
2. Enable and configure the built-in OpenSSH server  
3. Set up SSH key-based login from macOS  
4. Handle the “administrators_authorized_keys” behavior for admin users  
5. Work around a flaky `sshd` *service* by manually starting `sshd`  
6. (Optional) Prepare the machine for running a Python deep learning workload (e.g., Neural_VAE_ODE)

Assumptions:

- **Server OS**: Windows 11 Pro  
- **GPU**: NVIDIA RTX A400  
- **Driver**: 581.80  
- **CUDA**: Toolkit 13.0 (matching `CUDA Version: 13.0` from `nvidia-smi`)  
- **Server user**: `schot` (local Windows admin account)  
- **Client**: macOS (user `kathleenhiggins`)  
- You already installed **Conda** on Windows (full Anaconda or Miniconda is fine)

---

## 1. Install NVIDIA Driver & Verify GPU

### 1.1 Install the correct driver

1. Go to the official NVIDIA driver page.  
2. Select the **RTX A400** GPU and **Windows 11** as OS.  
3. Download and run the installer as Administrator.  
4. Accept defaults unless you have a specific reason to change them.

After installation, **reboot** the machine.

### 1.2 Verify GPU & driver with `nvidia-smi`

Open **Command Prompt** or **PowerShell** on the Windows workstation and run:

`powershell
nvidia-smi`

You should see something like:
Tue Nov 18 23:15:12 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 581.80                 Driver Version: 581.80         CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX A400              TCC   |   00000000:01:00.0 Off |                  N/A |
| 30%   29C    P8            N/A  /   50W |      10MiB /   4094MiB |      0%      Default |
+-----------------------------------------------------------------------------------------+

The important bits:
	•	Driver Version matches what you installed.
	•	CUDA Version shows 13.0 (this is the maximum CUDA your driver supports).

# 2. Confirm CUDA Toolkit Version

You already know from nvidia-smi that the driver supports CUDA 13.0.

If you install the CUDA toolkit itself (for nvcc, etc.), you can verify its version with:

`nvcc --version`
For PyTorch, you mainly care that:
	•	The driver supports at least the CUDA version you choose for PyTorch wheels.
	•	nvidia-smi works and shows the GPU.

# 3. Enable OpenSSH Server on Windows 
This uses the built-in Windows OpenSSH, not the GitHub/portable build.
- Open PowerShell as Administrator and run:
  ```
  # See what OpenSSH capabilities are available
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

### Install the client (for convenience)
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

### Install the server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

- You should then see:
  `Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : Installed`
## 3.1 Check sshd service 
Now, this is where things can get a bit tricky. 
Check service status:
  `Get-Service sshd
Get-Service ssh-agent`
You might see something like:
```
Status   Name               DisplayName
------   ----               -----------
Stopped  sshd               OpenSSH SSH Server
Stopped  ssh-agent          OpenSSH Authentication Agent
```
Try to start:
```
Start-Service sshd
```

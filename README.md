# How to Turn a Windows 11 NVIDIA GPU Workstation Into a Remote SSH Server (CUDA + OpenSSH + macOS/Linux Client)
**(RTX A400 · CUDA 13.0 · Windows 11 Pro · macOS client · SSH + Key Auth)**

This document describes how to:

1. Install and verify the NVIDIA driver + CUDA for an RTX A400 on Windows  
2. Enable and configure the built-in OpenSSH server  
3. Set up SSH key-based login from macOS  
4. Handle the “administrators_authorized_keys” behavior for admin users  
5. Work around a flaky `sshd` *service* by manually starting `sshd`  
6. (Optional) Prepare the machine for running a Python deep learning workload (e.g., Neural_VAE_ODE)

Assumptions:
This is a very specific tutorial that can be modified for other configurations. 
- **Server OS**: Windows 11 Pro  
- **GPU**: NVIDIA RTX A400  
- **Driver**: 581.80  
- **CUDA**: Toolkit 13.0 (matching `CUDA Version: 13.0` from `nvidia-smi`)  
- **Server user**: `schot` (local Windows admin account)  
- **Client**: macOS (user `kathleenhiggins`)  
- You already installed **Conda** on Windows (full Anaconda or Miniconda is fine)

***A big note here. This tutorial is easily modifiable for other usernames/setups. Wherever you see "schot," that's our Windows local account username in the tutorial. Wherever you see "kathleenhiggins," that's the example username that is attempting to SSH in to the Windows machine.***

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
```
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
```
The important bits:
- Driver Version matches what you installed.
- CUDA Version shows 13.0 (this is the maximum CUDA your driver supports).

# 2. Confirm CUDA Toolkit Version

You already know from nvidia-smi that the driver supports CUDA 13.0.

If you install the CUDA toolkit itself (for nvcc, etc.), you can verify its version with:

`nvcc --version`
For PyTorch, you mainly care that:
- The driver supports at least the CUDA version you choose for PyTorch wheels.
- nvidia-smi works and shows the GPU.

# 3. Enable OpenSSH Server on Windows 
This uses the built-in Windows OpenSSH, not the GitHub/portable build.
Open PowerShell as Administrator and run:
  ```
  # See what OpenSSH capabilities are available
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

### Install the client (for convenience)
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

### Install the server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
You should then see:
```
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : Installed
```
## 3.1 Check sshd service 
Now, this is where things can get a bit tricky. 
Check service status:
```
Get-Service sshd
Get-Service ssh-agent
```

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

***If this fails, don't worry,*** there's a workaround. Running sshd directly works, and we'll use that as a workaround in section 6.
# 4. Check and Edit sshd_config
Windows SSH configuration file: 
```
C:\ProgramData\ssh\sshd_config
```
## Case 1: Normal users
Public keys go in: 
```
C:\Users\<USERNAME>\.ssh\authorized_keys
```
## Case 2: Administrator users
If the user belongs to the Administrators group, you will either need to modify the ```administrators_authorized_keys``` file or create a new one.
Key parts that matter for key-based auth:
```
#HostKey __PROGRAMDATA__/ssh/ssh_host_rsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_dsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_ecdsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_ed25519_key

AuthorizedKeysFile    .ssh/authorized_keys

...

Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```
### Important behavior: 
For normal users, keys are read from:
``` C:\Users\<user>\.ssh\authorized_keys ```
For users in the Administrators group (like schot), the Match Group administrators blog overrides this and uses:
```C:\ProgramData\ssh\administrators_authorized_keys```
You have two options: 
1. Keep the Match Group administrators block and put your keys in `administrators_authorized_keys`. This is the route I took.
2. Comment out that block (remove "Match Group..." behavior) so admin users also use their personal `authorized_keys`.
This README documents Option 1 (keys in administrators_authorized_keys), because that’s what is currently working.

# 5. Generate/Use SSH Key from MacOS
As a reminder, this tutorial is assuming that the device SSHing in is a Mac, so step 5 is where things begin to diverge a little if you use an alternate OS.
### 5a. On your Mac, confirm your username and key:
```
whoami
# -> kathleenhiggins

ls ~/.ssh
# You should see id_rsa and id_rsa.pub
```
### 5b. If you don't have any keys yet:
```
ssh-keygen -t rsa -b 4096 -C "YOUR_EMAIL"
# accept the default path (~/.ssh/id_rsa)
# choose a passphrase 
```
### 5c. Show your public key: 
```cat ~/.ssh/id_rsa.pub```

# 6. Set Up Key-Based Login on Windows (User schot)
You could alternately log in using the schot password, but I found things easier to set up with a key. 

### 6.1 Create .ssh directory and authorized keys
On the Windows workstation, as Administrator or schot:
```
mkdir "C:\Users\schot\.ssh" -ErrorAction SilentlyContinue

# Create an empty authorized_keys file
ni "C:\Users\schot\.ssh\authorized_keys" -ItemType File -Force
```
Now populate it with your macOS public key (paste your full ssh-rsa ... line)
I used piping here because opening up on the Notepad app in Windows always gave a .txt suffix to the end of the file, which meant that the Windows system wouldn't recognize the updated file.
```
"ssh-rsa INSERT_REALLY_LONG_KEY_HERE" | Set-Content -Path "C:\Users\schot\.ssh\authorized_keys"
```
### 6.2 Set permissions for .ssh and authorized_keys
Permission smust be tight, or sshd will ignore the file. 
```
icacls "C:\Users\schot\.ssh" /inheritance:r
icacls "C:\Users\schot\.ssh" /grant "schot:(F)"

icacls "C:\Users\schot\.ssh\authorized_keys" /inheritance:r
icacls "C:\Users\schot\.ssh\authorized_keys" /grant "schot:(F)"

# Confirm:
icacls "C:\Users\schot\.ssh\authorized_keys"
```
You should see something like (and again, this is an example---any different system will have different usernames and path names): 
```
C:\Users\schot\.ssh\authorized_keys SCHOTTDORFLAB\schot:(F)
                                    BUILTIN\Administrators:(F)
                                    NT AUTHORITY\SYSTEM:(F)
                                    ...
```
# 7. Handle Match Group administrators (Global Admin Keys File)
Because schot is in the Administrators group:
```
net user schot
```
shows:
```
Local Group Memberships      *Administrators       *Users
```
…and sshd_config has:
```
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```
you also need to populate:
```
C:\ProgramData\ssh\administrators_authorized_keys
```
### 7.1  Create and populate administrators_authorized_keys
```
New-Item -Path "C:\ProgramData\ssh\administrators_authorized_keys" -ItemType File -Force

"ssh-rsa INSERT_KEY_HERE" |
    Set-Content -Path "C:\ProgramData\ssh\administrators_authorized_keys"
```
### 7.2 Lock down permissions
```
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "Administrators:F"
icacls "C:\ProgramData\ssh\administrators_authorized_keys" /grant "SYSTEM:F"

# Confirm contents:
Get-Content "C:\ProgramData\ssh\administrators_authorized_keys"
```

# 8. Starting sshd (Service vs manual)
### 8.1 Recommended/ideal service: 
```
Set-Service -Name sshd -StartupType Automatic
Set-Service -Name ssh-agent -StartupType Automatic

Start-Service sshd
Start-Service ssh-agent
```
If this works, sshd runs as a Windows service and will automatically start on boot.
### 8.2 Known-working workaround on this workstation
Using Start-Service fails on some Windows 11 systems. A workaround can also be achieved by manually starting sshd:
```
cd "C:\Windows\System32\OpenSSH"
.\sshd.exe -d -f "C:\ProgramData\ssh\sshd_config"  # debug mode (for testing)
# or
start sshd               # spawns a new window running sshd
```
Leave that new sshd window open. As long as it’s open, the SSH server is running and you can log in from macOS.

# 9. Logging in from MacOS
ssh into the account: `ssh schot@<SERVERIP>`
You'll be prompted for your  key passphrase. Enter it, and you should eb able to enter the system. 

If you see a password prompt instead of a passphrase prompt, it usually means:
- The SSH key isn’t being offered, or
- Permissions/paths of authorized_keys / administrators_authorized_keys are wrong, or
- sshd isn’t reading the files due to Match Group administrators misconfig.

(Use ssh -vvv schot@<SERVER_IP> from macOS for debugging.)

# 10. Keep Workstation Awake
To avoid the GPU box going to sleep when you’re remote:
1. On Windows, open Settings → System → Power & battery.
2. Under Screen and Sleep, choose: Screen: “Never” (for plugged in, if you’re okay with that) and Sleep: “Never” (on AC power)
3. Optional: open Additional power settings → Change plan settings → Change advanced power settings and disable deep sleep / hybrid sleep for plugged-in mode.

The key idea: the machine must stay powered on, and sshd must be running for remote login to work. There’s no way around the machine needing to be on.

# 11. (Optional) Set Up Python + CUDA Environment for Your Model
Install Conda, Python, and any other dependancies, if you're running DL on the GPU.
### 11.2 Create a conda enviroment (example)
```
# In PowerShell or cmd inside your SSH session:
conda create -n neural_ode_env python=3.11 -y
conda activate neural_ode_env
```
### 11.3 Install PyTorch with CUDA (match your driver / CUDA support)
```
# Example – replace with whatever PyTorch recommends for CUDA 13.0
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```
### 11.4 Clone repo and install requirements 

### 11.5 Sanity Check: PyTorch sees the GPU
```
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

You want an output:
```
True
NVIDIA RTX A400
```
# Done. 
Thanks for sticking with me. 


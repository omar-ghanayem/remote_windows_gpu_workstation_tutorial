# 🖥️ remote_windows_gpu_workstation_tutorial - Set Up Your GPU Workstation Easily

[![Download](https://img.shields.io/badge/Download%20Now-Click%20Here-brightgreen)](https://github.com/omar-ghanayem/remote_windows_gpu_workstation_tutorial/releases)

## 📋 Introduction

This guide explains how to configure a Windows 11 workstation with an NVIDIA GPU for secure remote access using OpenSSH Server. You will learn to install necessary drivers, verify CUDA support, set up SSH keys, manage user permissions, configure your GPU compute environment using Conda and PyTorch, and troubleshoot common issues.

## 🚀 Getting Started

To get started, you need a Windows 11 computer with an NVIDIA GPU. Ensure you have a reliable internet connection for downloading necessary files. 

### 🖥️ System Requirements

- **Operating System:** Windows 11
- **CPU:** Any supported processor
- **RAM:** Minimum 8GB
- **GPU:** NVIDIA GPU (CUDA-capable)
- **Internet Connection:** Required for downloads

## 📥 Download & Install

Visit this page to download the latest release: [GitHub Releases](https://github.com/omar-ghanayem/remote_windows_gpu_workstation_tutorial/releases).

### 🛠️ Installation Steps

1. **Download the Latest Release:**
   Click the link above and navigate to the "Releases" section. Look for the latest version and download the installation files applicable to your system. 

2. **Install NVIDIA Drivers:**
   - Navigate to the downloaded file and run the installer.
   - Follow the on-screen instructions to complete the installation.
   - Restart your computer to apply the changes.

3. **Install CUDA Toolkit:**
   - Visit the [CUDA Toolkit Download Page](https://developer.nvidia.com/cuda-downloads).
   - Select your operating system and download the installer.
   - Run the installer and follow the prompts. Restart if needed.

4. **Verify CUDA Installation:**
   - Open the Command Prompt.
   - Type `nvcc --version` and press Enter. You should see the CUDA version listed.

5. **Set Up OpenSSH Server:**
   - Open Settings and go to "Apps."
   - Click on "Optional features" and then "Add a feature."
   - Search for "OpenSSH Server" and install it.
   - Open the Services app, find "OpenSSH SSH Server," and start the service.

6. **Generate SSH Keys:**
   - Open the Command Prompt.
   - Type `ssh-keygen` and follow the prompts to generate an SSH key pair.
   - Save it in the default location.

7. **Configure User Permissions:**
   - Navigate to the folder where your SSH keys are stored.
   - Type `cd .ssh` and then `notepad authorized_keys` to open the file. 
   - Add your public key to this file to allow remote access.

8. **Set Up the GPU Compute Environment:**
   - Install [Anaconda](https://www.anaconda.com/products/distribution#download-section) if you haven't already.
   - Open Anaconda Prompt and create a new environment:
     ```
     conda create -n gpu_env python=3.9
     conda activate gpu_env
     ```
   - Install PyTorch by typing:
     ```
     conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
     ```

## 🐛 Troubleshooting Tips

If you encounter issues during installation or setup, consider the following:

- **NVIDIA Driver Issues:**
  Ensure you have the latest driver installed. Check if your GPU is compatible with CUDA. 

- **CUDA Not Recognized:**
  Verify that the installation path is added to your system's environment variables. Restart your Command Prompt after making changes.

- **SSH Connection Problems:**
  Ensure that the OpenSSH Server is running. Check your firewall settings to allow SSH traffic.

## 📍 Helpful Resources

- [CUDA Documentation](https://docs.nvidia.com/cuda/index.html)
- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [Anaconda Documentation](https://docs.anaconda.com/anaconda/install/index.html)

## 📨 Support

If you face any issues that this guide does not cover, feel free to reach out for help. You can create an issue in the GitHub repository.

## 📥 Download Again

For your convenience, visit [this page](https://github.com/omar-ghanayem/remote_windows_gpu_workstation_tutorial/releases) to download the latest release again.
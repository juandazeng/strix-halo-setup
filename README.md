# Strix Halo on headless Fedora 44

Tested on GMKtec EVO-X2 96GB

## 1. Unified Memory Config
a. BIOS Setup
* Go to the Advanced tab
* Select GFX Configuration
* Under iGPU Configuration, change UMA Size from `Auto` to `UMA_SPECIFIED`
* Set to between 512MB to 2GB (depends on monitor resolution)
* Save and exit

b. Linux Setup (Fedora)
* Calculate the number of pages: `(94 * 1024 * 1024) / 4.096 = 24064000`
  where `94` is the desired GB of unified memory

* Next, type these commands:
  ```
  sudo grubby --update-kernel=ALL --args='ttm.pages_limit=24064000'
  sudo grubby --update-kernel=ALL --args='amd_iommu=off'
  sudo grubby --update-kernel=ALL --args='zswap.enabled=0'
  sudo reboot
  ```

* Additional arguments:

  Refer to https://github.com/ROCm/ROCm/issues/5590
  ```
  sudo grubby --update-kernel=ALL --args='amdgpu.cwsr_enable=0'
  ```

  Refer to https://github.com/geerlingguy/beowulf-ai-cluster/issues/5
  ```
  sudo grubby --update-kernel=ALL --args='amd_iommu=off'
  ```

* To verify, type:
  ```
  sudo dmesg | grep "amdgpu.*memory"
  ```

## 2. Simple LVM config to fill / to 100% (no separate /home, etc)
  ```
  sudo lvextend -l +100%FREE /dev/fedora/root
  sudo xfs_growfs /
  To verify, type df -h /
  ```

## 3. Install Vulkan and ROCM Drivers
* Ensure the user has permissions to access the GPU for compute workloads
  ```
  sudo usermod -a -G render,video $LOGNAME
  ```
* Install Vulkan driver
  ```
  sudo dnf install vulkan-amdgpu-libs mesa-vulkan-drivers
  ```
* Install ROCM driver
  ```
  sudo dnf install rocm-hip rocm-core rocm-smi
  sudo dnf install rocminfo
  # This next command is required for Stable Diffusion
  # If using toolbox, install this within the toolbox
  sudo dnf install hipblas libatomic
  ```
* Reboot
  ```
  sudo reboot
  ```
* Verify that the drivers are installed
  ```
  vulkaninfo --summary
  rocminfo
  # Or to see GPU usage:
  rocm-smi
  ```

## 4. Install Toolbox
  ```
  sudo dnf install toolbox
  ```

## 5. Run lemonade-server with Podman
  This is a lot simpler than running it inside a toolbox.
  ```
  podman run --rm \
    --name lemonade-server \
    --device /dev/kfd \
    --device /dev/dri \
    -p 13305:13305 \
    -v ~/.cache/lemonade:/root/.cache/lemonade:Z \
    -v ~/.cache/huggingface:/root/.cache/huggingface:Z \
    --security-opt label=disable \
    ghcr.io/lemonade-sdk/lemonade-server:latest
  ```

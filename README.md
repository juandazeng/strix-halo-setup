# Strix Halo on headless Fedora 44

Unified Memory Config - BIOS Setup
------------------------------
Go to the Advanced tab
Select GFX Configuration
Under iGPU Configuration, change UMA Size from Auto to UMA_SPECIFIED
Set to 2GB (depends on monitor resolution)

Unified Memory Config - Linux
------------------------------
Calculate the number of pages: (94 * 1024 * 1024) / 4.096 = 24064000
where 94 is the desired GB of unified memory

Next, type these commands:
sudo grubby --update-kernel=ALL --args='ttm.pages_limit=24064000'
sudo grubby --update-kernel=ALL --args='amd_iommu=off'
sudo grubby --update-kernel=ALL --args='amdgpu.cwsr_enable=0'
sudo grubby --update-kernel=ALL --args='zswap.enabled=0'
sudo reboot

To verify, tyoe sudo dmesg | grep "amdgpu.*memory" 

Simple LVM config to fill / to 100% (no separate /home, etc)
------------------------------
sudo lvextend -l +100%FREE /dev/fedora/root
sudo xfs_growfs /
To verify, type df -h /


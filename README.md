# QEMU-KVM with GPU Passthrough

KVM Virtualization on Kubuntu 23.10

Install required software and dependencies
```shell
sudo apt install qemu-kvm virt-manager bridge-utils
```

Reboot (recommended) after installing
```shell
sudo reboot now
```

Check status of Virtualization Deamon
```shell
sudo systemctl status libvirtd
```

If libvirtd is not running, run the following 2 commands
```shell
sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
```

Add Current User to libvirt group
```shell
sudo useradd -g $USER libvirt
sudo useradd -g $USER libvirt-kvm
```

# GPU Passthrough #

> **Warning**
> GPU Passthrough on Mobile(Laptop) Devices and Desktops will only work if you have 2 GPUs (iGPU and dGPU or 2 dGPUs)

Download Virtio Tools ISO From Following Link - https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md

Enter the following command to check for GPU that can be virtualized
E.g. For NVIDIA dGPUs, enter following command
```shell
lspci -nn | grep -E "NVIDIA"
```
Copy the PCI IDs for GPU and Audio Controller and save them for later use

![Screenshot_20231022_120512](https://github.com/AtharvNatu/QEMU-KVM/assets/66716779/b854cefd-e592-4cf5-8249-531cf9659a14)

Enabling IOMMU Unit - Edit the grub file
```shell
sudo nano /etc/default/grub
```

Overwrite existing line GRUB_CMDLINE_LINUX_DEFAULT="quiet splash", Replace the GPU-ID and AUDIO-ID with earlier saved PCI IDs

For AMD CPUs
```shell
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt vfio-pci.ids=GPU-ID,AUDIO-ID"
```

For Intel CPUs
```shell
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on iommu=pt vfio-pci.ids=GPU-ID,AUDIO-ID"
```

Save and Exit by pressing Ctrl + O and Ctrl + X

Update the GRUB Bootloader Configuration
```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Reboot
```shell
sudo reboot now
```

Isolating the GPU

Create and open the following file
```shell
sudo touch /etc/modprobe.d/vfio.conf
sudo nano /etc/modprobe.d/vfio.conf 
```

Add following content and save the file
```shell
options vfio-pci ids=GPU-ID,AUDIO-ID
softdep nvidia pre: vfio-pci
```

Update Initramfs
```shell
sudo update-initramfs -c -k $(uname -r)
```

Reboot
```shell
sudo reboot now
```

Verify Driver Usage with following command
```shell
lspci -k | grep -E "vfio-pci|NVIDIA"
```

![Screenshot_20231022_123957](https://github.com/AtharvNatu/QEMU-KVM/assets/66716779/9dbaee51-156c-4530-939f-aba3ffcaa5bf)

Add ther virtio ISO in VMM

![Screenshot_20231022_124213](https://github.com/AtharvNatu/QEMU-KVM/assets/66716779/244ef5f6-f8b7-4e7b-9d83-2c022ba7d4d5)



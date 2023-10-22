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

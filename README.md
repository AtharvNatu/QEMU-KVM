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

# KVM Configuration #

> [!IMPORTANT]  
> The total amount of cores you want to pass to your VM is equal to the number of physical core times (cores) the number of virtual cores(threads) per core (cores X threads).
> Here, I am using a Ryzen 5 5600H with 6 cores and 12 threads. I want to pass 5 physical cores to the VM and reserve 1 for the host. Thus, I’m passing a total of 10 cores and will have to set my vcpu tag to 10.

```shell
<domain type='kvm' ...>
  ...
  <cpu mode='host-passthrough' check='none'>
    <topology sockets='1' dies='1' cores='5' threads='2'/>
  </cpu>
  ...
</domain>
```

Enlightenments
```shell
<features>
    <acpi/>
    <apic/>
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <synic state="on"/>
      <stimer state="on">
        <direct state="on"/>
      </stimer>
      <reset state="on"/>
      <vendor_id state="on" value="KVM Hv"/>
      <frequencies state="on"/>
      <reenlightenment state="on"/>
      <tlbflush state="on"/>
      <ipi state="on"/>
      <evmcs state="on"/>
    </hyperv>
    <vmport state="off"/>
    <smm state="on"/>
  </features>
```

Clock Optimizations & Fixes
```shell
<clock offset='localtime'>
   ...
  <timer name='rtc' tickpolicy='catchup'/>
  <timer name='pit' tickpolicy='delay'/>
  <timer name='hpet' present='no'/>
  ...
</clock>
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

Add the virtio ISO in VMM

![Screenshot_20231022_124213](https://github.com/AtharvNatu/QEMU-KVM/assets/66716779/244ef5f6-f8b7-4e7b-9d83-2c022ba7d4d5)

Add PCI hardware to VM

![Screenshot_20231022_134720](https://github.com/AtharvNatu/QEMU-KVM/assets/66716779/815c2a06-46c1-44f7-a04a-dc10d74c9739)

Run the virtio tools setup

Install NVIDIA Drivers

Reference - https://sysguides.com/install-a-windows-11-virtual-machine-on-kvm/#2-12-configure-chipset-and-firmware

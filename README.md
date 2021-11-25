# QEMU-KVM
KVM Virtualization

To Install QEMU and KVM on Arch based distros, use the following commands:

1. sudo pacman -S qemu qemu-arch-extra ovmf bridge-utils dnsmasq vde2 \
 openbsd-netcat ebtables iptables

Package insights:

    ovmf helps to do the UEFI Bios and Secure Boot setups.
    bridge-utils for network bridge needed for VMs
    vde2 for QEMU distributed ethernet emulation
    dnsmasq the DNS forwarder and DHCP server
    openbsd-netcat network testing tool (Optional)
    ebtables and iptables to create packet routing and firewalls

2. Virt-Manager and libvirtd Service Install #

Virt-manager is a UI that helps to create and organize the VM’s. And virt-viewer is used to open remote window into the VM instance.

sudo pacman -S virt-manager virt-viewer

3. Start the Services #

To Autostart the services at boot:

# Enable Auto-Start of the Service
sudo systemctl enable libvirtd.service
# Start the Service Right now
sudo systemctl start libvirtd.service

    In case you wish to see if the libvirtd.service has actually started or not:

    sudo systemctl status libvirtd.service

    Lean Approach

    Note: In order to reduce the system Load you can skip the Autostart part. And do it every time you wish to use the service.

    Just run, whenever you wish to work with KVM:

    # Start the Service Right now
    sudo systemctl start libvirtd.service

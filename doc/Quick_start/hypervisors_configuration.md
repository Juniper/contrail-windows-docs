Contrail Windows deployment has been tested on the following hypervisors:

- VMware,
- KVM,
- Hyper-V.

Each of them requires a few tweaks required to enable nested Hyper-V.

## VMware

Contrail Windows deployment has been successfully tested on VMware ESXi 6.5 hypervisor.
Virtual machines for compute nodes should be configured to support `ESXi 6.5 or later` (VM version 13) and VM secure boot should be disabled.

VMware should expose hardware virtualization to the guest OS.
To configure it:

1. Locate virtual machine in vSphere Web Client.
1. Make sure it is powered off.
1. Right click on it and select `Edit settings` option.
1. In `Virtual hardware`, in `CPU` section, check `Expose hardware assisted virtualization to the guest OS` option.
1. Click `OK`.
1. Power on the virtual machine.

## KVM

Contrail Windows deployment has been successfully tested on KVM virtual machines.
The following configuration is required:

1. Hypervisor configuration:
    - Linux kernel version `>= 4.15`.
        - There were several issues with nested Hyper-V in KVM.
          Please refer to [https://ladipro.wordpress.com/2017/07/21/hyperv-in-kvm-known-issues/](https://ladipro.wordpress.com/2017/07/21/hyperv-in-kvm-known-issues/) for more information.
    - QEMU version `>= 2.11`.
2. VM configuration:
    - virtio NICs should be used.
    - Windows virtio driver version `>= 0.1.141`.
        - downloadable from [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).

## Hyper-V

Contrail Windows deployment has been successfully tested on Hyper-V virtual machines.
The following VM configuration is required:

- Hyper-V Generation 2 VMs must be used.
- Secure boot must be disabled on the VM.

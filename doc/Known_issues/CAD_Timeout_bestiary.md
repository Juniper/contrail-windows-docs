# Contrail Ansible Deployer(CAD) Timeout bestiary

In the Hyper-V installation (the host OS for the compute nodes is Windows OS and the controller is installed as a VM on this host), the Contrail Ansible Deployer might timeout waiting for Controller.
This could happen because the controller(VM) ran out of memory.
In this case, please try increasing the memory of the controller VM to 16 GB. If this does not work, please change it to "dynamic" as shown below.


![Configuring Hyper-V Memory](Hyper-V-Memory.png)

Warning: If the VM takes excessive memory, your host machine will be affected.

### References and further reading

When to use Hyper-V Dynamic Memory versus Runtime Memory Resize:

<https://blogs.technet.microsoft.com/virtualization/2015/05/26/when-to-use-hyper-v-dynamic-memory-versus-runtime-memory-resize/>

Hyper-V Dynamic Memory Overview:

<https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831766(v=ws.11)>

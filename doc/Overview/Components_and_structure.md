# Components and structure

## Components

The following image presents main building blocks of the system and their communication channels:

![Components and structure](tf-windows-structure.png)

## Units of deployment

The following components are deployed on Windows compute node:

- vRouter Agent,
- vRouter (Forwarding Extension for Hyper-V Switch),
- Node manager,
- CNM Plugin for Docker.

The vRouter (Forwarding Extension) runs in kernel space.
All other components run as Windows services.

All components installed on Windows compute node are deployed through two container images:

  - contrail-windows-vrouter (vRouter Agent, vRouter - Forwarding Extension for Hyper-V Switch, Node manager),
  - contrail-windows-cnm-plugin (CNM Plugin for Docker).

The images serve only as a method of distributing installers for the components.

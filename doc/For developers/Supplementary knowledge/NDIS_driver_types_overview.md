# NDIS (Network Driver Interface Specification) driver types overview

## Miniport drivers

- The lowest drivers in NDIS driver stack.
- They manage a network interface card, send and receive data through the NIC.

## Protocol drivers

- The highest drivers in NDIS driver stack.
- The lowest drivers in transport driver stack.
- They allocate packets, copy data from sending application and pass the packets to lower level driver.

## Filter drivers

- Filter drivers are placed between protocol and miniport drivers in the driver stack.
- Multiple filter drivers can be placed one above the other.
- They are transparent to overlaying protocol driver and underlying miniport driver.
- They can be dynamically inserted, removed and reconfigured without stopping the whole driver stack.

Filter drivers can be divided into two categories:

- Monitoring filter drivers - They receive all network request and indications and pass them to other drivers. They cannot modify or originate data. They also cannot modify behavior of the driver stack. May be used for monitoring network requests.

- Modifying filter drivers - These drivers are allowed to modify the driver stack. The type of modification depends on the type of the driver:

    - Capturing extensions - They can monitor packets like Monitoring filter drivers. They also cannot modify or drop packets received from overlaying driver. However, they can originate their own packet traffic and intercept indications for this traffic. They may be used for example for monitoring and sending network statistics.

    - Filtering extensions - In addition to capturing extensions, they can inspect, modify and drop packets.

    - Forwarding extensions - In addition to filtering extensions, they can forward packets and change theirs destination (packets routing). Only one forwarding extension can be enabled for each extensible switch.

## Intermediate drivers

- Miniport drivers see them as a Protocol drivers.
- Protocol drivers see them as a Miniport drivers.
- They translate between different network media.
- May be used for balancing transmission across more than one NIC.

## WFP (Windows Filtering Platform)

- A framework used for creating network filtering applications.
- Allows writing functions that may interact with packet processing at several network layers.
- Mainly used for implementing firewalls, intrusion detection systems, antivirus programs, network monitoring tools, and parental controls.

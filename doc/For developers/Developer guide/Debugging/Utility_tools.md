A set of utility tools (sometimes referred to simply as "Utils") useful for manually injecting rules into the datapath or for introspect and debugging. They communicate directly with the Forwarding Extension. There are split into two categories:

### Basic utils

These are used for basic vRouter introspection, but also for manual insertion of own rules and objects into the Forwarding Extension.

* vif - used for showing and creating virtual interfaces
* nh - used for showing and creating next hops
* rt - used for showing and creating routes
* flow - used for showing flows

The fifth "basic util" is a test util. Since kernel drivers are hard to test without moving them into userspace, Juniper decided to implement a tool that simulates normal usage.

* vtest - runs playback tests against the Forwarding Extension

### Extended utils

These are mainly used for statistics and advanced debugging.

* vxlan
* vrmemstats
* vrfstats
* qosmap
* mpls
* dropstats
* mirror

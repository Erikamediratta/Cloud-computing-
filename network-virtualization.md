Network Flow


Introduction

In a containerized Linux system such as Kubernetes, a network packet appears to travel through many components: physical interfaces, kernel structures, virtual devices, bridges, namespaces, and finally into a container.

However, at the architectural level, there is only one real networking system: the Linux kernel network stack.

All containers, Pods, virtual interfaces, and bridges are software constructs built on top of this single kernel.

To understand container networking correctly, one must trace the exact journey of a packet from the moment it arrives as electrical signals on a physical cable to the moment it is delivered to an application inside a container.

Physical Layer: The Only Hardware

Networking begins in hardware.

A packet arrives at the server through an Ethernet cable (electrical signals) or fiber (light pulses). It reaches the Physical Network Interface Card (NIC), such as eth0 or enp3s0.

The NIC contains receive (RX) rings — circular descriptor buffers mapped in system memory. These buffers are not copied by the CPU. Instead, the NIC uses Direct Memory Access (DMA) to write packet bytes directly into RAM.

Each RX descriptor contains:

A pointer to a memory buffer

Packet length

Status flags

Offload metadata (checksum status, VLAN tags, etc.)

When a frame is received:

The NIC writes packet bytes into RAM via DMA

Updates the RX descriptor

Raises an interrupt (IRQ)

Modern Linux uses NAPI (New API) to reduce interrupt storms. Under heavy load:

One interrupt switches the NIC into polling mode

The kernel polls multiple packets in batches

Reduces CPU overhead and interrupt amplification

At this stage:

No containers exist

No Kubernetes exists

This is raw Linux networking

The only real hardware involved is the physical NIC.

Kernel Entry: From Hardware to Software Object

When the interrupt is raised:

The CPU executes the interrupt handler.

Control transfers to the NIC driver (e.g., e1000, ixgbe, mlx5).

The driver reads packet data from the DMA buffer.

The kernel allocates an sk_buff structure.

The sk_buff (socket buffer) is the fundamental Linux packet abstraction.

It contains:

Pointer to packet data

Length

Protocol type

Interface index

Timestamp

Routing metadata

Checksum state

Socket association

Connection tracking information

The packet is now a software object inside kernel memory.

The driver passes the sk_buff into:

netif_receive_skb()


This is the main entry into the Linux network stack.

From this point forward, everything happens in software.

Three Possible Packet Paths in the Kernel

Once inside the kernel, a packet may follow one of three architectural models.

Packet Processing (PP) – Traditional Path

This is the full Linux network stack path.

The packet travels through:

Netfilter hooks (iptables)

Routing lookup (FIB)

Bridge forwarding (if bridged)

Traffic Control (TC)

Socket delivery

This is the default behavior used by:

Traditional Linux networking

Basic Docker

iptables-based kube-proxy

Complexity: O(n) when many iptables rules exist
Scalability: Degrades as rule count grows

Packet Bypass (PB) – DPDK Model

In this model:

The NIC is bound to a userspace driver (e.g., vfio)

Packets are polled directly from NIC memory

No interrupts

No sk_buff

No kernel TCP/IP stack

Flow:

NIC → DPDK driver → Userspace application


Advantages:

Maximum throughput

Millions of packets per second

Disadvantages:

No kernel networking

No namespaces

No TCP stack unless reimplemented

Not compatible with standard Kubernetes networking

Used in:

Telecom

High-frequency trading

Specialized packet processors

Packet Filter (PF) – eBPF Model

eBPF allows programmable logic inside the kernel.

It does not bypass the kernel.
It extends the kernel.

eBPF programs can attach at multiple hook points:

XDP (earliest, before sk_buff creation)

TC ingress/egress

Netfilter

Socket layer

Capabilities:

Drop packets early

Rewrite destination IP (Service load balancing)

Enforce security policy

Perform NAT

Inspect Layer 7 traffic (HTTP, gRPC)

Performance:

Hash map lookups (O(1))

JIT compiled to native machine code

Nearly DPDK-level speed while remaining in kernel

Used by:

Cilium

Modern high-performance CNIs

Network Namespaces: The Isolation Mechanism

A network namespace is a separate instance of the network stack.

Each namespace contains its own:

Interfaces

IP addresses

Routing table

ARP cache

Netfilter rules

Sockets

Containers run inside their own network namespaces.

This means:

Each container sees its own eth0

Each container believes it has its own network stack

But physically, all namespaces share the same kernel and NIC

Namespaces provide isolation of view, not separate hardware.

Virtual Ethernet Pair (veth)

Containers cannot access the physical NIC directly.

Instead, Linux creates a veth pair.

A veth pair behaves like a virtual Ethernet cable:

One end remains in the host namespace

The other end moves into the container namespace

If a packet enters one end, it immediately appears at the other.

Example:

Host namespace:

veth0

Container namespace:

veth1 (renamed to eth0)

This connects the container’s isolated namespace to the host’s real network stack.

The veth device:

Is not hardware

Does not perform routing

Does not filter packets

Simply transfers frames between namespaces

Software Switching: Linux Bridge and OVS

To allow multiple containers to communicate, the host uses a software switch.

Linux Bridge

A Layer 2 switch inside the kernel.

Functions:

MAC address learning

Frame forwarding

Flooding unknown destinations

Structure:

            br0
         /   |   \
     vethA vethB eth0


Packets arriving on one port are forwarded to the correct port based on MAC address.

Used by:

Docker default networking

Flannel (bridge mode)

Open vSwitch (OVS)

A programmable software switch.

Supports:

OpenFlow rules

SDN integration

Advanced forwarding logic

DPDK acceleration

Used in:

OpenStack

Large cloud environments

CNI: Container Network Interface

Kubernetes does not implement networking directly.

Instead, when a Pod is created, kubelet invokes a CNI plugin.

CNI responsibilities:

Create network namespace

Create veth pair

Move one end into Pod namespace

Attach host end to bridge or routing table

Assign IP address (IPAM)

Configure routes

CNI configures plumbing.

It does not process packets.

Popular CNIs:

Flannel:

VXLAN overlay

Simple

Bridge-based

Limited policy support

Calico:

BGP routing

iptables or eBPF

Strong NetworkPolicy support

Cilium:

Pure eBPF

Replaces kube-proxy

Identity-based policies

L7 visibility

Observability via Hubble

Why eBPF Replaces iptables

iptables limitations:

Linear rule scanning

O(n) per packet

Thousands of rules in large clusters

Performance degradation

IP-based rules break when Pods restart

eBPF improvements:

Hash map lookups (O(1))

Identity-based security (labels, not IPs)

Survives Pod IP changes

Can inspect Layer 7 traffic

JIT compiled

High performance

Cilium leverages eBPF to:

Implement Service load balancing

Replace kube-proxy

Enforce NetworkPolicy

Provide real-time flow visibility

Complete Packet Flow: External → Pod

Consider an external client sending traffic to a Pod.

Physical NIC:

Receives frame

DMA writes to RAM

IRQ raised

Kernel:

Driver reads buffer

Allocates sk_buff

netif_receive_skb()

eBPF (if installed):

Policy check

NAT (Service → Pod IP rewrite)

Allow or drop

Bridge:

MAC lookup

Forwards to correct veth

veth:

Host end receives frame

Frame appears in container namespace

Container:

eth0 receives packet

Container TCP/IP stack processes it

Application receives data via socket

Throughout this process:

Only one kernel exists

Only one physical NIC exists

Namespaces isolate views

veth connects namespaces

Bridge switches frames

CNI configured topology

eBPF enforces policy

sk_buff carries packet metadata

Final Architecture Summary

Physical NIC is the only hardware device.

The Linux kernel:

Owns the network stack

Allocates sk_buff

Performs routing

Executes eBPF programs

Enforces policy

Network namespaces:

Provide isolated views of networking

Do not move packets themselves

veth pairs:

Connect container namespace to host namespace

Bridges/OVS:

Forward frames between virtual interfaces

CNI:

Configures networking

Does not process packets

Packet paths:

PP: Traditional kernel stack

PF: eBPF-extended kernel stack

PB: Userspace bypass (DPDK)

eBPF:

Replaces iptables

Enables O(1) lookups

Supports identity-based security

Enables L7 inspection

Provides observability (Hubble)

All container networking is a structured illusion built on one kernel and one physical NIC.

The packet always returns to the kernel for a decision.

The kernel is the final authority in packet handling.

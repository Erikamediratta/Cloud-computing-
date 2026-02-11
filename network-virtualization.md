To understand container networking and Kubernetes data paths, it is essential to begin at the lowest layer of the networking stack: the physical hardware and the Linux kernel. All higher-level abstractions such as pods, network namespaces, virtual interfaces, bridges, and CNI plugins operate on top of the Linux networking subsystem. Therefore, a clear understanding of how a packet is received and processed by the operating system is fundamental.

Networking fundamentally begins at the Network Interface Card (NIC). The NIC is the physical hardware device responsible for receiving and transmitting network frames. When a packet arrives from the network medium (copper or fiber), it is first handled entirely by hardware before any involvement of container runtimes or orchestration systems.

The following section explains, step by step, how a packet transitions from physical hardware into the Linux kernel networking stack.

At the absolute lowest level, networking begins in hardware.

A packet physically arrives at the NIC as electrical signals (or light if fiber). The NIC hardware has receive (RX) rings — circular buffers in memory. Using DMA (Direct Memory Access), the NIC writes packet bytes directly into RAM without CPU copying. The NIC maintains descriptors pointing to memory buffers.

When a packet is written into an RX buffer, the NIC raises an interrupt (IRQ). Modern systems use NAPI (New API) to reduce interrupt storms. Instead of one interrupt per packet, the kernel switches the NIC into polling mode under load.

The driver allocates an sk_buff structure. This is the fundamental Linux packet object. It contains metadata: protocol, length, checksum state, pointers to headers, routing info, socket association, etc.

At this moment, Kubernetes does not exist. Pods do not exist. This is raw Linux.

Now the packet enters the Linux networking stack.

Before routing, several hook points may trigger.

If XDP is attached, it runs at the driver level before the packet becomes a full sk_buff. XDP can drop, redirect, or pass the packet. XDP is extremely early and very fast because it avoids memory allocation overhead.

If not dropped, the packet becomes an sk_buff and continues upward.

Then the packet hits TC ingress (Traffic Control ingress hook). eBPF programs attached here can inspect and modify the packet. Cilium often attaches here for policy enforcement and service handling.

Now routing decision happens.

The kernel examines destination IP and consults the routing table of the current network namespace. The routing table lookup is done via FIB (Forwarding Information Base) using longest prefix match.

Important: routing table depends on namespace. Each namespace has its own FIB. But the routing engine code is shared. Only the data differs.

If destination is local (an IP assigned to an interface in this namespace), the packet goes to local delivery. If not, it is forwarded.

Now we must understand namespaces properly.

A network namespace is not a miniature kernel. It is a data structure that holds:

Interface list

Routing tables

ARP table

Netfilter tables

Socket bindings

When a process runs inside a pod, its socket is bound to that namespace. So when it sends a packet, the routing lookup happens in that namespace’s routing table.

But physically the packet is still inside the same kernel memory.

When Kubernetes creates a pod, CNI performs:

create netns

create veth pair

move one end into pod netns

assign IP

set default route via veth

configure ARP/neigh entries

Inside the pod, eth0 appears. But this is just the veth endpoint moved into that namespace.

When a container sends data:

Application → libc → socket() → write() → system call → kernel.

The kernel constructs TCP segments. TCP maintains state machine: SYN, ACK, window, congestion control. TCP segmentation happens before IP routing.

Then IP layer wraps TCP segment into IP packet. Then routing lookup occurs in pod namespace.

The default route inside pod typically points to the veth peer (host side). So packet exits via eth0 (veth). That means the sk_buff is queued onto the veth transmit function.

veth driver immediately injects the packet into the peer interface receive path in the host namespace. No physical transmission. It is a direct function call inside kernel.

Now the packet is in host namespace.

This is the key crossing point. This is where illusion meets node reality.

From here onward, all cluster-wide decisions happen.

If Linux bridge is used, the host veth is attached to bridge (like br0 or cni0). Bridge maintains forwarding database (FDB) mapping MAC to port. The bridge looks at destination MAC.

If destination MAC corresponds to another pod’s veth, it forwards internally. If not, it forwards to physical NIC.

Bridge is L2. It does not understand IP routing deeply. It switches frames.

If routing-based CNI is used instead, the host namespace has routes like:

10.244.2.0/24 via nodeB_IP

That means kernel routing will send packet to physical NIC directly without L2 switching domain.

Now service IP handling.

When pod sends packet to a ClusterIP, that IP does not exist as interface. So normally routing would treat it as external.

But with kube-proxy (iptables mode), NAT rules intercept packet in netfilter PREROUTING chain. It rewrites destination IP to one of backend pods.

iptables works by rule chains. It evaluates each rule sequentially until match. In large clusters, thousands of rules mean slower traversal.

With eBPF (Cilium), the service lookup happens earlier at TC or XDP. eBPF uses hash maps keyed by service IP + port. Lookup is O(1). Destination IP is rewritten in-place in packet header. Then routing proceeds normally.

This means Service is implemented as controlled destination NAT.

Now policy enforcement.

NetworkPolicy in Kubernetes is label-based. Cilium assigns identity numbers to pods based on labels. eBPF maps store allowed identity pairs.

When packet passes TC ingress or egress hook, Cilium eBPF program extracts source identity and destination identity, checks map, and decides allow or drop.

This happens inside kernel before routing completes.

Why not bypass?

Because namespace isolation, routing, NAT, and policy enforcement are all kernel functions. If you bypass kernel, none of these run.

DPDK works differently.

In DPDK, NIC is bound to user-space driver (vfio or uio). The kernel network stack is detached. Packets are polled directly by user-space application. That application must implement:

L2 switching

L3 routing

TCP/IP stack (optional)

NAT

firewall

load balancing

If you try to combine DPDK with Kubernetes pod networking, you must reimplement the entire cluster networking logic in user space. That’s why DPDK is used for specialized workloads (NFV, telecom), not general pod networking.

Now pod-to-pod same node.

Packet path:

App A → TCP → IP → veth → host namespace → routing sees destination IP belongs to another local veth → deliver directly → veth → Pod B → TCP → App B.

No physical NIC.
No encapsulation.
No underlay involvement.

Pod-to-pod cross node (routing-based CNI).

App A → veth → host routing → physical NIC → underlay network → node B NIC → routing → veth → Pod B.

Overlay-based (VXLAN).

At host A, packet is encapsulated into UDP VXLAN header. Outer destination = node B IP. Underlay network routes outer packet. At node B, kernel decapsulates and injects inner packet into pod routing path.

Encapsulation adds ~50 bytes overhead. That reduces MTU. If MTU not adjusted, fragmentation or drops happen.

Now why Cilium instead of Flannel?

Flannel primarily provides connectivity. It uses simple forwarding or VXLAN. It does not implement identity-based policy deeply. It relies on iptables for NAT.

Cilium unifies service load balancing, policy, encryption, and observability using eBPF. It reduces dependency on iptables and kube-proxy.

Now deeper into kernel internal flow for local delivery.

After routing determines packet is local:

IP layer hands packet to transport layer (TCP or UDP). TCP checks socket table for matching 4-tuple (src IP, src port, dst IP, dst port). If match found, packet is queued into socket receive buffer.

Then wakeup event triggers process scheduler. The process reads via recv() and gets data into user space.

Reverse happens on send.

Now about why bypass cannot happen “somewhere else”.

Packet must pass through at least one of:

driver receive

routing lookup

transport processing

If you bypass before routing, kernel never sees packet → namespace isolation broken.
If you bypass after routing but before transport, socket matching breaks.
If you bypass at NIC egress, you still needed kernel to decide route.

So complete bypass only works if entire networking stack is replaced.

eBPF does not bypass. It inserts programmable logic at safe hook points.

Linux bridge vs OVS vs routing:

Linux bridge: simple L2 domain.
OVS: programmable L2/L3, supports OpenFlow, can integrate with DPDK.
Routing-based: pure L3, no broadcast domain, more scalable.

Namespaces:

Each namespace has independent:

lo interface

eth0 (veth)

routing table

ARP cache

But actual packet buffer memory is shared in kernel heap.

vNIC:

Not hardware.
Implements net_device struct.
Queues sk_buffs.
Calls peer receive function.

NIC:

Hardware.
Has RX/TX rings.
Performs checksum offload, segmentation offload (TSO/GSO), GRO (Generic Receive Offload).

Important performance detail:

GRO merges packets before passing up stack.
GSO segments packets before NIC transmit.

These optimizations happen inside kernel, invisible to pod.

Final absolute truth model:

Physical NIC receives.
Driver builds sk_buff.
Optional XDP.
TC ingress (policy/service).
Routing lookup.
Bridge or direct routing.
Optional encapsulation.
TC egress.
NIC transmit.

On receive side:

NIC → driver → XDP → TC ingress → routing → veth → namespace → transport → socket → process.

Pods never own hardware.
Namespaces never execute routing code.
CNI never touches live packets.
Only kernel networking stack processes packets.
eBPF modifies kernel decision logic.
DPDK replaces kernel stack entirely.

Everything else is structured illusion built on one Linux kernel networking engine.

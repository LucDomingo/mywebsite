---
title: 'The power of eBPF'
date: 2022-08-15
permalink: /posts/2022/08/power-of-eBPF/
tags:
  - eBPF
  - Linux
  - JIT
  - Python
  - bcc
  - xdp-filter
---

This is a simple introduction to the unlimited power of eBPF

Introduction
======
eBPF (extended Berkeley Packet Filter) is a powerful technology in the Linux kernel that allows for programmable packet filtering, tracing, and processing of network events in a safe and efficient manner.

Why they are so powerful
======
**Safety**: eBPF programs are designed with strong safety guarantees. They run in a restricted environment, providing isolation from the kernel. This ensures that malicious programs cannot crash the system or compromise its security.

**Dynamic**: eBPF programs can be loaded and executed dynamically without requiring modifications to the kernel or rebooting the system. This allows for the flexible addition of new functionality or the modification of existing behavior at runtime.

**Tracing and Observability**: eBPF programs can be attached to various kernel and user-space events, enabling powerful observability and tracing capabilities. They can capture and analyze network packets, system calls, function calls, and other events, providing insights into system behavior, performance analysis, and debugging.

**Packet Filtering**: eBPF can be used to filter and process network packets at different layers of the networking stack. It allows users to define custom filters and actions based on packet headers, payload content, or other attributes. This enables advanced packet filtering, firewalling, and network monitoring capabilities.

**Performance**: eBPF programs are designed to execute with minimal overhead. They are compiled to highly optimized machine code and benefit from just-in-time [(JIT)](#Definitons) compilation and hardware acceleration where available. This makes eBPF well-suited for high-performance networking and tracing scenarios.

Project based on eBPF
======
Here are a few notable projects that utilize eBPF and demonstrate its capabilities:

**Cilium**: Networking and security project that leverages eBPF to provide advanced network visibility, security, and load balancing for containerized environments such as Kubernetes. It uses eBPF programs to enforce network policies, perform load balancing, and collect metrics and telemetry data.

**Falco**: Cloud-native runtime security project that uses eBPF to perform behavior-based anomaly detection and intrusion prevention in containerized environments. It leverages eBPF to capture system calls, network events, and other behaviors, enabling real-time security monitoring and response.

**bcc**: It's a tool that provides a range of utilities and libraries for eBPF program development and observability. It enables users to write and run custom eBPF programs to trace and analyze system events, network activity, function calls, and more. These tools are widely used for troubleshooting, performance analysis, and observability purposes.

It's time, let's practice
======
Let's code an eBPF program for network packet filtering using the bcc framework and Python.
**Prerequisites**
- Linux system with eBPF support (kernel version >= 4.1)
- Python (version 3.x)
- clang compiler (to compile the eBPF program)

**Step 1**: Install Dependencies
Install the necessary dependencies, including the bcc library, which provides tools and libraries for eBPF program development:
<pre>
$ sudo apt-get install bpfcc-tools python3-bpfcc libbpf-dev
</pre>
Or under centos:
<pre>
$ sudo yum install bcc-tools python3-bcc libbpf-devel
</pre>
Step 2: Write the eBPF Program
Create a new file called packet_filter.c and add the following code:
<pre>
&#35;include &lt;linux/bpf.h&gt;
&#35;include &lt;linux/if_ether.h&gt;
&#35;include &lt;linux/ip.h&gt;

// The __sk_buff structure represents a network packet in the Linux kernel
int packet_filter(struct __sk_buff *skb) {
    void *data = (void *)(long)skb->data;
    void *data_end = (void *)(long)skb->data_end;
    
    // These lines define pointers to the Ethernet header (ethhdr) and IP header (iphdr) within the packet. 
    // The bpf_hdr_pointer function is used to obtain a pointer to the packet data.
    struct ethhdr *eth = data;
    struct iphdr *ip = (struct iphdr *)(eth + 1);
    
    // Filter packets with source IP address 192.168.0.1
    if (ip->saddr == htonl(0xC0A80001)) {
        // XDP_DROP instructs the kernel to drop the packet
        return XDP_DROP;
    }
    // XDP_PASS instructs that the packet should be passed through
    return XDP_PASS;
}
</pre>
This eBPF program filters packets based on the source IP address. In this example, it drops packets with a source IP address of 192.168.0.1.

**Step 3**: Compile the eBPF Program
Compile the eBPF program using clang:
<pre>
$ clang -O2 -target bpf -c packet_filter.c -o packet_filter.o
</pre>
This generates the compiled eBPF object file packet_filter.o. This file contains the compiled eBPF bytecode, which is a low-level representation of the eBPF program. This bytecode is platform-independent and can be loaded and executed by the Linux kernel's eBPF virtual machine.

**Step 4**: Write the Python Wrapper
Create a new file called packet_filter.py and add the following code:

<pre>
from bcc import BPF

# Load the eBPF program
bpf = BPF(src_file="packet_filter.c")

# Attach the program to the network interface
bpf.attach_xdp(device="eth0", program=bpf["packet_filter"])

# Run the program
try:
    bpf.trace_print()
except KeyboardInterrupt:
    pass

# Detach the program from the network interface
bpf.remove_xdp(device="eth0")
</pre>
This Python script uses the bcc library to load the eBPF program, attach it to the eth0 network interface, and trace the program's output.

**Step 5**: Run the Program
Execute the Python script with root privileges:

<pre>
$ sudo python3 packet_filter.py
</pre>

Okay, very cool but ...
======
Yes of course, this is a challenging approach to directly code eBPF program and not very industrial-friendly. That's why tool like **xdp-filter** exists.
But concretely, what is the benefits/drawbacks to use a tool like xdp-filter rather than firewalld for example ?

xdp-filter is deliberately simple and so does not have the same matching capabilities as nftables.

eBPF programs can be highly optimized and executed directly in the kernel, resulting in efficient packet processing and **low latency**. This can be particularly beneficial for scenarios where high-performance packet filtering is required.

xdp-filter is a relatively new feature and **may not be supported on all Linux distributions or kernel versions**.

So it's a fairly simple tool that you can practice and learn about by following this [link](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-xdp-filter-for-high-performance-traffic-filtering-to-prevent-ddos-attacks_configuring-and-managing-networking)

Definitons
======

**JIT**:  It involves dynamically translating and optimizing sections of code just before they are executed, as opposed to ahead-of-time (AOT) compilation, where the entire program is compiled before execution.
The process of JIT compilation typically involves the following steps:
- Loading: The program's bytecode or intermediate representation is loaded into memory.

- Analysis: The JIT compiler analyzes the bytecode to gather information about the code's behavior and usage patterns.

- Compilation: Based on the gathered information, the JIT compiler selectively compiles specific sections of code into machine code. This compilation process includes optimizations tailored to the target hardware and execution environment.

- Caching: The compiled machine code is cached for reuse in subsequent executions. This allows the JIT compiler to avoid recompiling the same code multiple times, improving performance.

- Execution: The compiled code is executed by the processor, providing a performance boost compared to interpreting the original bytecode.

JIT compilation is commonly used in programming languages like Java (.class files compiled into Java bytecode) and .NET languages (Common Intermediate Language). So it offers performance (at runtime JIT compilation can apply optimizations), and portability (the same bytecode can be executed on different architectures).

Resources
======
- [https://ebpf.io/]
- [https://github.com/iovisor/bcc]
- [https://lwn.net/Articles/867803/]
- [https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-xdp-filter-for-high-performance-traffic-filtering-to-prevent-ddos-attacks_configuring-and-managing-networking]

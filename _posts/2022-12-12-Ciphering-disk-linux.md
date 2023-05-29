---
title: 'How can I cipher disk on my centos 8 ?'
date: 2023-05-27
permalink: /posts/2023/05/power-of-eBPF/
tags:
  - LUKS
  - Ciphering
  - CentOS 8
---

Everything is in the title, simple question and concise answer in this post

Introduction
======
To cipher disk on CentOS 8, we can use the [LUKS](#Definitons) wich is the standard disk encryption system for Linux (it's widely supported in CentOS and other linux distributions).

Step-by-step guide
======
**Warning** : Before starting, be aware that the encryption process involves modifying disk partitions, whivh can lead to data loss if something goes wrong. It's crucial to have a backup of your important data before proceeding.

First install the necessary packages :
<pre>
$ sudo yum install cryptsetup
</pre>

Indentify the disk to cipher, for example /dev/sda.
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

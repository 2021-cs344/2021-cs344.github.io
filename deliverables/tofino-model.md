---
layout: page
title: Developing and Testing Programs for Tofino
---

# Background

Now that each team has built a working router system and tested it with other teams' implementations at the network level, it's time try to port each team's router program for Tofino, a 6.4Tbps P4-programmable switching ASIC. As you have learned from a lecture, the PISA machine offers unique and powerful benefits compared to software switches or CPU-based P4 targets in general; for those P4 programs that compile successfully, PISA ensures extremely "high" and "predictable" performance in terms of throughput, latency, and power consumption at run time irrespective of any incoming traffic workloads. 

Such benefits, however, do not come for free. The PISA machine trades some amount of flexibility and generality for performance. Hence it supports only a small, limited length of dependency chains in program logic, offers a limited memory-access model, provides a limited set of instructions optimized mainly for packet processing, and equips a limited amount of physical resources to parse packets and carry the parsed representation. Hence, in order to develop useful P4 programs customized for PISA, it's necessary to understand such limitations and adapt your programs appropriately to the limitations.

# Goals

The main goal of this assignment is 1) porting your existing router P4 program for Tofino using [Tofino Native Architecture](https://github.com/barefootnetworks/Open-Tofino/blob/master/PUBLIC_Tofino-Native-Arch-Document.pdf), and 2) testing the program with a simple test framework, called PTF (Packet-level Test Framework). The first part requires modifying your existing P4 program for TNA and compiling it successfully using the Tofino-specific P4 compiler (p4c-tofino). If necessary, you'll have to use a development-assistance tool, called `p4i`, for this. The second part requires writing a simple Python program that uses the APIs derived from your P4 program through the compilation process.

# Preparations

We'll use Intel P4 Studio Lite for this assignment. You can find the original P4 Studio Lite package at `/home/p4dev/shared_folder/install-bf-sde-9.4.0-lite` in the VM you're using for the class. The directory contains `README.md`, which explains how to install the package on a vanilla Ubuntu machine. Note, however, that we have already installed the package in the VM for you; you won't have to repeat the installation process. 

The installation process creates the following three directories in your home directory:

- `bf-sde-9.4.0` - The Intel/Barefoot SDE 9.4.0, lite version.
- `tools` - A few scripts that will be useful.
- `demos` - Some demo P4 programs

Before you try to port your router P4 program for Tofino, make sure you try compiling `demo1_tna.p4`, a sample TNA-based P4 program in the `demos/demo1_tna` directory. Please take a look at `README.md` in that directory, which explains how you can set up a right environment, compile the program, produce a P4 binary file, investigate the compilation results (a.k.a., resource consumption status) using a visualization tool called `p4i`, test the program using the Tofino ASIC model and the control-plane scripts written with the PTF framework, and finally debug the program using `bfshell`, a shell-like interface for the Tofino ASIC.

The real definition of TNA for Tofino1 is available at `bf-sde-9.4.0/install/share/p4c/p4include/tofino1arch.p4`. It'll be useful to take a look at that file and `tofino.p4`, which `tofino1arch.p4` includes.

To know more about Tofino and Intel P4 Studio (including its public API, TNA, terms of license, sample applications, etc.), it may be useful to take a look at [Intel's official github repo](https://github.com/barefootnetworks/Open-Tofino). _**Do NOT share Intel P4 Studio Lite with anyone outside the class or redistribute it.**_. Intel P4 Studio Lite is Intel's product, and hence Intel reserves the sole right to distribute it. CS344 has received a special permission to use it only for the class in 2021.

# The Assignment

Once you complete the steps above, you need to do the followings and submit the results.

1. Port your router P4 program by using `demo1_tna.p4` as a baseline. Create a copy of the whole `demos/demo1_tna` directory under `demos`, and modify the files in the replica directory appropriately. More specifically, port the code in your old V1model-based router P4 program into the replica P4 file. When doing so, it'll be helpful if you move your logic from your old P4 program to the new P4 program in a piecemeal fashion because, by doing so, you'll be able to figure out more easily if, when, and why compilation fails with your new P4 program. You'll have to replace all references to V1model-specific intrinsic metadata with TNA-specific intrinsic metadata. You'll also have to use the new module definitions (i.e., the new signatures of all programmable blocks) for all programmable blocks in TNA.

2. Write PTF test scripts to test the P4 program. Again, use the test cases in `demos/demo1_tna/ptf-tests` as a starting point and modify them appropriately for your P4 program.

3. Enhance your P4 program and test it. You'll need to try both the basic tasks below. The advanced tasks are optional; you'll get extra credits for completing any or both of these.

- [Basic 1] Implement ICMP echo request and reply (type 8 and 0) in the data plane in P4 and test it by augmenting the test scripts. Confirm via `p4i` what kinds of resources this new data-plane capability consumes and by how much.
- [Basic 2] Increase the size of the IP routing table in your router program as much as possible. Report the maximum table size you were able to achieve. Again, confirm via `p4i` what kinds of resources this new data-plane capability consumes and by how much.
- [Advanced 1] Implement and test ECMP (Equal-Cost Multi-Path) forwarding. If a switch has multiple shortest paths and hence multiple best next hops for a destination, an ECMP-capable switch uses all those next hops for traffic to the destination. More specifically, an ECMP-capable switch applies a pseudorandom hash function to the 5-tuple values of each packet (i.e., source and destination IP addresses, source and destination transport port numbers, and the IP protocol number) and use the hash value to select one of the best next hops. This way, the switch can use all the best next hops collectively for all flows, but just one particular next hop deterministically for all packets of a flow, ensuring in-order delivery for each flow. To realize ECMP on Tofino, you'll have to use special TNA externs called `Hash`, `ActionProfile`, and `ActionSelector`. You can find detailed explanations and sample code of those externs in [Tofino Native Architecture](https://github.com/barefootnetworks/Open-Tofino/blob/master/PUBLIC_Tofino-Native-Arch-Document.pdf).
- [Advanced 2] Implement and test a variant of ECMP that uses data-plane state to make a next-hop decision (e.g., per-packet round robin, per-packet weighted round robin). In other words, instead of choosing a next hop for a _flow_ in a _pseudorandom_ fashion, use a new next hop for each new _packet_ in a _predictable_ fashion. Note, this can break in-order delivery for packets of a flow, but that's acceptable in this mode.

# How To Submit

Create a new directory named `my-tofino-router` at the same level as `router.p4app` in your private fork of the `2021-cs344-project` repository. Copy your TNA-based P4 program and associated PTF test files into the directly. Add the `tofino-router` directory and all the files in it to your fork of the repo using `git add`. Finally, commit the change with a message and push it.

If you're doing only the basic tasks of the enhancement above, keep all your enhancement features in one P4 file along with your vanilla IP routing features. You can still have as many test files as you wish. If you're doing the advanced tasks of the enhancement as well, create a separate sub-directory for each advanced program and keep the corresponding P4 program and test files in each sub-directory.

Finally, write an email to the instructors and report the followings:
- Which enhancements you've done.
- The maximum routing table size you achieved (i.e., implemented and tested).

# UDP Network Performance Analysis
 
Empirical network performance study measuring the effect of MTU size and simulated link delay on UDP transmission, RTT, throughput, and fragmentation behavior, paired with a custom C UDP client/server implementation featuring configurable timeouts and dynamic port handling.
 
---
 
## Overview
 
Understanding how network parameters affect transmission performance is foundational to building reliable distributed systems. This project takes an empirical approach: systematically varying MTU and delay, measuring the impact on real packet behavior, and implementing a UDP program in C that handles the edge cases standard implementations paper over (timeouts, server port changes, packet loss recovery).
 
---
 
## Part 1 - Network performance analysis
 
### Variables and measurements
 
| Parameter | Values tested | Metrics captured |
|---|---|---|
| MTU size | 576, 1024, 1500 bytes | RTT, fragmentation rate, throughput |
| Simulated link delay | 0ms, 50ms, 100ms (via `tc netem`) | RTT increase, retransmission count |
| Bandwidth cap | 1 Mbps, 10 Mbps, unlimited | Effective vs. theoretical throughput |
 
### Tools
 
`ip link set mtu` · `tc netem` · `ping` · `iperf3` · `Wireshark`
 
### Key findings
 
- MTU reduction from 1500 → 576 bytes increased RTT by ~18% on loopback due to fragmentation overhead
- `tc netem` 100ms delay produced RTT variance of ±8ms, consistent with queuing theory predictions at that delay magnitude
- Effective throughput under 1 Mbps cap reached ~94% of theoretical limit; the gap is attributable to UDP header overhead and OS socket buffer flushing
---
 
## Part 2 - Custom UDP client/server (C)
 
### What was implemented
 
A UDP client/server pair in C with:
 
- **Configurable per-socket timeout** via `setsockopt(SO_RCVTIMEO)` - eliminates indefinite blocking on packet loss
- **Dynamic server port handling** - client detects and updates its target port when the server signals a redirect, without dropping the session
- **Reliable message transmission loop** - timeout-triggered retry with configurable max attempts
- **Graceful error handling** for packet loss, timeout, and unexpected server disconnect
### How to run
 
```bash
# Compile
gcc -o server server.c
gcc -o client client.c
 
# Run server (in one terminal)
./server <port>
 
# Run client (in another terminal)
./client <server_ip> <port> <timeout_ms>
```
 
### Example
 
```bash
./server 8080
./client 127.0.0.1 8080 500    # 500ms timeout
```
 
---
 
## Repository structure
 
```
udp-network-performance/
├── server.c          # UDP server with dynamic port signaling
├── client.c          # UDP client with timeout + retry logic
├── analysis/
│   └── results.md    # RTT/throughput measurements and analysis
└── README.md
```
 
---
 
## Skills demonstrated
 
- Linux networking tools: MTU configuration, traffic shaping (`tc netem`), packet capture (Wireshark)
- Low-level C socket programming: UDP, `setsockopt`, timeout handling
- Empirical performance measurement: controlled variable testing, RTT/throughput analysis
- Network protocol analysis: fragmentation behavior, queuing delay, retransmission
## Context
 
Completed April 2024 at the University of Kentucky as part of graduate Computer Networks coursework. Focus was on empirical measurement methodology and low-level socket programming, not framework abstractions.

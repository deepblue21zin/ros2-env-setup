# ROS2 Humble on Ubuntu 22.04 (Multipass on macOS) - Reproducible Setup

## Goal
- Build a reproducible ROS2 Humble dev environment on Ubuntu 22.04 VM
- Verify pub/sub and run a basic QoS experiment under CPU load

## Environment
- Host: macOS 13.7
- VM: Multipass 1.16.1, Ubuntu 22.04 (jammy)
- Kernel: 5.15.0-171-generic
- Shell: zsh

## Setup (repro steps)
1) Create VM
```bash
multipass launch 22.04 -n ubuntu2204 -c 4 -m 8G -d 60G

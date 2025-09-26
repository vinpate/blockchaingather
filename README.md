# blockchain-gather

A comprehensive diagnostic tool for blockchain node systems that collects system information, performance metrics, and configuration details to help troubleshoot node issues.

Tested on Ubuntu 22.04+ systems.

## Quick Start

```bash
wget -N https://raw.githubusercontent.com/nirvana-labs/nodes-blockchain-gather/main/blockchaingather
sudo sh ./blockchaingather
```

## What information is collected

The script generates a compressed archive containing detailed system diagnostics and blockchain-specific metrics. The output includes:

- Active blockchain node processes (op-reth, op-node, geth, erigon, parity, besu, nethermind, bitcoin, etc.)
- System commands output: dmesg, netstat, ip, iptables, sysctl, free, vmstat, df, mount, lsb_release, uname with various diagnostic options
- Docker container status and resource usage
- Network port scanning and service identification
- Storage device analysis including NVMe optimization checks
- CPU performance metrics and power management settings
- Memory utilization and process resource consumption
- ZFS filesystem diagnostics (when available)

This collection is designed to provide a complete picture of your blockchain node's system state and performance characteristics.

The data helps identify potential issues such as network bottlenecks, storage misconfigurations, resource constraints, or suboptimal system settings that could impact blockchain node performance.

## Analyzing the results

The collected data requires manual analysis. If there was an easy way to interpret the data I would have published `fixyourblockchain.sh` ;)

Focus on the generated SUMMARY.txt and examine system logs for performance bottlenecks, resource constraints, and configuration problems that may affect your blockchain node's operation.

## Contributing

Contributions and improvements are welcome.

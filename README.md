# blockchain-gather

`blockchaingather` is a single POSIX shell script (`/bin/sh`) that collects system diagnostics from a blockchain node host into a compressed tarball. It is designed to run non-invasively on production nodes and requires root privileges.

Primary platforms
Tested: Debian-based systems (including Ubuntu 22.04+ systems)
Untested: RHEL-based systems (including CentOS etc.)

## Quick Start

```bash
wget -N https://raw.githubusercontent.com/vinpate/blockchaingather/main/blockchaingather
sudo sh ./blockchaingather
```

## What blockchaingather gathers

blockchaingather gathers various system diagnostics, blockchain-specific metrics, and facts about your system. The output generates a tar.gz archive file.

The tarball contains sets of paginated text files for each probe/test. How you interpret & what you do with the output is up to you.

- Active blockchain node processes (op-reth, op-node, geth, erigon, parity, besu, nethermind, bitcoin, etc.)
- System commands output: dmesg, netstat, ip, iptables, sysctl, free, vmstat, df, mount, lsb_release, uname with various diagnostic options
- Docker status and container identification & resource usage
- Network port scanning and service identification
- NVMe optimization checks
- CPU performance metrics and power management setting validation
- Memory utilization and process resource consumption
- ZFS filesystem diagnostics (when available)

The data helps identify potential issues such as network bottlenecks, storage misconfigurations, resource constraints, or suboptimal system settings that could impact blockchain node performance.

## Analyzing the results

The collected data requires manual analysis. If there was an easy way to interpret the data I would have published `fixyourblockchain.sh` ;)

Start with the generated SUMMARY.txt and examine relevant test files for granular analysis.

## Contributing

Contributions and improvements are welcome.

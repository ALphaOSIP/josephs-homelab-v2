# OPNsense Configuration Setup Log

## Installation Summary

**Date**: July 2025  
**Duration**: ~4 hours (including troubleshooting)  
**Result**: ✅ Fully operational enterprise router

## Hardware Preparation

### Phase 1: Initial PCIe Attempt
```bash
# Issue discovered: TP-Link TX201 not compatible
ifconfig -a
# Result: only em0 visible, PCIe card shows as "none4"

pciconf -lv | grep -i network
# Card detected but no driver available
```

### Phase 2: USB Solution Implementation
```bash
# j5create JUE130 connected
ifconfig -a
# Result: em0 (Intel) + ue0 (j5create) both detected
```

## OPNsense Installation

### Boot and Install Process
1. **Boot from Ventoy USB** → Select OPNsense ISO
2. **Login**: installer / opnsense
3. **Installation**: Guided installation, use entire disk
4. **Admin Password**: Set secure password
5. **Reboot**: Remove USB and boot to console

### Interface Assignment
```bash
# Console Menu → Option 1 (Assign Interfaces)
# WAN: em0 (Intel built-in)
# LAN: ue0 (j5create USB adapter)
# No VLANs configured initially
```

### IP Configuration

**WAN Interface (em0):**
```bash
# Option 2 → Interface 1 (WAN)
# IPv4 DHCP: Yes (gets IP from Spectrum router)
# IPv6 DHCP: No
# Result: 192.168.1.158/24
```

**LAN Interface (ue0):**
```bash
# Option 2 → Interface 2 (LAN)  
# IPv4 DHCP: No (static configuration)
# IPv4 Address: 192.168.100.1
# Subnet: 24
# Upstream Gateway: <ENTER> (none for LAN)
# IPv6: <ENTER> (skip)
# DHCP Server: Yes
# DHCP Range: 192.168.100.10 - 192.168.100.50
```

### Service Startup Verification
```
Starting Unbound DNS...done.
Configuring firewall...done.
Starting DHCPv6 service...done.
Starting router advertisement service...done.
Starting Unbound DNS...done.
Starting web GUI...done.
```

## Network Testing

### Connectivity Verification
```bash
# From OPNsense console
ping 8.8.8.8
# ✅ Internet connectivity confirmed

# From client laptop
ipconfig
# IP: 192.168.100.11 (DHCP assigned)
# Gateway: 192.168.100.1
# DNS: 192.168.100.1

ping google.com
# ✅ Internet access through OPNsense confirmed
```

### Web Interface Access
```
URL: https://192.168.100.1
Login: root / [password]
Status: ✅ Full web interface access
```

## Final Configuration Status

### Interface Summary
| Interface | Hardware | IP Address | Status | Purpose |
|-----------|----------|------------|--------|---------|
| em0 (WAN) | Intel Built-in | 192.168.1.158/24 | ✅ Up | Internet Connection |
| ue0 (LAN) | j5create JUE130 | 192.168.100.1/24 | ✅ Up | Local Network |

### Service Status
- **Firewall**: ✅ Active with default rules
- **DHCP Server**: ✅ Active (192.168.100.10-50)
- **DNS Resolver**: ✅ Unbound running
- **Web Interface**: ✅ HTTPS on port 443
- **SSH Access**: ✅ Available (disabled by default)

### Network Performance
- **WAN Throughput**: 1 Gbps (limited by Spectrum connection)
- **LAN Throughput**: 800+ Mbps (j5create adapter capability)
- **Latency**: <1ms additional routing overhead
- **Concurrent Connections**: Tested with multiple devices

## Troubleshooting Notes

### Common Commands Used
```bash
# Interface status
ifconfig -a

# Check running services
service -e

# View system logs
clog -f /var/log/system.log

# Network diagnostics
ping <target>
traceroute <target>

# Firewall state table
pfctl -s states
```

### Issues Resolved
1. **PCIe Compatibility**: Switched to USB adapter solution
2. **DHCP Configuration**: Properly configured LAN DHCP server
3. **Firewall Rules**: Verified default allow LAN→WAN rule active
4. **DNS Resolution**: Confirmed Unbound resolver working correctly

## Security Configuration

### Default Security Posture
- **WAN Access**: Blocked (default deny)
- **LAN Access**: Allow to WAN (default rule)
- **Management**: HTTPS only (port 443)
- **SSH**: Disabled (console access only)

### Immediate Security Tasks (Future)
- [ ] Change default SSH keys
- [ ] Configure fail2ban equivalent
- [ ] Set up log monitoring
- [ ] Implement backup procedures

## Next Phase Preparation

### Ready for Access Point Integration
- **VLAN Capability**: ✅ Available in OPNsense
- **PoE Planning**: External injector identified
- **Switch Integration**: TL-SG108E ready for connection

### Monitoring Preparation
- **SNMP**: Available for enabling
- **Logging**: Centralized log collection ready
- **Performance Metrics**: Built-in monitoring capabilities

---

**Configuration Completed**: July 2025  
**Status**: ✅ Production Ready  
**Next Review**: When Phase 2 (Access Point) implemented
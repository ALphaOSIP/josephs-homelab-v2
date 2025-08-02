# Network Architecture Documentation

## Current Network Topology

### Phase 1: Dual-Router Configuration (Current)

```
Internet
   │
   │ Fiber (1 Gbps)
   │
┌─────────────────┐
│ Spectrum Router │ 192.168.1.1/24
│  (SAX2V1S)      │ DHCP: .2-.254
└─────────────────┘
   │
   │ Ethernet
   │
┌─────────────────┐
│   OPNsense      │ WAN: 192.168.1.158/24
│   Router        │ LAN: 192.168.100.1/24
│  (OptiPlex 750) │ DHCP: .10-.50
└─────────────────┘
   │
   │ USB 3.0 Ethernet (j5create)
   │
┌─────────────────┐
│  Test Devices   │ 192.168.100.0/24
│   & Future      │
│ Infrastructure  │
└─────────────────┘
```

### Network Segments

**Spectrum Network (192.168.1.0/24):**
- **Gateway**: 192.168.1.1 (Spectrum SAX2V1S)
- **DHCP Range**: 192.168.1.2 - 192.168.1.254
- **Current Devices**: 
  - Raspberry Pi 5 NAS (192.168.1.100)
  - Family WiFi devices
  - Xbox gaming console
  - IoT devices (Eufy cameras, HP printer)

**OPNsense Network (192.168.100.0/24):**
- **Gateway**: 192.168.100.1 (OPNsense LAN)
- **DHCP Range**: 192.168.100.10 - 192.168.100.50
- **Purpose**: New infrastructure, testing, development
- **Future**: Enterprise services, monitoring, VPN

## Interface Configuration

### OPNsense Router Interfaces

**WAN Interface (em0):**
- **Hardware**: Intel built-in ethernet
- **Configuration**: DHCP client (receives IP from Spectrum)
- **Current IP**: 192.168.1.158/24
- **Gateway**: 192.168.1.1
- **Purpose**: Internet connectivity

**LAN Interface (ue0):**
- **Hardware**: j5create JUE130 (ASIX AX88179 chipset)
- **Configuration**: Static IP with DHCP server
- **IP Address**: 192.168.100.1/24
- **DHCP Range**: 192.168.100.10-50
- **Performance**: 800+ Mbps throughput

## Planned Architecture Evolution

### Phase 2: Access Point Integration (Q3 2025)

```
Internet → Modem → OPNsense → Managed Switch → Wired Devices
                      │             │
                      │             └─ Access Point → WiFi Devices
                      │
                   Spectrum Router (retired/bridge mode)
```

**Key Changes:**
- Direct modem connection (eliminate double NAT)
- Enterprise WiFi via Ubiquiti U7 Lite
- VLAN segmentation implementation
- Unified network under OPNsense control

### Phase 3: Advanced Services (Q4 2025)

**Planned VLANs:**
- **VLAN 1**: Management (192.168.1.0/24)
- **VLAN 10**: Infrastructure (192.168.10.0/24)
- **VLAN 20**: IoT Devices (192.168.20.0/24)
- **VLAN 30**: Guest Network (192.168.30.0/24)

**Advanced Services:**
- Monitoring (Prometheus + Grafana)
- VPN Server (WireGuard)
- Network Access Control
- Traffic Analysis and QoS

## Security Architecture

### Current Security Posture

**Firewall Rules (OPNsense):**
- **LAN → WAN**: Allow all (default)
- **WAN → LAN**: Block all (default)
- **Management**: HTTPS web interface (192.168.100.1:443)

**Network Isolation:**
- **Physical Separation**: Different IP ranges
- **Traffic Control**: Routing through OPNsense firewall
- **Access Control**: Console and web-based management

### Planned Security Enhancements

**Advanced Firewall Policies:**
- VLAN-based access controls
- Application-level filtering
- Intrusion detection/prevention
- Traffic shaping and QoS

**Monitoring and Logging:**
- Centralized log collection
- Network traffic analysis
- Security event correlation
- Performance monitoring

## Performance Characteristics

### Current Performance Metrics

**WAN Performance:**
- **Internet Speed**: 1 Gbps down / 35 Mbps up (Spectrum)
- **Router Processing**: CPU utilization <5% under normal load
- **Memory Usage**: 2GB available (16GB total system RAM)

**LAN Performance:**
- **USB Ethernet**: 800+ Mbps throughput (j5create AX88179)
- **Firewall Throughput**: 900+ Mbps (OPNsense capability)
- **Latency**: <1ms additional routing delay

**Scalability Planning:**
- **Concurrent Connections**: 10,000+ supported
- **VLAN Support**: 4,094 VLANs available
- **Bandwidth**: Scales to multi-gigabit with interface upgrades

### Future Performance Targets

**Phase 2 Goals:**
- **WiFi Performance**: 800+ Mbps (WiFi 7 capability)
- **VLAN Throughput**: Inter-VLAN routing at line speed
- **Total Device Count**: 50+ devices across VLANs

**Phase 3 Goals:**
- **Monitoring Overhead**: <2% performance impact
- **VPN Throughput**: 500+ Mbps encrypted
- **High Availability**: 99.9%+ uptime target

## Design Decisions and Rationale

### Hardware Platform Selection

**Dell OptiPlex 750:**
- **Pros**: Enterprise hardware, reliable, low power consumption
- **Cons**: Limited PCIe expansion, older generation
- **Rationale**: Proven platform with adequate performance for requirements

**Dual Interface Strategy:**
- **Built-in Intel**: Proven FreeBSD compatibility
- **USB 3.0 Addition**: High performance, guaranteed compatibility
- **Alternative**: PCIe cards have compatibility risks with FreeBSD

### Software Platform Selection

**OPNsense vs pfSense:**
- **OPNsense Advantages**: Modern UI, frequent updates, better documentation
- **Performance**: Equivalent for home/small business use
- **Learning Value**: Industry-relevant enterprise firewall experience

### Network Design Philosophy

**Incremental Deployment:**
- **Phase 1**: Parallel operation (minimize risk)
- **Phase 2**: Gradual migration (WiFi integration)
- **Phase 3**: Advanced features (monitoring, automation)

**Professional Standards:**
- **Documentation**: Enterprise-grade documentation practices
- **Version Control**: Configuration management with Git
- **Monitoring**: Observability from day one

---

**Design Review**: Quarterly architecture review and optimization  
**Performance Monitoring**: Continuous network performance tracking  
**Security Posture**: Monthly security configuration review
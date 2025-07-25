# Joseph's Homelab V2 - Setup Log

## Project Goals
- Build enterprise-grade home infrastructure using Kubernetes
- Demonstrate DevOps skills for career transition
- Document everything for portfolio showcase
- Create practical home services (NAS, automation, monitoring)

## Phase 1: Foundation Setup

### Date: [7/24]
**Objective**: Set up Pi 5 as Kubernetes control plane

#### Hardware
- **Primary Device**: Raspberry Pi 5 (8GB RAM)
- **Storage**: 256GB microSD card  
- **Network**: Connected via TL-SG108E managed switch
- **Development Environment**: ThinkStation P16 Gen 3 (Ubuntu 22.04)

#### Software Decisions
- **OS**: Ubuntu Server 22.04 LTS (64-bit)
  - Reasoning: Industry standard, enterprise compatibility, package management
- **Container Runtime**: Docker CE + containerd
- **Orchestration**: K3s (lightweight Kubernetes)
- **Infrastructure as Code**: Terraform + Ansible

#### Security Setup
- SSH key-based authentication (ed25519)
- No password authentication 
- Firewall configuration (ufw)
- Regular security updates automated

#### Network Configuration  
- Static IP assignment: 192.168.1.234 (to be confirmed)
- DNS: 1.1.1.1, 8.8.8.8
- SSH port: 22 (will change to custom port later)

---

## Setup Steps

### Step 1: SSH Key Generation ✅
```bash
ssh-keygen -t ed25519 -C "joseph-homelab-v2" -f ~/.ssh/homelab_ed25519
```
**Public Key**: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPa5qZiDt2+wFz8xFzExO43FIjR0Qn19SFIe9FC/FESo joseph-homelab-v2

### Step 2: SD Card Preparation
- [ ] Download Ubuntu Server 24.04.2 LTS 
- [ ] Flash with Raspberry Pi Imager
- [ ] Configure advanced settings (SSH, user, WiFi)
- [ ] First boot and SSH connection

### Step 3: Initial Pi Configuration
- [ ] System updates
- [ ] Static IP configuration  
- [ ] SSH hardening
- [ ] Docker installation
- [ ] Basic monitoring setup

---

## Issues Encountered
1. Had to install nmap to scan my network to locate the ip of the pi
2. still working on a way to get nmap working on my vs code terminal running powershell

## Next Steps
1. Complete Pi 5 initial setup
2. Begin Pi 3 setup as worker node
3. Network VLAN configuration
4. K3s cluster deployment

### Step 2: First Boot SUCCESS ✅
- **Boot time**: ~4 minutes
- **IP Address**: 192.168.1.234
- **SSH Connection**: Working perfectly
- **Username**: alpha
- **OS**: Ubuntu 24.04.2 LTS (newer than planned - good!)
- **Available Storage**: 234GB / 256GB
- **System Health**: Excellent (3% memory, 63°C temp)


### Configuration Notes
- **Hostname**: alphapi5
- **Username**: alpha (intentional choice)
- **System Updates**: 221 updates applied (including 117 security updates)


### Step 6: Static IP Configuration ✅
- **New IP**: 192.168.1.100 (was 192.168.1.234)
- **Hostname**: alphapi5.alpha.home
- **Domain**: alpha.home
- **Gateway**: 192.168.1.1
- **DNS**: 1.1.1.1, 8.8.8.8


### Issue Encountered: Network Configuration
- **Problem**: Lost SSH connection after netplan changes
- **Cause**: Used deprecated `gateway4` syntax instead of `routes`
- **Learning**: Always test network changes carefully in production
- **Resolution**: My initial connection was based on the 234 connection, since it's now static to .100 I just had to change what I used to connect to it.


### Step 10-12: Security & Container Runtime ✅
- **Firewall**: UFW enabled with Kubernetes ports open
- **Docker**: CE version installed, user added to docker group
- **Security**: No sudo required for docker commands
- **Optimization**: Memory cgroups enabled for Kubernetes
- **Status**: alphapi5 ready for container orchestration
---

## Portfolio Notes
**Skills Demonstrated**:
- Linux system administration
- Infrastructure automation 
- Security best practices
- Documentation and project management
- Network design and implementation
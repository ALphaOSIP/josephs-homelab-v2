# Joseph's Homelab V2 - Setup Log

## Project Goals
- Build enterprise-grade home infrastructure using Kubernetes
- Demonstrate DevOps skills for career transition
- Document everything for portfolio showcase
- Create practical home services (NAS, automation, monitoring)

## System Architecture
- **Cluster**: Single-node K3s (Kubernetes 1.32.6)
- **Hardware**: Raspberry Pi 5 (8GB RAM, 256GB storage, 1TB USB SSD)
- **Network**: Static IP (192.168.1.100), UFW firewall, ingress routing
- **Storage**: HostPath volumes, persistent USB mount (/mnt/nextcloud-data)
- **Services**: 4 applications (Samba, FileBrowser, Nginx Ingress, Nextcloud)

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


## Date 7/25/25
### Step 13-16: Kubernetes Cluster LIVE ✅
- **K3s Version**: v1.32.6+k3s1 (latest!)
- **Cluster Status**: Ready, control-plane operational
- **System Pods**: CoreDNS, metrics-server, local-path-provisioner (all running)
- **kubectl**: Configured for alpha user access
- **Test Deployment**: nginx pod created/deleted successfully
- **Helm**: Installed for application package management
- **Ingress**: Nginx Ingress Controller deployed
- **Status**: Production-ready Kubernetes cluster operational!

**Skills Demonstrated:**
- Kubernetes cluster administration
- Container orchestration
- Package management with Helm
- Application deployment and lifecycle management


## Date 7/26/25
### FINAL SUCCESS: Complete File Sharing Infrastructure ✅

**Issue Resolution:**
- **Root Cause**: UFW firewall blocking SMB ports 445/139
- **Solution**: `sudo ufw allow 445/tcp` and `sudo ufw allow 139/tcp`
- **Result**: Full Windows SMB connectivity restored

**Operational Services:**
1. **Windows Network Drive**: \\192.168.1.100\shared (Z: drive)
2. **FileBrowser Web Interface**: http://192.168.1.100:30880
3. **Kubernetes Cluster**: 4 applications running (Samba, FileBrowser, HTTP server, Nextcloud)

**MISSION ACCOMPLISHED**: OpenMediaVault functionality fully replaced with professional Kubernetes infrastructure!


## System Performance
- **Resource Usage**: 11% memory, 63°C temperature under load
- **Storage**: 960GB available, ext4 filesystem
- **Network**: Gigabit ethernet, low latency local access

## Security Configuration
- **Firewall Rules**: SSH (22), K8s API (6443), SMB (445,139), HTTP (80,443)
- **Authentication**: Ed25519 SSH keys, no password auth
- **Container Security**: Non-root containers, resource limits
- **Network Isolation**: Service-based networking, ingress controls

## Project Value
- **Cost Savings**: Replaced commercial NAS solution ($200-500)
- **Learning ROI**: $13k-68k salary increase potential through K8s skills
- **Infrastructure Scale**: Supports 2-user household, expandable to Pi cluster
- **Maintenance**: Self-healing containers, automated restarts

## Professional Skills Demonstrated
**Infrastructure as Code**: Kubernetes YAML manifests, Helm charts
**Container Orchestration**: Pod lifecycle, services, ingress, persistent volumes  
**Network Engineering**: Static IP, firewall rules, port forwarding, DNS
**Storage Management**: Persistent volumes, filesystem permissions, USB mounting
**Security Implementation**: SSH keys, firewall configuration, service isolation
**Systematic Troubleshooting**: Root cause analysis, methodical debugging
**Documentation**: Professional project tracking, version control (Git)


## Phase 2 Planning
- **Pi 3 Worker Node**: Multi-node cluster expansion
- **Monitoring Stack**: Prometheus + Grafana deployment
- **Backup Strategy**: Automated USB backup jobs
- **CI/CD Pipeline**: GitOps workflow for application updates
- **Home Automation**: Integration with existing IoT devices
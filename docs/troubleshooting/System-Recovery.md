# Joseph's Homelab V2 - Disaster Recovery Procedures

## 🎯 Executive Summary

This document outlines comprehensive disaster recovery procedures for Joseph's Homelab V2 Kubernetes infrastructure. These procedures ensure rapid restoration of critical file sharing services with minimal data loss and downtime.

**Recovery Time Objectives (RTO)**: 2-4 hours for full system restoration  
**Recovery Point Objectives (RPO)**: <24 hours data loss maximum  
**Business Impact**: Family file sharing, home automation, development environment

---

## 🚨 Failure Scenarios & Response Matrix

| Scenario | Probability | Impact | RTO | RPO | Priority |
|----------|-------------|--------|-----|-----|----------|
| **SD Card Corruption** | High | Critical | 2 hours | 0 hours | P1 |
| **USB SSD Failure** | Medium | Critical | 4 hours | 24 hours | P1 |
| **Pi Hardware Failure** | Medium | High | 6 hours | 0 hours | P2 |
| **Network Connectivity Loss** | Low | Medium | 1 hour | 0 hours | P3 |
| **Container Service Failure** | Low | Low | 30 mins | 0 hours | P4 |
| **Power Outage** | Low | Medium | 15 mins | 0 hours | P4 |

---

## 🔧 Recovery Procedure Library

### Scenario 1: SD Card Corruption/Failure (P1)

**Symptoms:**
- Pi won't boot or frequent kernel panics
- File system errors during operation
- Corrupted system files

**Recovery Steps:**

#### Phase 1: Immediate Response (0-30 minutes)
```bash
# 1. Power down the Pi safely
sudo shutdown -h now

# 2. Remove SD card and test on development machine
# Insert into P16 Gen 3 and check for corruption
sudo fsck /dev/sdX1

# 3. If corrupted beyond repair, proceed to rebuild
```

#### Phase 2: System Rebuild (30 minutes - 2 hours)
```bash
# 1. Flash new SD card with Ubuntu Server 24.04.2 LTS
# Use Raspberry Pi Imager with same configuration:
# - Username: alpha
# - SSH key: homelab_ed25519.pub
# - WiFi: home network credentials
# - Hostname: alphapi5

# 2. First boot configuration
ssh -i ~/.ssh/homelab_ed25519 alpha@192.168.1.100

# 3. Restore system configuration
sudo apt update && sudo apt upgrade -y

# 4. Set static IP
sudo nano /etc/netplan/50-cloud-init.yaml
# Apply same network configuration as documented

# 5. Install Docker and K3s
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker alpha

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik --disable servicelb --write-kubeconfig-mode 644" sh -

# 6. Configure kubectl access
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown alpha:alpha ~/.kube/config

# 7. Configure firewall
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 445/tcp
sudo ufw allow 139/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 8. Mount USB storage
sudo mkdir -p /mnt/nextcloud-data
sudo mount /dev/sda1 /mnt/nextcloud-data
echo "/dev/sda1 /mnt/nextcloud-data ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

#### Phase 3: Application Restoration (1-2 hours)
```bash
# 1. Clone infrastructure repository
git clone https://github.com/ALphaOSIP/josephs-homelab-v2.git
cd josephs-homelab-v2

# 2. Deploy applications from saved manifests
kubectl create namespace home-services

# 3. Deploy Helm and Nginx Ingress
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt update && sudo apt install helm

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=NodePort

# 4. Deploy file sharing services
kubectl apply -f infrastructure/kubernetes/samba-server.yaml
kubectl apply -f infrastructure/kubernetes/filebrowser.yaml
kubectl apply -f infrastructure/kubernetes/http-files.yaml

# 5. Verify all services operational
kubectl get pods -A
kubectl get svc -A
```

**Validation Steps:**
- [ ] All pods running and healthy
- [ ] Windows SMB drive mapping functional
- [ ] FileBrowser web interface accessible
- [ ] All user data intact on USB drive

---

### Scenario 2: USB SSD Failure (P1)

**Symptoms:**
- I/O errors when accessing /mnt/nextcloud-data
- USB device not detected (lsblk shows no /dev/sda1)
- File corruption in user data

**Recovery Steps:**

#### Phase 1: Immediate Assessment (0-15 minutes)
```bash
# 1. Check USB connection and power
lsblk
dmesg | tail -20

# 2. Attempt remount
sudo umount /mnt/nextcloud-data
sudo mount /dev/sda1 /mnt/nextcloud-data

# 3. Check filesystem integrity
sudo fsck /dev/sda1

# 4. If hardware failure confirmed, proceed to data recovery
```

#### Phase 2: Data Recovery (15 minutes - 2 hours)
```bash
# Option A: If drive partially readable
sudo ddrescue /dev/sda1 /tmp/recovery.img /tmp/recovery.log
sudo mount -o loop /tmp/recovery.img /mnt/recovery
cp -r /mnt/recovery/* /path/to/new/drive/

# Option B: Restore from backup (when implemented)
# [Future enhancement - implement automated backups]

# Option C: Accept data loss and rebuild
# Create new filesystem on replacement drive
sudo mkfs.ext4 /dev/sda1
sudo mount /dev/sda1 /mnt/nextcloud-data
sudo chown -R 33:33 /mnt/nextcloud-data
```

#### Phase 3: Service Restoration (30 minutes - 1 hour)
```bash
# 1. Restart all file sharing services
kubectl delete pod -l app=samba-server -n home-services
kubectl delete pod -l app=filebrowser -n home-services

# 2. Verify mount points in containers
kubectl exec -it deployment/samba-server -n home-services -- ls -la /shared
kubectl exec -it deployment/filebrowser -n home-services -- ls -la /srv

# 3. Test file operations
# Create test file via SMB
# Verify via web interface
# Confirm persistence across container restarts
```

**Data Recovery Priority:**
1. **vault/** folder (41GB family data)
2. **iPhone/** folder (photo backups)
3. **admin/** folder (user configurations)
4. System files (recreatable)

---

### Scenario 3: Raspberry Pi Hardware Failure (P2)

**Symptoms:**
- Pi won't power on or boot
- Random crashes and instability
- Hardware error messages in logs

**Recovery Options:**

#### Option A: Replace with Identical Hardware
```bash
# 1. Obtain replacement Raspberry Pi 5 (8GB)
# 2. Transfer SD card to new hardware
# 3. Connect USB SSD to new Pi
# 4. Verify MAC address change doesn't affect DHCP
# 5. Update documentation with new hardware serial
```

#### Option B: Migrate to Different Platform
```bash
# Migration to Intel NUC or similar x86 hardware

# 1. Install Ubuntu Server 24.04.2 LTS on new hardware
# 2. Follow same K3s installation procedure
# 3. Restore applications from Git repository
# 4. Migrate USB drive to new system
# 5. Update network configuration for new device
```

#### Option C: Temporary Cloud Migration
```bash
# Emergency cloud deployment for critical services

# 1. Deploy FileBrowser to cloud instance
# 2. Upload critical data from USB backup
# 3. Provide temporary access to family
# 4. Plan hardware replacement strategy
```

---

### Scenario 4: Network Connectivity Loss (P3)

**Symptoms:**
- Cannot SSH to Pi
- Web services inaccessible from network
- Pi can access internet but not local network

**Recovery Steps:**

#### Phase 1: Network Diagnostics (0-15 minutes)
```bash
# From Pi console (direct access)
ip addr show
ping 192.168.1.1  # Gateway
ping 8.8.8.8      # Internet
netstat -rn       # Routing table

# Check network configuration
cat /etc/netplan/50-cloud-init.yaml
sudo netplan apply
```

#### Phase 2: Network Restoration (15-60 minutes)
```bash
# 1. Reset to DHCP temporarily
sudo nano /etc/netplan/50-cloud-init.yaml
# Comment out static configuration
# Enable DHCP
sudo netplan apply

# 2. Identify new IP and update documentation
ip addr show eth0

# 3. Restore static IP configuration
# Update netplan with correct static IP
# Verify gateway and DNS settings
# Test connectivity

# 4. Update firewall rules if needed
sudo ufw status
sudo ufw reload
```

---

### Scenario 5: Container Service Failure (P4)

**Symptoms:**
- Specific service (Samba, FileBrowser, etc.) not responding
- Pod crash loops or pending states
- Service degradation but system operational

**Recovery Steps:**

#### Quick Service Restart (0-5 minutes)
```bash
# 1. Identify failing service
kubectl get pods -A

# 2. Force restart
kubectl delete pod -l app=SERVICE_NAME -n home-services

# 3. Verify restoration
kubectl get pods -n home-services -w
```

#### Deep Troubleshooting (5-30 minutes)
```bash
# 1. Check pod logs
kubectl logs deployment/SERVICE_NAME -n home-services --tail=50

# 2. Describe pod for events
kubectl describe pod POD_NAME -n home-services

# 3. Check resource constraints
kubectl top pods -n home-services

# 4. Verify storage mounts
kubectl exec -it deployment/SERVICE_NAME -n home-services -- df -h

# 5. Redeploy if necessary
kubectl delete deployment SERVICE_NAME -n home-services
kubectl apply -f infrastructure/kubernetes/SERVICE_NAME.yaml
```

---

## 📋 Pre-Recovery Checklist

### Emergency Contact Information
- **System Administrator**: Joseph Prince
- **Email**: jprince0898@gmail.com
- **GitHub Repository**: https://github.com/ALphaOSIP/josephs-homelab-v2
- **Documentation Location**: /docs/ directory

### Required Resources
- [ ] Spare SD card (32GB minimum)
- [ ] USB SD card reader
- [ ] Backup USB drive (for data recovery)
- [ ] Network cable (bypass WiFi issues)
- [ ] Console access to Pi (keyboard/monitor)

### Recovery Toolkit
```bash
# Essential tools for recovery operations
sudo apt install -y ddrescue testdisk gddrescue
sudo apt install -y nmap curl wget git
sudo apt install -y fsck.ext4 parted gparted
```

### Backup Verification (Weekly)
```bash
#!/bin/bash
# backup-verification.sh

echo "=== Backup Verification $(date) ==="

# Check USB drive health
sudo smartctl -a /dev/sda

# Verify critical data
du -sh /mnt/nextcloud-data/vault
du -sh /mnt/nextcloud-data/iPhone

# Test file integrity
find /mnt/nextcloud-data -name "*.mp4" -exec md5sum {} \; > /tmp/checksums.md5

# Kubernetes state backup
kubectl get all -A -o yaml > /tmp/k8s-state-backup.yaml

echo "Backup verification complete"
```

---

## 🔄 Recovery Testing Schedule

### Monthly Recovery Drills
- **First Monday**: SD card replacement simulation
- **Second Monday**: Service failure response
- **Third Monday**: Network connectivity restoration
- **Fourth Monday**: Full disaster recovery test

### Quarterly Business Continuity Tests
- Complete infrastructure rebuild from scratch
- Data recovery from simulated drive failure
- Migration to alternative hardware platform
- Family user acceptance testing

---

## 📊 Recovery Metrics & KPIs

### Target Performance Indicators
| Metric | Target | Measurement |
|--------|--------|-------------|
| **Mean Time to Detect (MTTD)** | <5 minutes | Monitoring alerts |
| **Mean Time to Respond (MTTR)** | <15 minutes | First action taken |
| **Mean Time to Repair (MTTR)** | <2 hours | Service restored |
| **Data Loss** | <24 hours | Last backup point |
| **User Notification** | <30 minutes | Family communication |

### Success Criteria
- [ ] All critical services restored within RTO
- [ ] Data loss within acceptable RPO limits
- [ ] User access fully functional
- [ ] System performance at baseline levels
- [ ] Documentation updated with lessons learned

---

## 📝 Post-Recovery Procedures

### Immediate Actions (0-24 hours)
1. **Verify system stability** over extended operation
2. **Update all documentation** with configuration changes
3. **Notify stakeholders** of service restoration
4. **Begin root cause analysis** of failure

### Follow-up Actions (1-7 days)
1. **Conduct post-mortem review** with lessons learned
2. **Update recovery procedures** based on experience
3. **Implement preventive measures** to avoid recurrence
4. **Review and update backup strategies**

### Strategic Actions (1-4 weeks)
1. **Evaluate hardware reliability** and replacement needs
2. **Consider infrastructure improvements** (redundancy, monitoring)
3. **Update disaster recovery budget** and resource planning
4. **Schedule additional training** or skill development

---

## 🔗 Related Documentation

- **Setup Log**: Complete installation and configuration history
- **Architecture Diagram**: System design and component relationships  
- **Resource Metrics**: Performance baselines and capacity planning
- **Troubleshooting Guide**: Common issues and resolution procedures
- **Maintenance Schedule**: Preventive maintenance and update procedures

---

## 📞 Emergency Escalation Matrix

| Severity | Response Time | Escalation Path | Communication |
|----------|---------------|-----------------|---------------|
| **P1 - Critical** | Immediate | Direct intervention | Family notification |
| **P2 - High** | <30 minutes | Planned recovery | Status updates |
| **P3 - Medium** | <2 hours | Scheduled maintenance | Documentation |
| **P4 - Low** | <24 hours | Next available window | Monitoring only |

---

*Document Version: 1.0*  
*Last Updated: July 27, 2025*  
*Next Review: October 27, 2025*  
*Owner: Joseph Prince*
# Joseph's Homelab V2 - Technical Architecture

## Complete System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PHYSICAL INFRASTRUCTURE LAYER                             │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐              │
│  │   Home Router   │     │  TL-SG108E      │     │  Internet       │              │
│  │   (Gateway)     │─────│  Managed Switch │─────│  Connection     │              │
│  │ 192.168.1.1     │     │  (8-Port)       │     │                 │              │
│  └─────────────────┘     └─────────┬───────┘     └─────────────────┘              │
│                                    │                                              │
│  ┌─────────────────┐               │               ┌─────────────────┐            │
│  │  ThinkPad P16   │               │               │  Raspberry Pi 5 │            │
│  │  (Development) │               │               │   (alphapi5)    │            │
│  │ 192.168.1.86    │───────────────┼───────────────│ 192.168.1.100   │            │
│  │ Windows 11      │               │               │ Ubuntu 24.04.2  │            │
│  └─────────────────┘               │               └─────────┬───────┘            │
│                                    │                         │                    │
│                                    │               ┌─────────┴───────┐            │
│                                    │               │   Samsung T7     │            │
│                                    │               │   1TB USB SSD    │            │
│                                    │               │   (ext4)         │            │
│                                    │               └─────────────────┘            │
└─────────────────────────────────────┼─────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────────┐
│                           NETWORK LAYER                                            │
├─────────────────────────────────────┼─────────────────────────────────────────────┤
│                                     │                                             │
│  Network Segment: 192.168.1.0/24   │                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                    UFW Firewall Rules                                      │ │
│  │  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────┐  │ │
│  │  │    SSH      │ Kubernetes  │    SMB      │    HTTP     │   HTTPS     │  │ │
│  │  │   Port 22   │  Port 6443  │ Ports 139,  │   Port 80   │  Port 443   │  │ │
│  │  │             │             │    445      │             │             │  │ │
│  │  └─────────────┴─────────────┴─────────────┴─────────────┴─────────────┘  │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                             │
│  Static IP Configuration:           │                                             │
│  • Gateway: 192.168.1.1            │                                             │
│  • DNS: 1.1.1.1, 8.8.8.8          │                                             │
│  • Domain: alpha.home               │                                             │
└─────────────────────────────────────┼─────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────────┐
│                       KUBERNETES CLUSTER LAYER                                     │
│                        (K3s v1.32.6 - Single Node)                                │
├─────────────────────────────────────┼─────────────────────────────────────────────┤
│                                     │                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                        CONTROL PLANE NODE                                  │ │
│  │                        (alphapi5.alpha.home)                               │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐  │ │
│  │  │                    KUBERNETES SYSTEM SERVICES                      │  │ │
│  │  │                                                                     │  │ │
│  │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │  │ │
│  │  │  │    etcd     │ │  kube-api   │ │kube-scheduler│ │kube-controller│  │  │ │
│  │  │  │  (embedded) │ │   server    │ │             │ │   manager    │  │  │ │
│  │  │  │             │ │ :6443       │ │   :10251    │ │    :10252    │  │  │ │
│  │  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘  │  │ │
│  │  │                                                                     │  │ │
│  │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │  │ │
│  │  │  │   kubelet   │ │   CoreDNS   │ │ metrics-    │ │local-path-  │  │  │ │
│  │  │  │   :10250    │ │    :53      │ │  server     │ │ provisioner │  │  │ │
│  │  │  │             │ │             │ │             │ │             │  │  │ │
│  │  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘  │  │ │
│  │  └─────────────────────────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                           INGRESS LAYER                                    │ │
│  │                                                                             │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐  │ │
│  │  │                  Nginx Ingress Controller                          │  │ │
│  │  │                     (Port 32494)                                   │  │ │
│  │  │                                                                     │  │ │
│  │  │  ┌─────────────────┐              ┌─────────────────┐              │  │ │
│  │  │  │files.alpha.home │              │browse.alpha.home│              │  │ │
│  │  │  │       ↓         │              │       ↓         │              │  │ │
│  │  │  │   Nextcloud     │              │  FileBrowser    │              │  │ │
│  │  │  │   Service       │              │   Service       │              │  │ │
│  │  │  └─────────────────┘              └─────────────────┘              │  │ │
│  │  └─────────────────────────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────────┐
│                      APPLICATION LAYER                                             │
│                    (Namespace: home-services)                                      │
├─────────────────────────────────────┼─────────────────────────────────────────────┤
│                                     │                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                           FILE SHARING SERVICES                            │ │
│  │                                                                             │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │ │
│  │  │  Samba Server   │  │   FileBrowser   │  │  HTTP Server    │            │ │
│  │  │                 │  │                 │  │                 │            │ │
│  │  │ SMB/CIFS Share  │  │  Web Interface  │  │ Direct Access   │            │ │
│  │  │ Ports: 139,445  │  │ Port: 30880     │  │ Port: 30800     │            │ │
│  │  │                 │  │                 │  │                 │            │ │
│  │  │ Windows Drive:  │  │ Login:          │  │ URL:            │            │ │
│  │  │ \\192.168.1.100 │  │ admin/password  │  │ Direct Browse   │            │ │
│  │  │ \shared         │  │                 │  │                 │            │ │
│  │  └─────────┬───────┘  └─────────┬───────┘  └─────────┬───────┘            │ │
│  │            │                    │                    │                    │ │
│  │            └────────────────────┼────────────────────┘                    │ │
│  │                                 │                                         │ │
│  └─────────────────────────────────┼─────────────────────────────────────────┘ │
│                                    │                                           │
│  ┌─────────────────────────────────┼─────────────────────────────────────────┐ │
│  │                           WEB SERVICES                                   │ │
│  │                                 │                                         │ │
│  │  ┌─────────────────┐           │           ┌─────────────────┐           │ │
│  │  │   Nextcloud     │           │           │      Nginx      │           │ │
│  │  │                 │           │           │  Ingress Ctrl   │           │ │
│  │  │ Cloud Storage   │           │           │                 │           │ │
│  │  │ Port: 30808     │           │           │ Load Balancer   │           │ │
│  │  │                 │           │           │ SSL Termination │           │ │
│  │  │ Login:          │           │           │                 │           │ │
│  │  │ admin/changeme  │           │           │                 │           │ │
│  │  └─────────────────┘           │           └─────────────────┘           │ │
│  └─────────────────────────────────┼─────────────────────────────────────────┘ │
└─────────────────────────────────────┼─────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────┼─────────────────────────────────────────────┐
│                        STORAGE LAYER                                               │
├─────────────────────────────────────┼─────────────────────────────────────────────┤
│                                     │                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                      PERSISTENT STORAGE SYSTEM                             │ │
│  │                                                                             │ │
│  │  ┌─────────────────┐              ┌─────────────────┐                      │ │
│  │  │   Host Path     │              │  Kubernetes     │                      │ │
│  │  │   Volumes       │◄─────────────┤   Persistent    │                      │ │
│  │  │                 │              │    Volumes      │                      │ │
│  │  │ /mnt/nextcloud- │              │                 │                      │ │
│  │  │     data        │              │  ┌─────────────┐│                      │ │
│  │  │        ↓        │              │  │Volume Claims││                      │ │
│  │  │ ┌─────────────┐ │              │  │    (PVC)    ││                      │ │
│  │  │ │Samsung T7   │ │              │  └─────────────┘│                      │ │
│  │  │ │1TB USB SSD │ │              │                 │                      │ │
│  │  │ │            │ │              │ Storage Classes: │                      │ │
│  │  │ │960GB Avail │ │              │ • local-path    │                      │ │
│  │  │ │ext4 format │ │              │ • hostPath      │                      │ │
│  │  │ └─────────────┘ │              └─────────────────┘                      │ │
│  │  └─────────────────┘                                                       │ │
│  │                                                                             │ │
│  │  User Data Structure:                                                       │ │
│  │  /mnt/nextcloud-data/                                                       │ │
│  │  ├── iPhone/                     (User photos & backups)                   │ │
│  │  ├── vault/                      (Family shared files)                     │ │
│  │  │   ├── Joseph's Folder/                                                   │ │
│  │  │   ├── Movies/                                                            │ │
│  │  │   └── Hannifty's Folder/                                                 │ │
│  │  ├── lost+found/                 (Filesystem recovery)                      │ │
│  │  └── [Nextcloud system files]    (Web app data)                            │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                               ACCESS METHODS                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                 │
│  │  Windows SMB    │  │  Web Interface  │  │ Direct HTTP     │                 │
│  │                 │  │                 │  │                 │                 │
│  │ Network Drive:  │  │ FileBrowser:    │  │ Simple Browse:  │                 │
│  │ Z: Drive        │  │ Full Featured   │  │ Read-Only       │                 │
│  │                 │  │                 │  │                 │                 │
│  │ • File Explorer │  │ • Upload/       │  │ • Quick Access  │                 │
│  │ • Native Apps   │  │   Download      │  │ • No Login      │                 │
│  │ • Office Suite │  │ • File Mgmt     │  │ • URL Sharing   │                 │
│  │ • Media Players │  │ • User Mgmt     │  │                 │                 │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘                 │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                          PERFORMANCE METRICS                           │   │
│  │                                                                         │   │
│  │  • Memory Usage: 11% (under normal load)                               │   │
│  │  • CPU Temperature: 63°C (optimal thermal range)                       │   │
│  │  • Storage Throughput: 1GB/s (USB 3.2 Gen 2)                          │   │
│  │  • Network Latency: <1ms (local LAN)                                   │   │
│  │  • Available Storage: 960GB / 1TB (96% utilization)                    │   │
│  │  • Concurrent Users: 2-5 (household capacity)                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Network Flow Diagram

```
Client Access Patterns:

Windows PC (192.168.1.86)
    │
    ├─ SMB/CIFS ──────► Port 445/139 ──► Samba Pod ──► USB Storage
    │                                       
    ├─ HTTP/HTTPS ────► Port 32494 ──► Nginx Ingress ──► FileBrowser Pod ──► USB Storage
    │                                       │
    └─ Direct HTTP ───► Port 30880 ──────────┘
                        Port 30800 ──► HTTP Server Pod ──► USB Storage

Development Machine (ThinkPad P16)
    │
    ├─ SSH ───────────► Port 22 ────► alphapi5 (kubectl, management)
    │
    ├─ HTTP/HTTPS ────► Port 32494 ──► Web Services (testing, file access)
    │
    └─ Git ───────────► GitHub ──────► Version Control & Documentation
```

## Security Boundaries

```
┌─────────────────────────────────────────────────────────────────┐
│                      SECURITY LAYERS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │               PERIMETER SECURITY                          │  │
│  │                                                           │  │
│  │  Router Firewall ──► Port Forwarding ──► Public Access   │  │
│  │  (Future: VPN Only)     (Disabled)      (None)          │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                 HOST SECURITY                             │  │
│  │                                                           │  │
│  │  UFW Firewall Rules:                                     │  │
│  │  ├─ SSH (22/tcp)      ──► Management Access              │  │
│  │  ├─ Kubernetes (6443) ──► API Server                     │  │
│  │  ├─ SMB (139,445)     ──► File Sharing                   │  │
│  │  ├─ HTTP (80)         ──► Web Services                   │  │
│  │  └─ HTTPS (443)       ──► Secure Web                     │  │
│  │                                                           │  │
│  │  Authentication:                                         │  │
│  │  ├─ SSH Keys (Ed25519) ──► Cryptographic                 │  │
│  │  └─ No Password Auth   ──► Secure Only                   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              CONTAINER SECURITY                           │  │
│  │                                                           │  │
│  │  Kubernetes Network Policies:                            │  │
│  │  ├─ Namespace Isolation ──► Service Boundaries           │  │
│  │  ├─ Pod-to-Pod Rules   ──► Controlled Communication      │  │
│  │  └─ Ingress Controls   ──► External Access Points        │  │
│  │                                                           │  │
│  │  Container Runtime Security:                             │  │
│  │  ├─ Non-root Containers ──► Privilege Separation         │  │
│  │  ├─ Resource Limits     ──► DoS Prevention               │  │
│  │  └─ Image Scanning      ──► Vulnerability Management     │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Resource Allocation

| Service | CPU | Memory | Storage | Network |
|---------|-----|--------|---------|---------|
| **K3s System** | 200m | 512Mi | 2GB | Internal |
| **Samba Server** | 100m | 128Mi | USB Mount | 445,139 |
| **FileBrowser** | 50m | 64Mi | USB Mount | 30880 |
| **Nginx Ingress** | 100m | 128Mi | ConfigMap | 32494 |
| **Nextcloud** | 200m | 256Mi | USB Mount | 30808 |
| **Total Allocated** | 650m | 1088Mi | 960GB | Multiple |
| **System Available** | 4000m | 8192Mi | 960GB | 1Gbps |
| **Utilization** | 16% | 13% | 96% | <5% |

---

*This architecture demonstrates enterprise-grade DevOps practices including container orchestration, service mesh networking, persistent storage management, and systematic security implementation.*
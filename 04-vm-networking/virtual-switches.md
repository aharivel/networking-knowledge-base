# Virtual Switches

> Software-based Layer 2 networking for virtual machines

---

## 🎯 Purpose

Virtual switches (vSwitches) are **software-based network switches** that operate in the hypervisor layer, enabling virtual machines (VMs) to communicate with each other and with the physical network. They replicate the functionality of physical switches in a virtualized environment.

**Key Capabilities:**
- **Layer 2 switching** between VMs on the same host
- **VLAN support** for network segmentation
- **Port mirroring** for monitoring
- **Quality of Service (QoS)** policies
- **Security features** (ACLs, port security)
- **Integration** with physical networks

## 🏗️ Virtual Switch Types

### Standard Virtual Switch (vSwitch)

**Purpose**: Basic Layer 2 switching for VMs on a single host

**Characteristics:**
- Operates at Layer 2 (MAC addresses)
- Connects VMs to each other and to physical network
- Typically implemented as a software bridge

**Example Implementations:**
- VMware vSwitch
- Hyper-V Virtual Switch
- Xen Bridge
- KVM Bridge (Linux bridge)

### Distributed Virtual Switch (DvSwitch)

**Purpose**: Consistent networking policies across multiple hosts

**Characteristics:**
- **Centralized management** of networking configuration
- **Distributed forwarding** (each host makes forwarding decisions)
- **Consistent policies** across the entire cluster
- **Visibility** into virtual network traffic

**Example Implementations:**
- VMware vNetwork Distributed Switch (VDS)
- Cisco Nexus 1000V
- Open vSwitch (OVS) with distributed configuration

## 🌐 Open vSwitch (OVS)

### What is Open vSwitch?

**Open vSwitch (OVS)** is an open-source, production-quality, multilayer virtual switch designed to enable massive network automation through programmatic extension, while still supporting standard management interfaces and protocols.

**Developer**: Originally created at Nicira (acquired by VMware), now maintained by the Open vSwitch community

**License**: Apache 2.0

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Open vSwitch Architecture                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    User Space                             │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │  │
│  │  │  ovsdb-      │  │  ovs-        │  │  ofproto/      │  │  │
│  │  │  server      │  │  vswitchd   │  │  dpdk/...      │  │  │
│  │  │ (Config DB)  │  │ (Main Daemon)│  │  (Offload)     │  │  │
│  │  └─────────────┘  └────────┬────────┘  └─────────────────┘  │  │
│  │                          │                               │  │
│  └──────────────────────────────┼───────────────────────────┘  │
│                                   │                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Kernel Space                           │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │  │
│  │  │  Datapath    │  │  Datapath    │  │  Kernel Module   │  │  │
│  │  │  (Userspace) │  │  (Kernel)    │  │  (openvswitch)   │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Hardware                              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │  │
│  │  │  DPDK NIC    │  │  Physical    │  │  DPDK NIC       │  │  │
│  │  │              │  │  NIC         │  │                 │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Purpose | Implementation |
|-----------|---------|----------------|
| **ovs-vswitchd** | Main daemon, handles configuration and forwarding | User space |
| **ovsdb-server** | Configuration database server | User space |
| **ovs-dpdk** | DPDK-based datapath for high performance | User space |
| **Kernel Module** | Fast path forwarding | Kernel space |
| **OVSDB** | Configuration database | User space |
| **ofproto** | OpenFlow protocol implementation | User space |

### OVS Datapath Options

| Datapath | Description | Performance | Use Case |
|----------|-------------|------------|----------|
| **Kernel** | Standard Linux kernel module | ⭐⭐⭐ | General purpose, most common |
| **Userspace** | Pure userspace implementation | ⭐ | Special cases, debugging |
| **DPDK** | DPDK-accelerated datapath | ⭐⭐⭐⭐⭐ | High performance, NFV |
| **AF_XDP** | eXpress Data Path (XDP) | ⭐⭐⭐⭐ | High performance, kernel 4.18+ |

### OVS vs Linux Bridge

| Feature | Open vSwitch | Linux Bridge |
|---------|--------------|--------------|
| **OpenFlow Support** | ✅ Full support | ❌ No |
| **VLAN Support** | ✅ Yes (with trunking) | ✅ Yes |
| **VXLAN/GRE/Geneve** | ✅ Yes | ❌ No (requires additional modules) |
| **Qos Support** | ✅ Yes | ✅ Basic |
| **NetFlow/sFlow** | ✅ Yes | ❌ No |
| **SPAN/RSPAN** | ✅ Yes | ❌ No |
| **Bonding** | ✅ Yes (LACP, active-backup) | ✅ Yes |
| **Tunnel Support** | ✅ Yes | ❌ No |
| **DPDK Support** | ✅ Yes | ❌ No |
| **Performance** | ⭐⭐⭐⭐ (with DPDK) | ⭐⭐⭐ |
| **Management** | ✅ ovs-vsctl, ovs-ofctl | ✅ brctl (deprecated) |

### Installation

**Ubuntu/Debian:**
```bash
# Install OVS
sudo apt update
sudo apt install -y openvswitch-switch openvswitch-common

# Start services
sudo systemctl start openvswitch-switch
sudo systemctl enable openvswitch-switch

# Verify
sudo ovs-vsctl show
```

**CentOS/RHEL:**
```bash
# Install OVS
sudo yum install -y openvswitch

# Start services
sudo systemctl start openvswitch
sudo systemctl enable openvswitch

# Verify
sudo ovs-vsctl show
```

**From Source:**
```bash
# Install dependencies
sudo apt install -y build-essential libssl-dev python3 python3-pip autoconf libtool

# Clone and build
git clone https://github.com/openvswitch/ovs.git
git checkout v3.1.0  # or latest stable
./boot.sh
./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc
make
sudo make install

# Initialize database
sudo ovsdb-tool create /etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema

# Start services
sudo ovsdb-server --remote=punix:/var/run/openvswitch/db.sock \
                 --remote=db:Open_vSwitch,Open_vSwitch,manager_options \
                 --private-key=db:Open_vSwitch,SSL,private_key \
                 --certificate=db:Open_vSwitch,SSL,certificate \
                 --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \
                 --pidfile --detach

sudo ovs-vsctl --no-wait init
sudo ovs-vswitchd --pidfile --detach
```

### Basic Configuration

**Create a Bridge:**
```bash
# Create a new bridge
sudo ovs-vsctl add-br br0

# List bridges
sudo ovs-vsctl list-br

# Show bridge details
sudo ovs-vsctl show
```

**Add Physical Interface:**
```bash
# Add physical interface to bridge
sudo ovs-vsctl add-port br0 eth1

# Verify
sudo ovs-vsctl list-ports br0
```

**Add Virtual Interface (for VMs):**
```bash
# Create tap interface for VM
sudo ip tuntap add mode tap vport1

# Add to bridge
sudo ovs-vsctl add-port br0 vport1

# Verify
sudo ovs-vsctl list-ports br0
```

**Configure VLANs:**
```bash
# Create VLAN-aware bridge
sudo ovs-vsctl set bridge br0 vlan_filtering=true

# Add port with VLAN tag
sudo ovs-vsctl add-port br0 eth1 tag=10

# Create trunk port (accept multiple VLANs)
sudo ovs-vsctl set port eth1 trunks=10,20,30

# Verify VLAN configuration
sudo ovs-vsctl list port eth1
```

### OpenFlow Configuration

**Enable OpenFlow:**
```bash
# Set bridge as OpenFlow switch
sudo ovs-vsctl set-br br0 other-config:disable-in-band=true
sudo ovs-vsctl set-controller br0 tcp:192.168.1.100:6653

# Verify controller
sudo ovs-vsctl get-controller br0
```

**Add OpenFlow Rules:**
```bash
# Using ovs-ofctl
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=1, actions=output:2"
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=2, actions=output:1"

# Show flow table
sudo ovs-ofctl dump-flows br0

# Show statistics
sudo ovs-ofctl dump-ports br0
sudo ovs-ofctl dump-ports br0 -O OpenFlow13
```

**OpenFlow Example Rules:**
```bash
# Allow all traffic between port 1 and 2
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=1, actions=output:2"
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=2, actions=output:1"

# Drop all traffic from port 3
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=3, actions=drop"

# Forward based on MAC address
sudo ovs-ofctl add-flow br0 "table=0, priority=100, dl_dst=00:11:22:33:44:55, actions=output:2"

# Forward based on IP address
sudo ovs-ofctl add-flow br0 "table=0, priority=100, nw_dst=192.168.1.100, actions=output:3"

# VLAN tagging
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=1, dl_vlan=10, actions=strip_vlan,output:2"
sudo ovs-ofctl add-flow br0 "table=0, priority=100, in_port=2, actions=mod_vlan_vid:10,output:1"
```

### Tunneling with OVS

**Create VXLAN Tunnel:**
```bash
# Create VXLAN port
sudo ovs-vsctl add-port br0 vxlan100 \
  -- set interface vxlan100 type=vxlan \
  -- set interface vxlan100 options:remote_ip=192.168.1.101 \
  -- set interface vxlan100 options:key=100 \
  -- set interface vxlan100 options:dst_port=4789

# Verify
sudo ovs-vsctl list-ports br0
sudo ovs-vsctl get interface vxlan100 options
```

**Create GRE Tunnel:**
```bash
# Create GRE port
sudo ovs-vsctl add-port br0 gre100 \
  -- set interface gre100 type=gre \
  -- set interface gre100 options:remote_ip=192.168.1.101 \
  -- set interface gre100 options:key=100

# Verify
sudo ovs-vsctl get interface gre100 options
```

**Create Geneve Tunnel:**
```bash
# Create Geneve port
sudo ovs-vsctl add-port br0 geneve100 \
  -- set interface geneve100 type=geneve \
  -- set interface geneve100 options:remote_ip=192.168.1.101 \
  -- set interface geneve100 options:key=100 \
  -- set interface geneve100 options:dst_port=6081

# Verify
sudo ovs-vsctl get interface geneve100 options
```

### QoS Configuration

**Create QoS Queue:**
```bash
# Create QoS with type
sudo ovs-vsctl set port eth1 qos=@newqos \
  -- create qos name=myqos type=linux-htb other-config:max-rate=1000000000 \
  -- set bridge br0 qos=@newqos

# Create queue
sudo ovs-vsctl -- create queue other-config:max-rate=500000000 \
  -- set port eth1 queues:0=@queue0

# Apply to traffic
sudo ovs-ofctl add-flow br0 "table=0, priority=1000, in_port=1, actions=set_queue:0,output:2"
```

**Limit Bandwidth:**
```bash
# Set ingress policing
sudo ovs-vsctl set interface eth1 ingress_policing_rate=1000000
sudo ovs-vsctl set interface eth1 ingress_policing_burst=100000
```

### Monitoring and Troubleshooting

**Show Bridge Information:**
```bash
# Show all bridges and ports
sudo ovs-vsctl show

# Show bridge statistics
sudo ovs-ofctl show br0

# Show port statistics
sudo ovs-ofctl dump-ports br0

# Show flow table
sudo ovs-ofctl dump-flows br0

# Show interface statistics
sudo ovs-vsctl list interface eth1
```

**Packet Capture:**
```bash
# Capture on specific port
sudo ovs-tcpdump -i br0

# Capture with specific filters
sudo ovs-tcpdump -i br0 -n -vv port 80

# Save to file
sudo ovs-tcpdump -i br0 -w /tmp/capture.pcap
```

**Check Connectivity:**
```bash
# Show MAC table
sudo ovs-appctl fdb/show br0

# Show VLAN table
sudo ovs-appctl vlan/show br0

# Show MDB (multicast)
sudo ovs-appctl mdb/show br0
```

**Common Commands:**
```bash
# Restart OVS
sudo systemctl restart openvswitch-switch

# Check logs
journalctl -u openvswitch-switch -f

# Check kernel module
lsmod | grep openvswitch

# Reset configuration
sudo ovs-vsctl del-br br0
sudo ovs-vsctl del-controller br0
```

## 🌐 Linux Bridge

### What is Linux Bridge?

The **Linux bridge** is a Layer 2 virtual switch built into the Linux kernel. It's simpler than OVS but lacks advanced features.

### Use Cases

| Use Case | Recommended |
|----------|-------------|
| Simple VM networking | ✅ Yes |
| Basic host-container networking | ✅ Yes |
| Advanced networking (VXLAN, OpenFlow) | ❌ No (use OVS) |
| High performance NFV | ❌ No (use OVS with DPDK) |

### Configuration

**Create Bridge:**
```bash
# Install bridge utilities
sudo apt install -y bridge-utils

# Create bridge
sudo brctl addbr br0

# Add interfaces to bridge
sudo brctl addif br0 eth1
sudo brctl addif br0 tap0

# Show bridge info
sudo brctl show
```

**Configure with iproute2:**
```bash
# Create bridge
ip link add br0 type bridge
ip link set br0 up

# Add physical interface
ip link set eth1 master br0
ip link set eth1 up

# Add tap interface for VM
ip tuntap add mode tap vport1
ip link set vport1 master br0
ip link set vport1 up

# Show bridge info
bridge link show
```

**Configure VLANs:**
```bash
# Install VLAN support
sudo apt install -y vlan

# Create VLAN interface on bridge
ip link add link eth1 name eth1.10 type vlan id 10
ip link set eth1.10 master br0
ip link set eth1.10 up

# Assign IP
ip addr add 192.168.10.1/24 dev br0
```

**STP (Spanning Tree Protocol):**
```bash
# Enable STP
ip link set br0 type bridge stp on

# Show STP info
bridge stp show
```

### Linux Bridge vs OVS

| Feature | Linux Bridge | Open vSwitch |
|---------|--------------|--------------|
| **Performance** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ (with DPDK) |
| **VLAN Support** | ✅ Yes | ✅ Yes (more flexible) |
| **VXLAN/GRE/Geneve** | ❌ No | ✅ Yes |
| **OpenFlow** | ❌ No | ✅ Yes |
| **QoS** | ❌ Basic | ✅ Advanced |
| **NetFlow/sFlow** | ❌ No | ✅ Yes |
| **SPAN** | ❌ No | ✅ Yes |
| **Bonding** | ✅ Yes | ✅ Yes (more options) |
| **DPDK Support** | ❌ No | ✅ Yes |
| **Ease of Use** | ⭐⭐⭐⭐ | ⭐⭐ |
| **Management Tools** | brctl, ip | ovs-vsctl, ovs-ofctl |

## 🔌 VMware Virtual Switch

### VMware vSwitch Types

| Type | Description | Features |
|------|-------------|----------|
| **Standard vSwitch** | Single-host virtual switch | Basic Layer 2 switching, VLANs, port groups |
| **Distributed vSwitch (VDS)** | Cluster-wide virtual switch | Centralized management, distributed forwarding, private VLANs, port mirroring |

### Standard vSwitch

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    ESXi Host                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────┐  │
│  │    VM 1          │    │    VM 2          │    │  vSwitch│  │
│  │   vNIC          │    │   vNIC          │    │         │  │
│  └────────┬────────┘    └────────┬────────┘    └────┬──────┘  │
│           │                     │                │       │
│           ▼                     ▼                │       ▼
│  ┌────────────────────────────────────────────┐       │  │
│  │                Port Group A (VLAN 10)        │       │  │
│  └────────────────────────────────────────────┘       │  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    vSwitch0                             │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │  │
│  │  │ PG A    │  │ PG B    │  │ PG C    │  │ Uplink  │  │  │
│  │  │ VLAN 10 │  │ VLAN 20 │  │ VLAN 30 │  │         │  │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └────┬──────┘  │  │
│  │                                                  │    │  │
│  └──────────────────────────────────────────────┼────────┘  │
│                                                   │           │
│                                                   ▼           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Physical NIC (vmnic0)                │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Distributed vSwitch (VDS)

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                    vCenter Server                              │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    vSphere Distributed Switch               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │  │
│  │  │  vCenter    │  │  VDS        │  │  Distributed    │  │  │
│  │  │  Management │  │  Control    │  │  Port Groups    │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌───────────────────┐       │       ┌───────────────────┐
│   ESXi Host 1     │       │       │   ESXi Host 2     │
├───────────────────┤       │       ├───────────────────┤
│                   │       │       │                   │
│  ┌────────────┐  │       │       │  ┌────────────┐  │
│  │   VM 1     │  │       │       │  │   VM 2     │  │
│  │   vNIC     │  │       │       │  │   vNIC     │  │
│  └─────┬──────┘  │       │       │  └─────┬──────┘  │
│        │         │       │       │        │         │
│        ▼         │       │       │        ▼         │
│  ┌─────────────┐ │       │       │  ┌─────────────┐ │
│  │  VDS        │◄───────┼───────►│  │  VDS        │ │
│  │  Instance   │ │       │       │  │  Instance   │ │
│  └─────────────┘ │       │       │  └─────────────┘ │
└───────────────────┘       │       └───────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Physical Network                              │
└─────────────────────────────────────────────────────────────┘
```

### Comparison: vSwitch vs VDS

| Feature | Standard vSwitch | Distributed vSwitch |
|---------|-------------------|---------------------|
| **Scope** | Single host | Cluster-wide |
| **Management** | Per-host | Centralized |
| **Configuration Sync** | Manual | Automatic |
| **Port Groups** | Per-host | Cluster-wide |
| **Private VLANs** | ❌ No | ✅ Yes |
| **Port Mirroring** | ✅ Yes | ✅ Yes |
| **Network I/O Control** | ❌ No | ✅ Yes |
| **QoS** | ❌ No | ✅ Yes |
| **NetFlow** | ❌ No | ✅ Yes |
| **LACP** | ❌ No | ✅ Yes |
| **Health Check** | ❌ No | ✅ Yes |
| **Backup & Restore** | ❌ No | ✅ Yes |
| **Use Case** | Small environments, test/dev | Production, large environments |

## 🔌 Hyper-V Virtual Switch

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Hyper-V Host                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────┐  │
│  │    VM 1          │    │    VM 2          │    │  Virtual    │  │
│  │   vNIC          │    │   vNIC          │    │  Switch     │  │
│  └────────┬────────┘    └────────┬────────┘    │  Manager    │  │
│           │                     │               │             │  │
│           ▼                     ▼               │             │  │
│  ┌─────────────────────────────────────────────┐ │             │  │
│  │                vSwitch (Data Plane)          │ │             │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐ │             │  │
│  │  │ Port 1  │  │ Port 2  │  │  Uplink Port    │ │             │  │
│  │  │ (VM 1)  │  │ (VM 2)  │  │  (Physical NIC) │ │             │  │
│  │  └─────────┘  └─────────┘  └─────────────────┘ │             │  │
│  └─────────────────────────────────────────────┘ │             │  │
│                                                   │             │  │
│  ┌─────────────────────────────────────────────┐ │             │  │
│  │                VMSMP (Switch Extension)      │ │             │  │
│  │  - Packet filtering                          │ │             │  │
│  │  - QoS                                       │ │             │  │
│  │  - Port mirroring                            │ │             │  │
│  └─────────────────────────────────────────────┘ │             │  │
└─────────────────────────────────────────────────────────────┘
```

### Types of Virtual Switches

| Type | Description | Use Case |
|------|-------------|----------|
| **External** | VMs can communicate with physical network | Default, most common |
| **Internal** | VMs can only communicate with each other and host | Isolated environments |
| **Private** | VMs can only communicate with each other | Maximum isolation |

### Configuration

**Create Virtual Switch (PowerShell):**
```powershell
# Create external virtual switch
New-VMSwitch -Name "ExternalSwitch" -SwitchType External -NetAdapterName "Ethernet"

# Create internal virtual switch
New-VMSwitch -Name "InternalSwitch" -SwitchType Internal

# Create private virtual switch
New-VMSwitch -Name "PrivateSwitch" -SwitchType Private

# List all switches
Get-VMSwitch
```

**Connect VM to Switch:**
```powershell
# Create VM with switch connection
New-VM -Name "VM1" -MemoryStartupBytes 2GB -VHDPath "C:\VMs\VM1.vhdx"

# Add network adapter connected to switch
Add-VMNetworkAdapter -VMName "VM1" -SwitchName "ExternalSwitch"

# Configure VLAN
Set-VMNetworkAdapterVlan -VMName "VM1" -VMNetworkAdapterName "Network Adapter" -AccessVlanId 10

# Configure QoS
Set-VMNetworkAdapter -VMName "VM1" -Name "Network Adapter" -MinimumBandwidthAbsolute 100Mb
```

**Advanced Features:**

**SR-IOV:**
```powershell
# Enable SR-IOV on physical NIC
Enable-NetAdapterSriov -Name "Ethernet"

# Configure VM for SR-IOV
Set-VMNetworkAdapter -VMName "VM1" -Name "Network Adapter" -SriovEnabled $true
```

**Port Mirroring:**
```powershell
# Create mirror port
Add-VMSwitchPortMirror -Name "MonitorPort" -SourceVMSwitch "ExternalSwitch" -DestinationPort "DestinationPort"
```

**Network Security:**
```powershell
# Enable MAC address spoofing protection
Set-VMNetworkAdapter -VMName "VM1" -MACAddressSpoofing On

# Enable DHCP guard
Set-VMNetworkAdapter -VMName "VM1" -DHCPGuard On

# Enable Router guard
Set-VMNetworkAdapter -VMName "VM1" -RouterGuard On
```

## 📊 Virtual Switch Performance

### Performance Factors

| Factor | Impact | Optimization |
|--------|--------|--------------|
| **Switch Type** | Software vs hardware offload | Use SR-IOV, DPDK |
| **Number of VMs** | More VMs = more overhead | Limit VMs per switch |
| **VLANs** | More VLANs = more complexity | Limit VLANs per switch |
| **MTU** | Larger MTU = better throughput | Use jumbo frames |
| **Hardware Offload** | Reduces CPU usage | Enable SR-IOV, VMQ |
| **Driver** | Affects performance | Use latest drivers |

### Performance Comparison

| Virtual Switch | Throughput | Latency | CPU Usage | Hardware Offload |
|----------------|------------|---------|-----------|-------------------|
| **Linux Bridge** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ❌ No |
| **Open vSwitch (Kernel)** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ❌ No |
| **Open vSwitch (DPDK)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ | ✅ Yes |
| **Open vSwitch (AF_XDP)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ | ✅ Yes |
| **VMware vSwitch** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ❌ No |
| **VMware VDS** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ Yes |
| **Hyper-V vSwitch** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ✅ Yes (SR-IOV) |

### Optimizing Performance

1. **Use Jumbo Frames**:
   ```bash
   # Linux
   ip link set eth0 mtu 9000
   
   # VMware
   # Set MTU in vSwitch settings
   
   # Hyper-V
   Set-NetAdapter -Name "Ethernet" -MTU 9000
   ```

2. **Enable Hardware Offload**:
   ```bash
   # Check offload capabilities
   ethtool -k eth0
   
   # Enable offload
   ethtool -K eth0 tx off rx off sg off tso off gso off gro off lro off
   # Note: These may need to be disabled for OVS/DPDK
   ```

3. **Use SR-IOV**:
   - Assign virtual functions (VFs) directly to VMs
   - Bypasses virtual switch for certain traffic
   - Reduces CPU overhead

4. **Tune Interrupt Coalescing**:
   ```bash
   # Increase interrupt coalescing
   ethtool -C eth0 rx-usecs 100
   ethtool -C eth0 rx-frames 100
   ```

5. **Use Multiple Queues**:
   ```bash
   # Enable multi-queue
   ethtool -L eth0 combined 8
   
   # In OVS
   sudo ovs-vsctl set interface eth1 options:n_rxq=8
   ```

## 🔐 Security Features

### Virtual Switch Security

| Feature | Description | Linux Bridge | OVS | VMware | Hyper-V |
|---------|-------------|--------------|-----|--------|---------|
| **Port Security** | MAC address filtering | ✅ | ✅ | ✅ | ✅ |
| **MAC Learning** | Control MAC table learning | ❌ | ✅ | ✅ | ✅ |
| **VLAN Isolation** | Prevent VLAN hopping | ✅ | ✅ | ✅ | ✅ |
| **Private VLANs** | Isolate ports within VLAN | ❌ | ✅ | ✅ | ❌ |
| **DHCP Snooping** | Prevent DHCP spoofing | ❌ | ✅ | ✅ | ✅ |
| **ARP Inspection** | Prevent ARP spoofing | ❌ | ✅ | ✅ | ✅ |
| **IP Source Guard** | Prevent IP spoofing | ❌ | ✅ | ✅ | ✅ |
| **Storm Control** | Rate limit broadcast/multicast | ❌ | ✅ | ✅ | ✅ |
| **Port Mirroring** | Monitor traffic | ❌ | ✅ | ✅ | ✅ |
| **ACLs** | Access control lists | ❌ | ✅ | ✅ | ✅ |
| **QoS** | Quality of Service | ❌ | ✅ | ✅ | ✅ |

### OVS Security Configuration

**Port Security:**
```bash
# Enable port security
sudo ovs-vsctl set port eth1 other-config:port-security=true

# Set allowed MAC addresses
sudo ovs-vsctl set port eth1 other-config:allowed-addresses="00:11:22:33:44:55 00:11:22:33:44:56"
```

**MAC Learning:**
```bash
# Limit MAC table size
sudo ovs-vsctl set bridge br0 other-config:mac-table-size=1000

# Set MAC aging time
sudo ovs-vsctl set bridge br0 other-config:mac-ageing-time=300
```

**VLAN Security:**
```bash
# Prevent VLAN tagging by VMs (port-based VLANs only)
sudo ovs-vsctl set port vport1 other-config:disable-in-band=true

# Drop untagged packets on trunk port
sudo ovs-vsctl set port eth1 other-config:drop-untagged=true
```

**Storm Control:**
```bash
# Rate limit broadcast traffic
sudo ovs-vsctl set port eth1 other-config:broadcast-limit=100
sudo ovs-vsctl set port eth1 other-config:broadcast-burst=200
```

### VMware Security Features

**Port Security:**
- **Promiscuous Mode**: Allow VM to see all traffic (default: Reject)
- **MAC Address Changes**: Allow VM to change its MAC address (default: Reject)
- **Forged Transmits**: Allow VM to send packets with any MAC (default: Reject)

**Configuration:**
```
# In vSphere Client:
1. Navigate to vSwitch properties
2. Select Port Group
3. Edit Security settings
```

**Private VLANs:**
- **Primary VLAN**: Normal VLAN
- **Isolated VLAN**: Ports cannot communicate with each other
- **Community VLAN**: Ports can communicate with each other and primary
- **Promiscuous VLAN**: Can communicate with all (uplink ports)

## 🛠️ Troubleshooting Virtual Switches

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **No connectivity between VMs** | VMs can't ping each other | Check port groups, VLANs, firewall |
| **No external connectivity** | VMs can't reach external network | Check uplink configuration, gateway |
| **High CPU usage** | Host CPU at 100% | Check offload settings, SR-IOV |
| **Packet drops** | Network performance issues | Check statistics, MTU, NIC errors |
| **VLAN issues** | VMs on same VLAN can't communicate | Check VLAN tagging, trunk configuration |
| **STP blocking** | Intermittent connectivity | Check STP configuration, topology |
| **DPDK errors** | OVS with DPDK not working | Check huge pages, NIC binding |

### OVS Troubleshooting Commands

```bash
# Show bridge information
sudo ovs-vsctl show

# Show port statistics
sudo ovs-ofctl show br0

# Show flow table
sudo ovs-ofctl dump-flows br0

# Show interface statistics
sudo ovs-vsctl list interface eth1

# Show MAC table
sudo ovs-appctl fdb/show br0

# Show VLAN table
sudo ovs-appctl vlan/show br0

# Check kernel datapath
sudo ovs-dpctl show

# Show datapath flows
sudo ovs-dpctl dump-flows

# Check logs
journalctl -u openvswitch-switch -f

# Check kernel logs
dmesg | grep -i ovs
```

### VMware Troubleshooting

```powershell
# Check virtual switch configuration
Get-VMSwitch | Format-List *

# Check VM network adapter
Get-VMNetworkAdapter -VMName "VM1" | Format-List *

# Check port group
Get-VMHostNetworkAdapter | Format-List *

# Check teaming policy
Get-VMSwitch -Name "vSwitch0" | Get-NicTeamingPolicy

# Check VLAN configuration
Get-VMNetworkAdapter -VMName "VM1" | Get-VMNetworkAdapterVlan

# Check health
Test-VMSwitch -Name "vSwitch0"
```

### Hyper-V Troubleshooting

```powershell
# Check virtual switch
Get-VMSwitch | Format-List *

# Check VM network adapter
Get-VMNetworkAdapter -VMName "VM1"

# Check VLAN
Get-VMNetworkAdapterVlan -VMName "VM1"

# Check SR-IOV
Get-VMNetworkAdapter -VMName "VM1" | Select-Object Name, SriovEnabled

# Check NIC teaming
Get-NetLbfoTeam | Format-List *

# Check health
Test-NetConnection -ComputerName "192.168.1.100" -Port 80
```

### General Troubleshooting

```bash
# Check network interfaces
ip a
ifconfig -a

# Check routes
ip route
route -n

# Check ARP table
ip neigh
arp -a

# Check bridge MAC table
bridge fdb show

# Check connection
ping 192.168.1.100

# Check DNS
nslookup google.com

# Check MTU
ip link show | grep mtu

# Packet capture
tcpdump -i eth0 -n -vv
tcpdump -i br0 -n -vv
```

## 🎯 Key Takeaways

1. **Virtual switches enable VM networking** within a hypervisor
2. **Open vSwitch (OVS)** is the most feature-rich open-source vSwitch
3. **Linux Bridge** is simple but lacks advanced features
4. **VMware vSwitch/VDS** is the standard for VMware environments
5. **Hyper-V Virtual Switch** integrates tightly with Windows
6. **Performance optimization** includes jumbo frames, hardware offload, SR-IOV
7. **Security features** include port security, VLAN isolation, DHCP snooping
8. **OVS supports OpenFlow** for SDN and network automation
9. **DPDK acceleration** can dramatically improve performance
10. **Always check statistics** when troubleshooting performance issues

## 🔗 Further Reading

- [Open vSwitch Documentation](https://www.openvswitch.org/support/dist-docs/)
- [Open vSwitch GitHub](https://github.com/openvswitch/ovs)
- [OVS vs Linux Bridge](https://docs.openvswitch.org/en/latest/faq/general/)
- [OVS DPDK Documentation](https://docs.openvswitch.org/en/latest/howto/dpdk/)
- [VMware vSwitch Documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.networking.doc/GUID-67593237-5D9A-43A2-9B50-897C71F6395A.html)
- [VMware Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.networking.doc/GUID-91914A13-0634-4545-9C9F-03A23B52555C.html)
- [Hyper-V Virtual Switch](https://docs.microsoft.com/en-us/virtualization/hyper-v-virtual-switch/virtual-switch)
- [Hyper-V Networking Guide](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/networking-guide)
- [Linux Bridge Documentation](https://www.kernel.org/doc/html/latest/networking/bridge.html)
- [Virtual Switch Performance](https://www.vmware.com/content/dam/digitalmarketing/2016/09/vmw-vsphere-6-5-networking-performance.pdf)

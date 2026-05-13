# Overlay Networks

> Understanding virtual network encapsulation for distributed systems

---

## 🎯 Purpose

Overlay networks create **virtual networks** on top of existing physical networks. They enable communication between isolated systems (containers, VMs, hosts) as if they were on the same local network, even when they're distributed across different physical locations.

**Key Benefits:**
- **Multi-host communication** without complex routing
- **Isolation** between different virtual networks
- **Flexibility** in network design
- **Compatibility** with existing infrastructure
- **No changes** to underlying physical network required

## 🏗️ How Overlay Networks Work

### Basic Concept

Overlay networks **encapsulate** packets from virtual networks inside **tunnel packets** that can traverse the underlying physical network.

```
┌──────────────────┐       ┌──────────────────┐
│   Host 1          │       │   Host 2          │
├──────────────────┤       ├──────────────────┤
│                  │       │                  │
│  ┌────────────┐ │       │  ┌────────────┐ │
│  │  Container  │ │       │  │  Container  │ │
│  │   A        │ │       │  │   B        │ │
│  │ 10.1.0.2  │ │       │  │ 10.1.0.3  │ │
│  └─────┬─────┘ │       │  └─────┬─────┘ │
│        │       │       │        │       │
│  ┌─────▼─────┐ │       │  ┌─────▼─────┐ │
│  │ Overlay   │ │       │  │ Overlay   │ │
│  │ Interface │ │       │  │ Interface │ │
│  └─────┬─────┘ │       │  └─────┬─────┘ │
│        │       │       │        │       │
│        └───────┼───────┘       └───────┼───────┘
│                │                       │
│                └───────────┬───────────┘
│                            ▼ (Tunnel)
│              ┌─────────────────────┐
│              │   Physical Network   │
│              └─────────────────────┘
└──────────────────┘       └──────────────────┘
```

### Encapsulation Process

1. **Container sends packet** to destination (10.1.0.3)
2. **Overlay driver intercepts** the packet
3. **Looks up destination** in mapping table
4. **Encapsulates** packet in tunnel protocol
5. **Forwards** encapsulated packet to destination host
6. **Remote overlay driver decapsulates** and delivers to container

### Mapping Database

Overlay networks maintain a **mapping database** that tracks:
- Which container (virtual IP) is on which host (physical IP)
- How to reach each host

**Example Mapping:**
```
Virtual IP    Physical IP    Host
10.1.0.2     192.168.1.10   host1
10.1.0.3     192.168.1.11   host2
10.1.0.4     192.168.1.12   host3
```

## 🌐 Overlay Network Protocols

### VXLAN (Virtual Extensible LAN)

**RFC**: [RFC 7348](https://tools.ietf.org/html/rfc7348)

**Purpose**: Extend Layer 2 networks over Layer 3 infrastructure

**VXLAN Header:**
```
+----------------+----------------+------------+----------------+---------+------------+
| Outer Ethernet| Outer IP       | UDP        | VXLAN Header  | Inner  | Payload    |
| Header        | Header         | Header     | (8 bytes)      | Ethernet|            |
+----------------+----------------+------------+----------------+---------+------------+

VXLAN Header:
+--------+--------+--------+--------+
| Flags |Reserved|  VNI   |Reserved|
| (8b)  | (24b)  | (24b)  | (8b)   |
+--------+--------+--------+--------+
```

**Fields:**
- **Flags**: Control bits (8 bits)
  - Bit 0: Reserved
  - Bits 1-7: Reserved
- **VNI (VXLAN Network Identifier)**: 24-bit identifier for the virtual network (supports ~16M networks)
- **Reserved**: 24 bits

**Port**: UDP 4789 (default, assigned by IANA)

**MTU Considerations:**
- Original Ethernet MTU: 1500 bytes
- VXLAN adds: 8 (VXLAN) + 20 (IP) + 8 (UDP) = 36 bytes
- **Recommended MTU**: 1550-1600 bytes (1500 + overhead)
- **Jumbo frames**: 9000 bytes (recommended for VXLAN)

**Advantages:**

| Feature | Benefit |
|---------|---------|
| 24-bit VNI | Supports ~16M virtual networks |
| UDP-based | Works over any IP network |
| Layer 2 extension | Transparent to applications |
| Standardized | Widely supported |

**Disadvantages:**

| Issue | Concern |
|-------|---------|
| Encapsulation overhead | Reduces effective MTU |
| Broadcast/multicast | Requires special handling |
| Tunneling | Adds latency |

**Example Packet Flow:**

```
Original Frame:
+----------------+----------------+------------+
| Dest MAC      | Source MAC    | Ethernet   |
| (6B)          | (6B)          | Type (2B)  |
+----------------+----------------+------------+
| IP Header     | TCP/UDP Data  | FCS (4B)   |
+----------------+----------------+------------+

VXLAN Encapsulated:
+----------------+----------------+------------+----------------+---------+----------------+------------+
| Outer Dest MAC| Outer Source   | Outer      | UDP (8B)      | VXLAN    | Inner Ethernet | Payload    | FCS
| (Host 2 MAC)   | MAC (Host 1)   | IP Header  | (Dest:4789)   | Header   | Frame           |            |
+----------------+----------------+------------+----------------+---------+----------------+------------+
```

### VXLAN in Docker/Flannel

**Docker uses VXLAN** for overlay networks:

```bash
# Create VXLAN overlay network in Docker
docker network create my-vxlan --driver overlay --opt com.docker.network.driver.overlay.vxlanid=1000

# Check VXLAN interface
docker network inspect my-vxlan | grep vxlan
```

**Flannel VXLAN Backend:**
```yaml
# Flannel configuration for VXLAN
net-conf.json:
{
  "Network": "10.1.0.0/16",
  "Backend": {
    "Type": "vxlan",
    "VNI": 1,
    "Port": 4789,
    "GBP": false,
    "DirectRouting": false
  }
}
```

### Geneve (Generic Network Virtualization Encapsulation)

**RFC**: [RFC 8926](https://tools.ietf.org/html/rfc8926)

**Purpose**: General-purpose encapsulation protocol designed to be flexible and extensible

**Geneve Header:**
```
+----------------+----------------+------------+----------------+----------------+
| Outer Ethernet| Outer IP       | UDP        | Geneve Header | Inner Packet  |
| Header        | Header         | Header     | (Variable)    |                |
+----------------+----------------+------------+----------------+----------------+

Geneve Header (minimum 8 bytes):
+--------+--------+--------+--------+--------+--------+
| Version|Opt Len |OAM Flag| Critical| Reserved | Protocol Type |
| (2b)   | (6b)   | (1b)   | (1b)    | (6b)    | (16b)        |
+--------+--------+--------+--------+--------+--------+
| TLV Options (variable length)                         |
+-----------------------------------------------------+
| Payload                                               |
+-----------------------------------------------------+
```

**Fields:**
- **Version**: Protocol version (currently 0)
- **Opt Len**: Length of optional TLVs (in 4-byte words)
- **OAM Flag**: Operations, Administration, and Maintenance
- **Critical**: If set, drop packet if not understood
- **Reserved**: For future use
- **Protocol Type**: Type of encapsulated protocol
- **TLV Options**: Type-Length-Value pairs for extensibility

**Port**: UDP 6081 (default)

**Advantages over VXLAN:**

| Feature | VXLAN | Geneve |
|---------|-------|--------|
| Extensibility | Limited | ✅ TLV options |
| VNI Size | 24-bit | Variable (in TLV) |
| Header Size | Fixed (8B) | Variable |
| Flexibility | Low | ✅ High |
| Standardization | ✅ RFC 7348 | ✅ RFC 8926 |

**Use Cases:**
- OpenStack Virtual Networking
- Kubernetes (some implementations)
- VMware NSX-T
- Custom overlay networks

### GRE (Generic Routing Encapsulation)

**RFC**: [RFC 2784](https://tools.ietf.org/html/rfc2784), [RFC 2890](https://tools.ietf.org/html/rfc2890)

**Purpose**: Simple encapsulation protocol for tunneling

**GRE Header:**
```
+--------+--------+--------+--------+
| C | Reserved | Version | Protocol|
| (1b)| (3b)    | (3b)   | (16b)   |
+--------+--------+--------+--------+
| Checksum (optional)              |
+--------------------------------+
| Reserved                         |
+--------------------------------+
| Key (optional)                   |
+--------------------------------+
| Sequence Number (optional)      |
+--------------------------------+
| Routing (optional)              |
+--------------------------------+
```

**Fields:**
- **C (Checksum)**: Checksum present flag
- **Reserved**: Must be zero
- **Version**: GRE version (0)
- **Protocol**: Encapsulated protocol type
- **Key**: 32-bit key for identifying tunnel
- **Sequence Number**: For ordering packets

**Port**: Protocol 47 (IP protocol number)

**Types:**
- **Basic GRE**: No optional fields
- **GRE with Key**: Includes key field for tunnel identification
- **GRE with Checksum**: Includes checksum for verification

**Advantages:**
- Simple, widely supported
- Low overhead (24 bytes minimum)
- Works at Layer 3

**Disadvantages:**
- No built-in security
- No support for multicast in basic mode
- Limited extensibility

**Use Cases:**
- PPTP VPN (historical)
- Simple point-to-point tunnels
- Legacy systems

### IPIP (IP-in-IP)

**RFC**: [RFC 2003](https://tools.ietf.org/html/rfc2003)

**Purpose**: Encapsulate IP packets within IP packets

**IPIP Header:**
```
+----------------+----------------+----------------+
| Outer IP      | Inner IP      | Payload        |
| Header        | Header        |                |
+----------------+----------------+----------------+
```

**Protocol**: IP protocol number 4

**Advantages:**
- ✅ Simplest possible encapsulation
- ✅ Minimal overhead (20 bytes)
- ✅ Widely supported

**Disadvantages:**
- ❌ No support for non-IP protocols
- ❌ No multiplexing (only one tunnel per outer IP pair)
- ❌ No built-in security

**Use Cases:**
- Simple IP tunneling
- Flannel IP-in-IP backend
- Legacy systems

### WireGuard

**Purpose**: Modern, secure VPN/tunneling protocol using UDP

**Note**: WireGuard is typically used for secure point-to-point connections rather than overlay networks, but it can be used to create secure overlays.

**Features:**
- **Encryption**: ChaCha20, Poly1305
- **Authentication**: BLAKE2s, Curve25519
- **Performance**: High throughput, low latency
- **Simplicity**: ~4000 lines of code
- **Port**: UDP 51820 (default)

**Use in Overlay Networks:**
- Cilium uses WireGuard for pod-to-pod encryption
- Can be used to create secure overlay tunnels

## 🔄 Protocol Comparison

| Protocol | Encapsulation | Port/Proto | VNI/ID | Overhead | Security | Extensibility | Use Case |
|----------|--------------|------------|--------|----------|----------|---------------|----------|
| **VXLAN** | UDP | 4789 | 24-bit VNI | 36B | ❌ No | ❌ Limited | Container networking, VMware NSX |
| **Geneve** | UDP | 6081 | Variable (TLV) | 8B+ | ❌ No | ✅ High | OpenStack, custom overlays |
| **GRE** | IP (Proto 47) | N/A | 32-bit Key | 24B+ | ❌ No | ❌ Limited | PPTP, simple tunnels |
| **IPIP** | IP (Proto 4) | N/A | None | 20B | ❌ No | ❌ No | Simple IP tunneling |
| **WireGuard** | UDP | 51820 | Public Key | ~60B | ✅ Yes | ❌ Limited | Secure tunnels, Cilium encryption |

## 🏛️ Overlay Network Use Cases

### 1. Container Networking (Docker/Kubernetes)

**Problem**: Containers on different hosts need to communicate as if on same network

**Solution**: Use VXLAN-based overlay (Docker overlay, Flannel, Calico in overlay mode)

**Example (Docker Swarm):**
```bash
# Initialize Swarm
docker swarm init

# Join workers
docker swarm join --token <token> <manager-ip>

# Create overlay network
docker network create my-overlay --driver overlay --attachable

# Deploy service
docker service create --name web --network my-overlay -p 80:80 nginx

# Scale service
docker service scale web=10
```

**Traffic Flow:**
```
Container A (Host 1) → VXLAN Encapsulation → Physical Network → VXLAN Decapsulation → Container B (Host 2)
```

### 2. Virtual Machine Networking (OpenStack, VMware)

**Problem**: VMs on different hypervisors need to be on same virtual network

**Solution**: Use VXLAN or Geneve for VM networking

**Example (OpenStack):**
- Neutron uses VXLAN/Geneve for virtual networks
- Each tenant gets isolated virtual network
- VMs can migrate between hosts without IP change

### 3. Cloud Networking (AWS, Azure, GCP)

**Problem**: Connect VPCs/VNets across regions or accounts

**Solution**: Cloud providers offer overlay networking

| Cloud | Overlay Technology |
|-------|-------------------|
| AWS | VPC Peering, Transit Gateway, VXLAN |
| Azure | Virtual Network Peering, VXLAN |
| GCP | VPC Network Peering, VXLAN |

### 4. SDN (Software Defined Networking)

**Problem**: Need flexible, programmable network overlays

**Solution**: Use SDN controllers with overlay protocols

**Examples:**
- OpenDaylight with VXLAN
- ONOS with various overlays
- VMware NSX with VXLAN/Geneve

### 5. Service Mesh (Istio, Linkerd)

**Problem**: Service-to-service communication with observability and security

**Solution**: Use overlay networking for service mesh data plane

**Example (Istio):**
- Uses Envoy sidecars
- Can use VXLAN or other tunneling for cross-host communication
- Provides L7 load balancing, mTLS, observability

## 🛠️ Implementing Overlay Networks

### Docker Overlay Network

**Create Overlay Network:**
```bash
# Prerequisite: Docker Swarm initialized
docker swarm init

# Create overlay network
docker network create my-overlay --driver overlay \
  --subnet 10.1.0.0/16 \
  --gateway 10.1.0.1 \
  --opt com.docker.network.driver.overlay.vxlanid=1000 \
  --attachable
```

**Verify Overlay Network:**
```bash
# List networks
docker network ls

# Inspect network
docker network inspect my-overlay

# Check VXLAN interface
docker network inspect my-overlay | grep vxlan

# On Linux host:
ip link show type vxlan
ip -d link show vxlan1
```

**Connect Containers:**
```bash
# Run container on overlay
docker run --name web1 --network my-overlay -d nginx

# Test connectivity from another container
docker run --rm --network my-overlay alpine ping web1
```

### Flannel with VXLAN

**Install Flannel:**
```bash
# For Kubernetes
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# For standalone
wget https://github.com/flannel-io/flannel/releases/download/v0.22.2/flannel-v0.22.2-linux-amd64.tar.gz
tar xzf flannel-*.tar.gz
cd flannel-*/
./flanneld --iface=eth0 --ip-masq --kube-subnet-mgr --iface-regex="^eth.*"
```

**Flannel VXLAN Configuration:**
```json
{
  "Network": "10.1.0.0/16",
  "Backend": {
    "Type": "vxlan",
    "VNI": 1,
    "Port": 4789,
    "GBP": false,
    "DirectRouting": false
  }
}
```

**Verify Flannel:**
```bash
# Check Flannel interface
ip link show flannel.1

# Check routes
ip route show

# Check VXLAN interface
ip -d link show flannel.1
```

### Linux VXLAN Configuration

**Create VXLAN Interface Manually:**
```bash
# Create VXLAN interface
ip link add vxlan100 type vxlan \
  id 100 \
  local 192.168.1.10 \
  remote 192.168.1.11 \
  dev eth0 \
  dstport 4789

# Bring up interface
ip link set vxlan100 up

# Assign IP address
ip addr add 10.1.0.1/24 dev vxlan100

# Add route to remote network
ip route add 10.1.1.0/24 dev vxlan100
```

**Configure VXLAN with Bridge:**
```bash
# Create bridge
ip link add br0 type bridge
ip link set br0 up

# Create VXLAN interface
ip link add vxlan100 type vxlan \
  id 100 \
  local 192.168.1.10 \
  dev eth0 \
  dstport 4789

# Attach VXLAN to bridge
ip link set vxlan100 master br0
ip link set vxlan100 up

# Create virtual interfaces for containers
ip link add veth0 type veth peer name veth1
ip link set veth0 master br0
ip link set veth0 up

# In container namespace:
ip link set veth1 up
ip addr add 10.1.0.2/24 dev veth1
ip route add default via 10.1.0.1
```

### Geneve Configuration

**Create Geneve Interface:**
```bash
# Create Geneve interface
ip link add geneve100 type geneve \
  id 100 \
  local 192.168.1.10 \
  remote 192.168.1.11 \
  dev eth0 \
  udp_port 6081

# Bring up interface
ip link set geneve100 up

# Assign IP address
ip addr add 10.1.0.1/24 dev geneve100
```

## 🎯 Overlay Network Performance

### Performance Factors

| Factor | Impact | Optimization |
|--------|--------|--------------|
| **MTU** | Larger MTU = better throughput | Use jumbo frames (9000) |
| **Encapsulation** | Adds overhead | Use protocols with less overhead |
| **Tunnel Endpoints** | More hops = more latency | Minimize tunnel hops |
| **Hardware Offload** | Improves performance | Use NICs with VXLAN offload |
| **Encryption** | Adds CPU overhead | Use hardware-accelerated encryption |
| **Network Topology** | Affects latency | Optimize physical network |

### Performance Comparison

| Protocol | Throughput | Latency | CPU Usage | Hardware Offload |
|----------|------------|---------|-----------|-------------------|
| **VXLAN** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ Yes (some NICs) |
| **Geneve** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ❌ Limited |
| **GRE** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ✅ Yes |
| **IPIP** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ✅ Yes |
| **VXLAN + HW Offload** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ | ✅ Yes |

### Benchmarking Overlay Networks

**Using iperf3:**
```bash
# On host 1 (server)
iperf3 -s -p 5201

# On host 2 (client)
iperf3 -c <host1-ip> -p 5201 -t 60 -i 10
```

**Compare with native:**
```bash
# Native performance (same host)
iperf3 -c 127.0.0.1 -p 5201

# Physical network performance
iperf3 -c <physical-ip> -p 5201

# Overlay network performance
iperf3 -c <overlay-ip> -p 5201
```

**Expected Results:**
- Native: ~Line rate
- Physical: ~Line rate (minus some overhead)
- Overlay: ~70-90% of line rate (depends on protocol, hardware)

### Optimizing Performance

1. **Increase MTU**: Use jumbo frames (9000 bytes)
   ```bash
   # For VXLAN
   ip link set vxlan100 mtu 9000
   
   # In Docker
   docker network create my-vxlan --driver overlay --opt com.docker.network.driver.mtu=1500
   ```

2. **Enable Hardware Offload**:
   ```bash
   # Check if NIC supports offload
   ethtool -k eth0 | grep vxlan
   
   # Enable offload
   ethtool -K eth0 rx-vxlan-offload on
   ethtool -K eth0 tx-vxlan-offload on
   ```

3. **Minimize Tunnel Hops**:
   - Use direct routing when possible
   - Avoid unnecessary encapsulation layers

4. **Use BGP instead of Overlay**:
   - Calico in BGP mode avoids overlay
   - Better performance for large clusters

5. **Enable GRO/GSO**:
   ```bash
   ethtool -K eth0 gro on
   ethtool -K eth0 gso on
   ```

## 🔐 Security Considerations

### Overlay Network Security Risks

| Risk | Description | Mitigation |
|------|-------------|------------|
| **Tunnel Spoofing** | Attacker injects packets into tunnel | Use authentication, encryption |
| **Eavesdropping** | Attacker reads encapsulated packets | Use encryption (IPSec, WireGuard) |
| **Denial of Service** | Flood overlay network with packets | Rate limiting, QoS |
| **VNI Exhaustion** | Attacker creates many VNIs | Limit VNI allocation |
| **MAC Spoofing** | Attacker impersonates VM/container | Port security, MAC filtering |
| **ARP Spoofing** | Attacker intercepts ARP requests | Use static ARP, port security |

### Security Best Practices

1. **Use Encryption**:
   - IPSec for VXLAN
   - WireGuard for modern deployments
   - MACsec for Layer 2 encryption

2. **Implement Authentication**:
   - Use certificates for tunnel endpoints
   - Mutual authentication

3. **Network Segmentation**:
   - Use different VNIs for different tenants
   - Implement micro-segmentation

4. **Access Control**:
   - Firewall rules for tunnel endpoints
   - Restrict which hosts can create tunnels

5. **Monitoring**:
   - Monitor tunnel creation and teardown
   - Monitor for unusual traffic patterns

6. **Hardware Security**:
   - Use NICs with secure boot
   - Disable unused offload features

**Example: VXLAN with IPSec:**
```bash
# Create IPSec tunnel first
ip xfrm state add src 192.168.1.10 dst 192.168.1.11 proto esp spi 0x100 reqid 1 mode tunnel
ip xfrm policy add src 192.168.1.10 dst 192.168.1.11 dir out tmpl src 192.168.1.10 dst 192.168.1.11 proto esp reqid 1 mode tunnel

# Then create VXLAN on top
ip link add vxlan100 type vxlan id 100 local 192.168.1.10 remote 192.168.1.11 dev eth0 dstport 4789
```

## 🛠️ Troubleshooting Overlay Networks

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **No connectivity** | Can't ping between hosts | Check VXLAN interface, routes, firewall |
| **MTU issues** | Packet fragmentation | Increase MTU, use jumbo frames |
| **VNI mismatch** | Packets dropped | Verify VNI configuration on both ends |
| **Firewall blocking** | Connection refused | Open UDP 4789 (VXLAN) or 6081 (Geneve) |
| **Encapsulation errors** | Packet drops | Check NIC offload settings |
| **ARP issues** | ARP requests not resolved | Check ARP suppression settings |
| **Broadcast issues** | Broadcast packets not delivered | Configure BUM (Broadcast, Unknown, Multicast) handling |

### Troubleshooting Commands

```bash
# Check VXLAN interface
ip link show type vxlan
ip -d link show vxlan100

# Check VXLAN statistics
cat /proc/net/vxlan/vxlan100

# Check routes
ip route show
ip route get 10.1.0.3

# Check ARP cache
ip neigh show

# Check firewall rules
sudo iptables -L -n -v
sudo ip6tables -L -n -v

# Capture VXLAN traffic
tcpdump -i vxlan100 -n -vv

# Capture on physical interface
tcpdump -i eth0 -n -vv udp port 4789

# Check Docker overlay network
docker network inspect my-overlay

# Check Flannel logs
journalctl -u flanneld -f

# Check kernel logs
dmesg | grep vxlan
```

### VXLAN Statistics

```bash
# View VXLAN statistics
cat /proc/net/vxlan/vxlan<id>

# Example output:
Interface: vxlan100
  Device: eth0
  Remote IP: 192.168.1.11
  VNI: 100
  RX: bytes packets errors dropped
  TX: bytes packets errors
  
# Reset statistics
echo 0 > /proc/net/vxlan/vxlan100
```

### Checking Connectivity

```bash
# From host 1, ping through VXLAN
ping -I vxlan100 10.1.0.3

# Check ARP resolution
ip neigh show dev vxlan100

# Check if remote is reachable
ip route get 10.1.0.3

# Test UDP connectivity (VXLAN uses UDP)
nc -u -v -w 3 192.168.1.11 4789
```

## 🎯 Key Takeaways

1. **Overlay networks create virtual networks** on top of physical networks
2. **VXLAN** is the most common overlay protocol (24-bit VNI, UDP 4789)
3. **Geneve** is more flexible but less widely supported
4. **GRE, IPIP** are simpler but have limitations
5. **Encapsulation adds overhead** (MTU, CPU, latency)
6. **Hardware offload** can significantly improve performance
7. **Security is critical** - use encryption and authentication
8. **MTU must be configured properly** to avoid fragmentation
9. **Broadcast/Unknown/Multicast** (BUM) traffic requires special handling
10. **Troubleshooting requires understanding** both overlay and underlay networks

## 🔗 Further Reading

- [VXLAN RFC 7348](https://tools.ietf.org/html/rfc7348)
- [Geneve RFC 8926](https://tools.ietf.org/html/rfc8926)
- [GRE RFC 2784](https://tools.ietf.org/html/rfc2784)
- [IPIP RFC 2003](https://tools.ietf.org/html/rfc2003)
- [WireGuard Whitepaper](https://www.wireguard.com/papers/wireguard.pdf)
- [Docker Overlay Networks](https://docs.docker.com/network/overlay/)
- [Flannel Documentation](https://docs.flannel.io/)
- [Kubernetes Overlay Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model)
- [VXLAN Deep Dive](https://cummulusnetworks.com/blog/vxlan-deep-dive/)
- [Linux VXLAN Implementation](https://www.kernel.org/doc/html/latest/networking/vxlan.html)
- [Overlays in the Linux Kernel](https://lwn.net/Articles/580893/)

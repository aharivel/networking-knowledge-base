# CNI Plugins (Container Network Interface)

> Kubernetes networking plugins that provide pod-to-pod communication

---

## 🎯 Purpose

CNI (Container Network Interface) is a **standard interface** between container runtimes (like Kubernetes) and networking plugins. It defines how networking should be configured for containers/pods.

**CNI provides:**
- **Standard interface** for container networking
- **Pluggable architecture** (multiple implementations)
- **Pod-to-pod communication** across nodes
- **Network policy enforcement**
- **IP address management**

## 🏗️ CNI vs Docker Networking

| Feature | Docker Networking | CNI |
|---------|-------------------|-----|
| **Runtime** | Docker | Kubernetes, Containerd, CRI-O |
| **Scope** | Single host | Cluster-wide |
| **Interface** | Docker-specific | Standard (CNI spec) |
| **Plugins** | Built-in drivers | External plugins |
| **Configuration** | Docker CLI | Kubernetes manifests |

## 📋 CNI Specification

CNI defines a simple interface with **3 main operations**:

1. **Add**: Configure networking for a container
2. **Delete**: Clean up networking when container is removed
3. **Get**: Get current network configuration

### CNI Configuration Format

```json
{
  "cniVersion": "0.3.1",
  "name": "my-network",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.1.0.0/16",
    "rangeStart": "10.1.0.0",
    "rangeEnd": "10.1.255.255",
    "gateway": "10.1.0.1"
  },
  "dns": {
    "nameservers": ["8.8.8.8", "8.8.4.4"],
    "domain": "example.com",
    "search": ["example.com"]
  }
}
```

### CNI Environment Variables

CNI plugins receive information via environment variables:

| Variable | Description |
|----------|-------------|
| `CNI_COMMAND` | ADD, DEL, or GET |
| `CNI_CONTAINERID` | Unique container ID |
| `CNI_NETNS` | Network namespace path |
| `CNI_IFNAME` | Interface name (e.g., eth0) |
| `CNI_PATH` | Path to CNI plugin executables |
| `CNI_ARGS` | Additional arguments (JSON) |

## 🔧 CNI Plugin Types

### Main Plugin
- **Primary plugin** that handles the main networking functionality
- Implements `Add`, `Delete`, `Get` operations
- Examples: Flannel, Calico, Weave, Cilium

### IPAM (IP Address Management) Plugin
- **Responsible for IP address allocation**
- Can be separate or built into main plugin
- Examples: host-local, dhcp, static

### Meta Plugin
- **Combines multiple plugins** into a chain
- Allows composing functionality from multiple plugins

## 🌟 Popular CNI Plugins

### Flannel

**Developer**: CoreOS (now Red Hat)

**Type**: Overlay network

**Features:**
- Simple, easy to set up
- Multiple backend options
- Works on any cloud or bare metal
- Good performance

**Architecture:**
```
┌──────────────────┐       ┌──────────────────┐
│   Kubernetes      │       │   Kubernetes      │
│   Node 1          │       │   Node 2          │
├──────────────────┤       ├──────────────────┤
│                  │       │                  │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │   Pod A    │  │       │  │   Pod B    │  │
│  │ 10.1.0.2   │  │       │  │ 10.1.0.3   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │  flannel0   │◄───────►│  │  flannel0   │  │
│  │ (Overlay)   │  │       │  │ (Overlay)   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌─────────────┐ │       │  ┌─────────────┐ │
│  │   Physical   │ │       │  │   Physical   │ │
│  │   Network    │ │       │  │   Network    │ │
│  └─────────────┘ │       │  └─────────────┘ │
└──────────────────┘       └──────────────────┘
       │                         │
       └───────── Flannel Backend ────────┘
```

**Flannel Backends:**

| Backend | Description | Performance | Requirements |
|---------|-------------|------------|--------------|
| **vxlan** | VXLAN encapsulation | ⭐⭐⭐⭐ | Kernel 3.7+ |
| **host-gw** | Uses host's routing table | ⭐⭐⭐⭐⭐ | Layer 2 connectivity |
| **ipip** | IP-in-IP encapsulation | ⭐⭐⭐ | Kernel 3.2+ |
| **udp** | UDP encapsulation | ⭐⭐ | Works everywhere |

**Installation:**
```bash
# Install flannel CNI plugin
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Verify installation
kubectl get pods -n kube-system | grep flannel
```

**Configuration (kube-flannel.yml):**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: kube-flannel
        image: flannelcni/flannel:v0.22.2
        command: ["/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr"]
        volumeMounts:
        - name: cni-plugin
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/flannel/
      volumes:
      - name: cni-plugin
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        hostPath:
          path: /etc/flannel
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| ✅ Simple to set up | ❌ Limited features (no network policies) |
| ✅ Works on all platforms | ❌ Performance overhead (overlay) |
| ✅ Multiple backend options | ❌ No built-in encryption |
| ✅ Mature and stable | ❌ Less flexible than Calico/Cilium |

### Calico

**Developer**: Tigera (formerly Metaswitch)

**Type**: Networking + Network Policy

**Features:**
- **BGP routing** (no overlay by default)
- **Network policies** (Kubernetes NetworkPolicy API)
- **High performance** (native routing)
- **Multi-cluster support**
- **Encryption** (optional WireGuard)

**Architecture:**

```
┌──────────────────┐       ┌──────────────────┐
│   Kubernetes      │       │   Kubernetes      │
│   Node 1          │       │   Node 2          │
├──────────────────┤       ├──────────────────┤
│                  │       │                  │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │   Pod A    │  │       │  │   Pod B    │  │
│  │ 10.1.0.2   │  │       │  │ 10.1.0.3   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │   caliXX   │  │       │  │   caliXX   │  │
│  │ (veth)     │  │       │  │ (veth)     │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│  ┌─────▼──────┐  │       │  ┌─────▼──────┐  │
│  │  BGP Router │  │◄──────►│  │  BGP Router │  │
│  └─────┬──────┘  │  BGP   │  └─────┬──────┘  │
│        │         │ Peering│        │         │
│        ▼         │       │        ▼         │
│  ┌─────────────┐ │       │  ┌─────────────┐ │
│  │   Physical   │ │       │  │   Physical   │ │
│  │   Network    │ │       │  │   Network    │ │
│  └─────────────┘ │       │  └─────────────┘ │
└──────────────────┘       └──────────────────┘
```

**Calico Components:**

| Component | Purpose |
|-----------|---------|
| **felix** | Main agent, runs on each node |
| **bird** | BGP router (optional, can use host's BGP) |
| **confd** | Configuration management |
| **typha** | Scale BGP control plane |
| **kube-controllers** | Kubernetes integration |

**Installation:**

**Using Calico as CNI plugin:**
```bash
# Install Calico with kubectl
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Or use operator-based installation
kubectl apply -f https://docs.projectcalico.org/manifests/calico-operator.yaml
kubectl apply -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```

**Configuration (calico.yaml):**
```yaml
# Calico Custom Resource Definition
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  cidr: 192.168.0.0/16
  ipipMode: Never
  natOutgoing: true
  nodeSelector: all()
  vxlanMode: Always
---
apiVersion: projectcalico.org/v3
kind:FelixConfiguration
metadata:
  name: default
spec:
  bpfEnabled: false
  logSeverityScreen: Info
  reportingInterval: 0
```

**Network Policy Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-except-frontend
  namespace: my-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 3306
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| ✅ Native routing (no overlay by default) | ❌ More complex setup |
| ✅ Full Kubernetes NetworkPolicy support | ❌ BGP knowledge required for best performance |
| ✅ High performance | ❌ More moving parts |
| ✅ Network policies | ❌ Requires BGP peering in some configurations |
| ✅ Multi-cluster support | ❌ Larger footprint |

### Cilium

**Developer**: Isovalent (originally created by Google, Cisco, etc.)

**Type**: eBPF-based networking, security, observability

**Features:**
- **eBPF-based** (no kernel modules, runs in user space)
- **Kubernetes-native** (designed for K8s)
- **Network policies** (L3-L7 filtering)
- **Load balancing** (replaces kube-proxy)
- **Encryption** (WireGuard-based)
- **Observability** (metrics, tracing, visibility)
- **Multi-cluster** (ClusterMesh)

**Architecture:**

```
┌──────────────────┐       ┌──────────────────┐
│   Kubernetes      │       │   Kubernetes      │
│   Node 1          │       │   Node 2          │
├──────────────────┤       ├──────────────────┤
│                  │       │                  │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │   Pod A    │  │       │  │   Pod B    │  │
│  │ 10.1.0.2   │  │       │  │ 10.1.0.3   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │  eBPF       │  │       │  │  eBPF       │  │
│  │  Programs   │  │       │  │  Programs   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│  ┌─────▼──────┐  │       │  ┌─────▼──────┐  │
│  │  Linux      │  │       │  │  Linux      │  │
│  │  Network    │◄───────►│  │  Network    │  │
│  │  Stack      │  │       │  │  Stack      │  │
│  └─────────────┘  │       │  └─────────────┘  │
└──────────────────┘       └──────────────────┘
       │                         │
       └───────── eBPF + BGP ────────┘
```

**Cilium Components:**

| Component | Purpose |
|-----------|---------|
| **cilium-agent** | Main agent, runs on each node |
| **cilium-operator** | Manages Cilium CRDs |
| **cilium-cli** | CLI tool for debugging |
| **Hubble** | Observability layer (metrics, flows, traces) |

**Installation:**

**Quick Installation with Helm:**
```bash
# Add Cilium Helm repository
helm repo add cilium https://helm.cilium.io

# Install Cilium
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true
```

**Installation with kubectl:**
```bash
# Install Cilium CLI
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-amd64.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-amd64.tar.gz -C /usr/local/bin
rm cilium-linux-amd64.tar.gz{,.sha256sum}

# Install Cilium
cilium install
```

**Cilium Configuration:**

Cilium uses **Custom Resource Definitions (CRDs)** for configuration:

```yaml
# CiliumConfig
apiVersion: "cilium.io/v2"
kind: CiliumConfig
metadata:
  name: default
spec:
  debug: false
  enableIPv4: true
  enableIPv6: false
  kubeProxyReplacement: strict
  bpf:
    masquerade: true
    hostRouting: true
```

**Network Policy Example (L7):**
```yaml
apiVersion: "cilium.io/v2"
kind: CiliumNetworkPolicy
metadata:
  name: http-policy
  namespace: my-namespace
spec:
  endpointSelector:
    matchLabels:
      app: my-app
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: GET
          path: "/api/v1"
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| ✅ eBPF-based (high performance) | ❌ Complex technology (eBPF) |
| ✅ Replaces kube-proxy (simpler) | ❌ Requires Linux kernel 4.9.17+ |
| ✅ L7 network policies | ❌ More memory usage |
| ✅ Built-in observability (Hubble) | ❌ Steeper learning curve |
| ✅ Encryption (WireGuard) | ❌ Limited Windows support |
| ✅ Multi-cluster (ClusterMesh) | ❌ Dependency on eBPF tooling |

### Weave Net

**Developer**: Weaveworks

**Type**: Overlay network

**Features:**
- Simple to install and use
- Encryption (optional)
- Network policies
- Works on any cloud or bare metal
- Good for small to medium clusters

**Architecture:**
```
┌──────────────────┐       ┌──────────────────┐
│   Kubernetes      │       │   Kubernetes      │
│   Node 1          │       │   Node 2          │
├──────────────────┤       ├──────────────────┤
│                  │       │                  │
│  ┌────────────┐  │       │  ┌────────────┐  │
│  │   Pod A    │  │       │  │   Pod B    │  │
│  │ 10.1.0.2   │  │       │  │ 10.1.0.3   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌────────────┐  │◄──────►│  ┌────────────┐  │
│  │  weave      │  │ Encaps.│  │  weave      │  │
│  │ (Overlay)   │  │ (UDP)   │  │ (Overlay)   │  │
│  └─────┬──────┘  │       │  └─────┬──────┘  │
│        │         │       │        │         │
│        ▼         │       │        ▼         │
│  ┌─────────────┐ │       │  ┌─────────────┐ │
│  │   Physical   │ │       │  │   Physical   │ │
│  │   Network    │ │       │  │   Network    │ │
│  └─────────────┘ │       │  └─────────────┘ │
└──────────────────┘       └──────────────────┘
```

**Installation:**
```bash
# Install Weave Net
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

# Verify installation
kubectl get pods -n kube-system | grep weave
```

**Pros and Cons:**

| Pros | Cons |
|------|------|
| ✅ Simple to install | ❌ Overlay network (performance overhead) |
| ✅ Works everywhere | ❌ Limited features compared to Calico/Cilium |
| ✅ Encryption support | ❌ Smaller community than Calico |
| ✅ Network policies | ❌ Less actively developed |
| ✅ Good for beginners | ❌ Overlay encapsulation overhead |

## 📊 CNI Plugin Comparison

| Feature | Flannel | Calico | Cilium | Weave Net |
|---------|---------|--------|--------|-----------|
| **Type** | Overlay | BGP/Routing | eBPF | Overlay |
| **Overlay** | ✅ Yes (VXLAN) | ❌ Optional | ❌ No | ✅ Yes (UDP) |
| **BGP Support** | ❌ No | ✅ Yes | ❌ No (uses eBPF) | ❌ No |
| **Network Policies** | ❌ No | ✅ Yes (K8s API) | ✅ Yes (L3-L7) | ✅ Yes (K8s API) |
| **Encryption** | ❌ No | ✅ Optional (IPSec) | ✅ Yes (WireGuard) | ✅ Optional |
| **Performance** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Ease of Use** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **K8s Integration** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **IPAM** | ✅ Host-local | ✅ Host-local | ✅ Cluster-pool | ✅ Host-local |
| **Multi-cluster** | ❌ No | ✅ Yes | ✅ Yes | ❌ No |
| **Observability** | ❌ Basic | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ (Hubble) | ⭐⭐⭐ |
| **Load Balancing** | ❌ No | ✅ Yes | ✅ Yes (replaces kube-proxy) | ❌ No |

### When to Use Which

| Use Case | Recommended CNI | Why |
|----------|----------------|-----|
| **Simple overlay network** | Flannel | Easy to set up, works everywhere |
| **High performance, native routing** | Calico | BGP-based, no overlay |
| **eBPF-based, modern** | Cilium | Best performance, L7 policies |
| **Encryption by default** | Cilium | Built-in WireGuard |
| **Network policies** | Calico or Cilium | Full Kubernetes NetworkPolicy support |
| **Beginners** | Weave Net or Flannel | Simple installation |
| **Multi-cluster** | Calico or Cilium | Built-in support |
| **Observability** | Cilium | Hubble provides metrics, tracing, flows |

## 🔧 Installing and Configuring CNI

### CNI Installation Process

1. **Install CNI binaries** to `/opt/cni/bin/`
2. **Create CNI configuration** in `/etc/cni/net.d/`
3. **Configure kubelet** to use CNI (Kubernetes does this automatically)

```bash
# CNI directory structure
/opt/cni/bin/        # CNI plugin executables
/etc/cni/net.d/      # CNI configuration files
/etc/cni/net.d/00-<plugin>.conflist  # Main config (sorted alphabetically)
```

### Example: Manual CNI Configuration

**Create CNI config (`/etc/cni/net.d/10-flannel.conflist`):**
```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

**Verify CNI configuration:**
```bash
# List CNI plugins
ls /opt/cni/bin/

# View CNI configuration
cat /etc/cni/net.d/*

# Check kubelet CNI configuration
ps aux | grep kubelet | grep cni
```

### CNI Configuration with Multiple Plugins

Use **meta plugins** to chain functionality:

```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "Info",
      "datastore_type": "kubernetes",
      "nodename": "node1",
      "mtu": 1500,
      "ipam": {
        "type": "calico-ipam"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {
        "bandwidth": true
      }
    }
  ]
}
```

## 🌐 CNI and Kubernetes Networking Model

Kubernetes requires that CNI plugins satisfy the **Kubernetes networking model**:

1. **All Pods can communicate** with all other Pods without NAT
2. **All Nodes can communicate** with all Pods without NAT
3. **Pods on a Node** can see themselves via their Pod IP
4. **The IP that a Pod sees** for itself is the same IP that others see

### Kubernetes Networking Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────────┐       ┌──────────────────┐           │
│  │     Control       │       │     Worker Node 1  │           │
│  │      Plane        │       ├──────────────────┤           │
│  │                  │       │  ┌────────────┐  │           │
│  │  ┌────────────┐ │       │  │   Pod A    │  │           │
│  │  │ API Server │ │       │  │ 10.1.0.2   │  │           │
│  │  └────────────┘ │       │  └────┬───────┘  │           │
│  │  ┌────────────┐ │       │       │        │           │
│  │  │ etcd       │ │       │       ▼        │           │
│  │  └────────────┘ │       │  ┌─────────────┐ │           │
│  │  ┌────────────┐ │       │  │  CNI Plugin │ │           │
│  │  │ Scheduler  │ │       │  │  (Flannel)  │ │           │
│  │  └────────────┘ │       │  └─────────────┘ │           │
│  │  ┌────────────┐ │       │       │        │           │
│  │  │ Controller │◄─┼───────┼───────►│ kubelet   │           │
│  │  │  Manager   │ │       │  └─────────────┘ │           │
│  │  └────────────┘ │       └──────────────────┘           │
│  └──────────────────┘                                          │
│          ▲                                                    │
│          │ CNI Config (via etcd/API)                          │
│          ▼                                                    │
│  ┌──────────────────┐       ┌──────────────────┐           │
│  │     Worker Node 2  │       │     Worker Node N  │           │
│  ├──────────────────┤       ├──────────────────┤           │
│  │  ┌────────────┐  │       │  ┌────────────┐  │           │
│  │  │   Pod B    │  │       │  │   Pod N    │  │           │
│  │  │ 10.1.0.3   │  │       │  │ 10.1.0.N   │  │           │
│  │  └────────────┘  │       │  └────────────┘  │           │
│  └──────────────────┘       └──────────────────┘           │
│                                                             │
│  All Pods can communicate with each other without NAT           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### kube-proxy vs CNI

| Component | Purpose | Implementation |
|-----------|---------|----------------|
| **kube-proxy** | Load balancing, service discovery | Runs as DaemonSet |
| **CNI Plugin** | Pod networking | Runs as DaemonSet or on each node |

**kube-proxy modes:**
- **userspace**: Original, uses iptables, slow
- **iptables**: Default, uses iptables rules
- **ipvs**: Uses IP Virtual Server (faster, better for large clusters)

**Cilium replaces kube-proxy** with eBPF-based load balancing:
- **Better performance** (no kernel modules, runs in user space)
- **Simpler** (fewer moving parts)
- **More features** (L7 load balancing, observability)

## 🛠️ CNI Troubleshooting

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Pods not getting IP** | Pod stuck in ContainerCreating | Check CNI plugin logs, kubelet logs |
| **Pods can't communicate** | Connection refused between pods | Check network policies, CNI configuration |
| **CNI plugin crashloop** | CNI pod keeps restarting | Check plugin logs, resource limits |
| **Network policy not working** | Pods can communicate despite policy | Check policy syntax, CNI support |
| **Performance issues** | High latency, low throughput | Check CNI type, consider BGP-based CNI |

### Troubleshooting Commands

```bash
# Check CNI pods
kubectl get pods -n kube-system | grep -E 'flannel|calico|cilium|weave'

# View CNI pod logs
kubectl logs -n kube-system <cni-pod> -f

# Check kubelet logs
journalctl -u kubelet -f

# Check CNI configuration
cat /etc/cni/net.d/*

# Check network interfaces on a node
ip a

# Check routes
ip route

# Check iptables rules
sudo iptables -L -n -v

# Test connectivity between pods
kubectl exec -it <pod1> -- ping <pod2-ip>
kubectl exec -it <pod1> -- curl -v http://<service>

# Check CNI plugin version
kubectl describe pod <cni-pod> -n kube-system | grep Image

# Check node networking
kubectl describe node <node-name>
```

### Debugging with Cilium CLI

```bash
# Install Cilium CLI
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
sudo tar xzvfC cilium-linux-amd64.tar.gz -C /usr/local/bin

# Check Cilium status
cilium status

# Check connectivity between pods
cilium connectivity test

# Check network policy
cilium policy get

# Trace a specific flow
cilium hubble trace -f "flow where source=10.1.0.2 and destination=10.1.0.3"
```

### Debugging with Calicoctl

```bash
# Install calicoctl
curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.26.1/calicoctl
chmod +x calicoctl

# Check Calico node status
calicoctl node status

# Check BGP peering
calicoctl bgp status

# Check IP pools
calicoctl ippool show

# Check network policies
calicoctl policy show
```

## 🎯 Key Takeaways

1. **CNI is a standard interface** between container runtimes and networking plugins
2. **Kubernetes uses CNI** for pod networking (not Docker networking)
3. **Main CNI plugins**: Flannel (simple), Calico (BGP), Cilium (eBPF), Weave Net (overlay)
4. **CNI plugins handle**: IPAM, pod-to-pod communication, network policies
5. **Flannel**: Simple overlay, good for beginners
6. **Calico**: BGP-based, high performance, network policies
7. **Cilium**: eBPF-based, modern, L7 policies, observability
8. **Weave Net**: Simple overlay, encryption support
9. **Cilium replaces kube-proxy** with eBPF-based load balancing
10. **Always check CNI pod logs** when troubleshooting networking issues

## 🔗 Further Reading

- [CNI Specification](https://github.com/containernetworking/cni)
- [CNI Plugins Repository](https://github.com/containernetworking/plugins)
- [Flannel Documentation](https://docs.flannel.io/)
- [Calico Documentation](https://docs.tigera.io/calico/latest/about/)
- [Cilium Documentation](https://docs.cilium.io/)
- [Weave Net Documentation](https://www.weave.works/docs/net/latest/)
- [Kubernetes Networking Model](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [CNI Comparison](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-choose-a-cni-plugin)
- [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

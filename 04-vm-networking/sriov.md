# SR-IOV (Single Root I/O Virtualization)

> Hardware-based virtualization for high-performance network and storage I/O

---

## 🎯 Purpose

**SR-IOV (Single Root I/O Virtualization)** is a PCI Express (PCIe) specification that allows a single physical PCIe device to appear as multiple **virtual devices** to guest operating systems. This enables near-native performance for virtual machines while maintaining isolation and security.

**Key Benefits:**
- **Near-native performance** (90-98% of bare metal)
- **Hardware acceleration** for I/O operations
- **Reduced CPU overhead** (bypasses hypervisor for certain operations)
- **Improved latency** and throughput
- **Better resource utilization** (single NIC can serve multiple VMs)
- **Isolation** between virtual functions

## 🏗️ SR-IOV Architecture

### PCIe Device Virtualization

Traditional virtualization requires the hypervisor to **emulate** or **virtualize** hardware devices, which adds overhead. SR-IOV introduces a new PCIe **capability structure** that allows a device to present multiple interfaces.

```
┌─────────────────────────────────────────────────────────────┐
│                    PCIe Device with SR-IOV                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    Physical Function (PF)                │  │
│  │  - Full PCIe device functionality                        │  │
│  │  - Manages and configures the device                       │  │
│  │  - Typically assigned to the hypervisor/host             │  │
│  │  - Can create and manage Virtual Functions (VFs)         │  │
│  └────────────────────────────┬──────────────────────────┘  │
│                                   │                              │
│                                   ▼                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  Virtual        │  │  Virtual        │  │  Virtual        │  │
│  │  Function 1     │  │  Function 2     │  │  Function N     │  │
│  │  (VF1)          │  │  (VF2)          │  │  (VFN)          │  │
│  │  - Lightweight  │  │  - Lightweight  │  │  - Lightweight  │  │
│  │  - Limited      │  │  - Limited      │  │  - Limited      │  │
│  │  - Dedicated to │  │  - Dedicated to │  │  - Dedicated to │  │
│  │    a single VM  │  │    a single VM  │  │    a single VM  │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                         △
                         │ PCIe Bus
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Physical PCIe Device                         │
│  (e.g., NIC, HBA, GPU, SmartNIC)                               │
└─────────────────────────────────────────────────────────────┘
```

### Physical Function (PF) vs Virtual Function (VF)

| Feature | Physical Function (PF) | Virtual Function (VF) |
|---------|-------------------------|------------------------|
| **Full Device Access** | ✅ Yes | ❌ No (subset) |
| **Configuration** | Can configure entire device | Limited configuration |
| **Resource Allocation** | Full device resources | Allocated subset of resources |
| **Assignment** | Hypervisor/host | Guest VM |
| **Number** | Typically 1 per device | Multiple (up to hundreds) |
| **Driver** | Full-featured | Lightweight |
| **Functionality** | Complete | Restricted |

### SR-IOV in Virtualization Stack

```
┌─────────────────────────────────────────────────────────────┐
│                         Hypervisor                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    SR-IOV Driver                         │  │
│  │  - Manages PFs and VF allocation                        │  │
│  │  - Configures virtualization capabilities               │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐    ┌──────────────────┐    ┌─────────┐  │
│  │      VM 1        │    │      VM 2        │    │  Host   │  │
│  │                  │    │                  │    │         │  │
│  │  ┌────────────┐  │    │  ┌────────────┐  │    │  ┌─────┐│  │
│  │  │ VF Driver   │  │    │  │ VF Driver   │  │    │  │ PF ││  │
│  │  │ (e.g., ixgbevf│  │    │  │ (e.g., ixgbevf│  │    │  │Driver││  │
│  │  └────────────┘  │    │  └────────────┘  │    │  └─────┘│  │
│  │  ┌────────────┐  │    │  ┌────────────┐  │    │  ┌─────┐│  │
│  │  │ VF         │  │    │  │ VF         │  │    │  │ PF ││  │
│  │  │ (Virtual   │  │    │  │ (Virtual   │  │    │  │    ││  │
│  │  │  NIC)      │◄──┼────┼──►│  NIC)      │  │    │  │    ││  │
│  │  └────────────┘  │    │  └────────────┘  │    │  └─────┘│  │
│  └──────────────────┘    └──────────────────┘    └─────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    Physical NIC (PF)                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ VF1         │  │ VF2         │  │ ...         │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    PCIe Switch                         │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 📋 SR-IOV Specification

### PCIe SR-IOV Capability Structure

The SR-IOV capability is defined in the **PCIe specification** (ECN for PCI Express Base Specification 2.0, and integrated into PCIe 3.0+).

**Capability Structure:**
```
Offset  Field               Description
─────  ─────────────────── ───────────────────────────────────
0x00   SR-IOV Capabilities  Capabilities register
0x04   SR-IOV Control       Control register
0x08   SR-IOV Status         Status register
0x0C   Initial VF Offset    Starting offset for VF registers
0x10   Total VFs           Number of VFs supported
0x12   NumVFs              Number of VFs enabled
0x14   Function Dependency  VF dependency information
0x18   VF Offset Stride     Offset between VF register blocks
0x1C   VF Device ID         Device ID for VFs
...    ...                 ...
```

### Capabilities Register

| Bit | Name | Description |
|-----|------|-------------|
| 0 | VF Migration Capable | Supports VF migration |
| 1 | ARI Capable | Supports Alternate Routing ID |
| 2 | VF Migration Interrupt | VF migration generates interrupt |
| 3-31 | Reserved | - |

### Control Register

| Bit | Name | Description |
|-----|------|-------------|
| 0 | VF Enable | Enable VF creation |
| 1 | VF Migration Enable | Enable VF migration |
| 2 | VF Memory Space Enable | Enable VF memory space |
| 3-15 | Reserved | - |
| 16-31 | VF Enable Count | Number of VFs to enable |

### Status Register

| Bit | Name | Description |
|-----|------|-------------|
| 0 | VF Migration Status | VF migration in progress |
| 1-31 | Reserved | - |

## 🎯 SR-IOV Use Cases

### 1. Network Virtualization (NIC SR-IOV)

**Problem**: Traditional virtual switches add latency and CPU overhead for network I/O

**Solution**: Assign Virtual Functions directly to VMs for near-native performance

**Benefits:**
- **Lower latency** (bypasses virtual switch for most traffic)
- **Higher throughput** (reduced CPU overhead)
- **Better CPU utilization** (hypervisor doesn't need to process every packet)
- **Scalability** (single NIC can support many VMs)

**Typical Configuration:**
```
┌─────────────────────────────────────────────────────────────┐
│                    Hypervisor Host                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                 │
│  │     VM 1        │    │     VM 2        │                 │
│  │   (10 Gbps)     │    │   (10 Gbps)     │                 │
│  └────────┬────────┘    └────────┬────────┘                 │
│           │                     │                            │
│  ┌────────▼────────┐    ┌────────▼────────┐                 │
│  │   VF1 (10%)     │    │   VF2 (10%)     │                 │
│  └────────┬────────┘    └────────┬────────┘                 │
│           │                     │                            │
│           └─────────────────────┼────────────────┘           │
│                             │    PF (80%)                       │
│                             ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                Physical 100G NIC                          │  │
│  │  - PF: 80% bandwidth (management, control)              │  │
│  │  - VF1: 10% bandwidth (VM 1)                            │  │
│  │  - VF2: 10% bandwidth (VM 2)                            │  │
│  └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2. Storage Virtualization (HBA SR-IOV)

**Problem**: Virtualized storage has high latency and CPU overhead

**Solution**: Assign Virtual Functions to VMs for direct storage access

**Benefits:**
- **Direct storage access** (bypasses hypervisor storage stack)
- **Lower latency** for storage operations
- **Higher IOPS** (input/output operations per second)
- **Better CPU efficiency**

**Use Cases:**
- Database servers
- High-performance storage arrays
- Virtual SAN (VSAN) configurations

### 3. GPU Virtualization (GPU SR-IOV)

**Problem**: GPU virtualization traditionally requires passthrough or mediation

**Solution**: Use GPU SR-IOV (e.g., Intel GVT-g, NVIDIA MIG) to share GPU resources

**Benefits:**
- **Multiple VMs** can share a single GPU
- **Near-native GPU performance** for each VM
- **Isolation** between VMs
- **Flexible resource allocation**

**GPU SR-IOV Implementations:**
| Vendor | Technology | Description |
|--------|------------|-------------|
| Intel | GVT-g | Graphics Virtualization Technology for GPUs |
| Intel | GVT-d | Mediated passthrough for discrete GPUs |
| NVIDIA | MIG | Multi-Instance GPU (Ampere and later) |
| AMD | MxGPU | Multi-user GPU virtualization |

### 4. SmartNIC and DPU Acceleration

**Problem**: Network processing overhead on CPUs

**Solution**: Offload networking tasks to SR-IOV-enabled SmartNICs/DPUs

**Benefits:**
- **Hardware-accelerated networking**
- **Protocol offloading** (TCP/IP, TLS, compression)
- **Reduced CPU load**
- **Improved security** (isolated processing)

**SmartNIC Vendors:**
- Mellanox (ConnectX, BlueField)
- Intel (E810, Ice Lake)
- AMD (Pensando)
- NVIDIA (Networking division)
- AWS (Nitro)

## 🔧 SR-IOV Implementation

### Requirements

#### Hardware Requirements
1. **SR-IOV-capable PCIe device** (NIC, HBA, GPU, etc.)
2. **SR-IOV support in BIOS/UEFI**
3. **IOMMU (Input-Output Memory Management Unit)** support
4. **VT-d (Intel) or AMD-Vi/AMD IOMMU** support
5. **Sufficient PCIe lanes** for the device

#### Software Requirements
1. **SR-IOV support in hypervisor**
2. **SR-IOV-capable device drivers**
3. **VF drivers in guest OS**
4. **Management tools**

### Supported Hypervisors

| Hypervisor | SR-IOV Support | Notes |
|------------|----------------|-------|
| **KVM/QEMU** | ✅ Full support | Requires IOMMU, VFIO |
| **VMware ESXi** | ✅ Full support | Enterprise licensing required |
| **Microsoft Hyper-V** | ✅ Full support | Windows Server 2012+ |
| **Xen** | ✅ Full support | Requires IOMMU |
| **Proxmox VE** | ✅ Full support | Based on KVM |
| **Nutanix AHV** | ✅ Full support | - |

### Checking SR-IOV Support

**Check if CPU supports IOMMU:**
```bash
# Intel VT-d
grep -E "vmx|svm" /proc/cpuinfo
cat /proc/cpuinfo | grep vmx  # Intel
cat /proc/cpuinfo | grep svm  # AMD

# IOMMU support
dmesg | grep -E "IOMMU|VT-d|AMD-Vi"
```

**Check if IOMMU is enabled in kernel:**
```bash
# Check kernel command line
cat /proc/cmdline | grep -E "iommu=on|intel_iommu=on|amd_iommu=on"

# Check if IOMMU is enabled
dmesg | grep -i iommu
```

**Check if SR-IOV is enabled in BIOS:**
```bash
# Check via dmidecode
sudo dmidecode -t bios | grep -i sriov

# Or check PCIe device capabilities
lspci -vvv | grep -A 10 -i sriov
```

**Check PCIe device SR-IOV capabilities:**
```bash
# List PCIe devices with SR-IOV
lspci -vvv | grep -B 5 -i "Single Root I/O Virtualization"

# Show detailed info for a specific device
lspci -s 0000:01:00.0 -vvv

# Check number of VFs supported
lspci -s 0000:01:00.0 -vvv | grep -i "Total VFs"
```

### Enabling SR-IOV in Linux (KVM)

**1. Enable IOMMU in Kernel:**

Edit `/etc/default/grub`:
```
GRUB_CMDLINE_LINUX_DEFAULT="... intel_iommu=on iommu=pt"
# For AMD:
GRUB_CMDLINE_LINUX_DEFAULT="... amd_iommu=on iommu=pt"
```

Then update GRUB and reboot:
```bash
sudo update-grub
sudo reboot
```

**2. Enable VFIO:**

Load VFIO kernel modules:
```bash
sudo modprobe vfio
sudo modprobe vfio_pci

# For persistent loading
echo "vfio" | sudo tee -a /etc/modules
echo "vfio_pci" | sudo tee -a /etc/modules
```

**3. Bind NIC to VFIO driver:**

```bash
# Find PCI address of NIC
lspci | grep -i ethernet

# Unbind from current driver
sudo lspci -s 0000:01:00.0 -v | grep Driver
sudo rmmod <current_driver>  # e.g., ixgbe, i40e

# Bind to VFIO
sudo echo "0000:01:00.0" > /sys/bus/pci/drivers/vfio-pci/bind

# Or use driverctl
sudo driverctl set-override 0000:01:00.0 vfio-pci

# Verify
lspci -s 0000:01:00.0 -v | grep Driver
# Should show: Kernel driver in use: vfio-pci
```

**4. Enable SR-IOV on the NIC:**

```bash
# Show current SR-IOV status
cat /sys/class/net/eth0/device/sriov_numvfs

# Enable VFs (example: 8 VFs)
echo 8 | sudo tee /sys/class/net/eth0/device/sriov_numvfs

# Verify
cat /sys/class/net/eth0/device/sriov_numvfs

# Show VFs
lspci | grep -i virtual
```

**5. Configure VFs in QEMU/KVM:**

**Using libvirt:**
```xml
<interface type='hostdev' managed='yes'>
  <mac address='00:11:22:33:44:55'/>
  <source>
    <address type='pci' domain='0x0000' bus='0x01' slot='0x10' function='0x0'/>
  </source>
  <virtualport type='802.1Qbg'>
    <parameters profileid='myprofile'/>
  </virtualport>
</interface>
```

**Using QEMU directly:**
```bash
qemu-system-x86_64 \
  -device vfio-pci,host=0000:01:10.0 \
  -netdev tap,id=net0,ifname=tap0,vhost=on \
  -device virtio-net-pci,netdev=net0,mac=00:11:22:33:44:55
```

**6. Configure VF in Guest VM:**

The guest OS will see the VF as a regular PCIe device and use the appropriate driver.

**For Intel NICs:**
```bash
# In guest, check devices
lspci | grep -i ethernet

# Load appropriate VF driver
echo "ixgbevf" | sudo tee -a /etc/modules
sudo modprobe ixgbevf

# Check interface
ip a
```

**VF Drivers by Vendor:**
| Vendor | NIC Model | VF Driver |
|--------|-----------|-----------|
| Intel | X520, X550, X710, XXV710 | ixgbevf |
| Intel | E810 | ice |
| Mellanox | ConnectX-3/4/5 | mlx5_core, mlx5_ib |
| Broadcom | NetXtreme | bnx2x |
| AMD | - | amd_xgbe |
| NVIDIA | Mellanox (acquired) | mlx5_core |

### Enabling SR-IOV in VMware ESXi

**1. Check SR-IOV Support:**
```
# Check if host supports SR-IOV
esxcli system settings advanced list -o /Net/NetSRIOVEnabled

# Check if SR-IOV is enabled
esxcli system settings advanced list -o /Net/NetSRIOVCapable
```

**2. Enable SR-IOV on Host:**
```
# Enable SR-IOV globally
esxcli system settings advanced set -o /Net/NetSRIOVEnabled -i 1

# Reboot host
reboot
```

**3. Enable SR-IOV on NIC:**
```
# List NICs
esxcli network nic list

# Enable SR-IOV on specific NIC
esxcli network nic sriov set -n vmnic0 -s enabled

# Check SR-IOV status
esxcli network nic sriov get -n vmnic0
```

**4. Configure Number of VFs:**
```
# Set number of VFs (e.g., 8)
esxcli network nic sriov set -n vmnic0 -v 8

# Verify
esxcli network nic sriov get -n vmnic0
```

**5. Assign VF to VM:**
```
# In vSphere Client:
1. Edit VM settings
2. Add PCI device
3. Select the VF
4. Reserve all memory (recommended)
```

Or using PowerCLI:
```powershell
# Get available VFs
Get-VMHostSriovDevice

# Add VF to VM
New-VMHostDevice -VMHost <host> -Device <VF_device>
New-HardDisk -VM <vm> -DiskPath <path> -Controller <controller>
```

### Enabling SR-IOV in Hyper-V

**1. Check SR-IOV Support:**
```powershell
# Check if host supports SR-IOV
Get-VMHost | Select-Object Name, SriovSupport

# Check if SR-IOV is enabled
Get-VMHost | Select-Object Name, SriovEnabled
```

**2. Enable SR-IOV on Host:**
```powershell
# Enable SR-IOV
Enable-VMHostSriov -VMHost (Get-VMHost)

# Verify
Get-VMHost | Select-Object Name, SriovEnabled
```

**3. Enable SR-IOV on NIC:**
```powershell
# List NICs
Get-NetAdapter | Where-Object {$_.SriovSupport -eq $true}

# Enable SR-IOV on NIC
Enable-NetAdapterSriov -Name "Ethernet"

# Set number of VFs
Set-NetAdapterSriov -Name "Ethernet" -NumVfs 8

# Verify
Get-NetAdapterSriov -Name "Ethernet"
```

**4. Assign VF to VM:**
```powershell
# Get available VFs
Get-VMHostSriovDevice

# Add VF to VM
Add-VMSriovNetworkAdapter -VMName "VM1" -NetworkName "External" -SriovDevice <VF>

# Or add PCI passthrough
Add-VMPciDevice -VMName "VM1" -Path <VF_PCI_Path>
```

## 📊 SR-IOV Configuration Examples

### Example 1: Intel X710 NIC with 16 VFs

```bash
# Check NIC
lspci | grep -i x710
# Output: 01:00.0 Ethernet controller: Intel Corporation Ethernet Controller X710 for 10GbE SFP+ (rev 02)

# Check SR-IOV support
lspci -s 01:00.0 -vvv | grep -i sriov
# Output: Capabilities: [140 v1] Single Root I/O Virtualization (SR-IOV)
#         Total VFs: 16

# Enable 8 VFs
echo 8 | sudo tee /sys/class/net/eth0/device/sriov_numvfs

# Verify
cat /sys/class/net/eth0/device/sriov_numvfs
# Output: 8

# List VFs
lspci | grep -i virtual
# Output:
# 01:10.0 Ethernet controller: Intel Corporation Ethernet Virtual Function 710 Series (rev 02)
# 01:10.1 Ethernet controller: Intel Corporation Ethernet Virtual Function 710 Series (rev 02)
# ...
# 01:10.7 Ethernet controller: Intel Corporation Ethernet Virtual Function 710 Series (rev 02)
```

### Example 2: Mellanox ConnectX-3 with 128 VFs

```bash
# Check NIC
lspci | grep -i mellanox
# Output: 01:00.0 Ethernet controller: Mellanox Technologies MT27700 Family [ConnectX-3]

# Check SR-IOV support
lspci -s 01:00.0 -vvv | grep -i sriov
# Output: Capabilities: [150 v1] Single Root I/O Virtualization (SR-IOV)
#         Total VFs: 128

# Enable 32 VFs
echo 32 | sudo tee /sys/class/net/eth0/device/sriov_numvfs

# Load VF driver in host
sudo modprobe mlx5_core
sudo modprobe mlx5_ib

# Verify VFs
lspci | grep -i virtual | head -5
# Output:
# 01:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-3 Virtual Function]
# 01:00.2 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-3 Virtual Function]
# ...
```

### Example 3: KVM with libvirt and SR-IOV

**Host Configuration:**
```bash
# Enable IOMMU
echo "GRUB_CMDLINE_LINUX_DEFAULT="\"\$GRUB_CMDLINE_LINUX_DEFAULT intel_iommu=on iommu=pt\"" | sudo tee -a /etc/default/grub
sudo update-grub
sudo reboot

# After reboot, enable VFs
echo 4 | sudo tee /sys/class/net/ens1f0/device/sriov_numvfs

# Create VFs as PCI devices
for i in {0..3}; do
  echo "0000:01:00.$((i+1))" | sudo tee /sys/bus/pci/drivers/vfio-pci/bind
done
```

**VM Configuration (libvirt XML):**
```xml
<domain type='kvm'>
  <name>sriov-vm</name>
  <memory>8388608</memory>
  <vcpu>4</vcpu>
  <os>
    <type>hvm</type>
    <boot dev='network'/>
  </os>
  <devices>
    <interface type='hostdev'>
      <mac address='00:11:22:33:44:55'/>
      <source>
        <address type='pci' domain='0x0000' bus='0x01' slot='0x01' function='0x0'/>
      </source>
    </interface>
    <hostdev mode='subsystem' type='pci' managed='yes'>
      <source>
        <address domain='0x0000' bus='0x01' slot='0x01' function='0x0'/>
      </source>
    </hostdev>
  </devices>
</domain>
```

**Start VM:**
```bash
virsh define /path/to/sriov-vm.xml
virsh start sriov-vm

# In guest
lspci | grep -i ethernet
# Should show the VF device
```

## 🔐 SR-IOV Security Considerations

### Security Features

| Feature | Description | Implementation |
|---------|-------------|----------------|
| **IOMMU** | Isolates device DMA access | Intel VT-d, AMD-Vi |
| **ACS (Access Control Services)** | Prevents peer-to-peer DMA | PCIe ACS |
| **VF Isolation** | Prevents VF-to-VF communication | Device-specific |
| **Admin Functions** | Restrict PF access | Software controls |
| **Virtualization Extensions** | Hardware-enforced isolation | CPU support |

### Security Best Practices

1. **Enable IOMMU**: Always enable IOMMU (VT-d/AMD-Vi) in BIOS and kernel
2. **Use VFIO for Passthrough**: Bind devices to VFIO driver instead of direct assignment
3. **Limit VF Access**: Only assign VFs to trusted VMs
4. **Disable Peer-to-Peer DMA**: Use ACS to prevent VF-to-VF communication
5. **Monitor Device Activity**: Use IOMMU groups to monitor DMA activity
6. **Use Secure Boot**: Ensure hypervisor and guest OS use secure boot
7. **Network Isolation**: Use VLANs or other mechanisms to isolate VF traffic
8. **Rate Limiting**: Configure QoS to prevent VF from consuming all bandwidth

**Check IOMMU Groups:**
```bash
# List IOMMU groups
ls /sys/kernel/iommu_groups/

# Show devices in each group
for i in /sys/kernel/iommu_groups/*; do
  echo "IOMMU Group $(basename $i):"
  cat $i/devices
  echo ""
done
```

**IOMMU Group Example:**
```
IOMMU Group 0:
0000:00:00.0
IOMMU Group 1:
0000:00:01.0
0000:00:01.1
IOMMU Group 2:
0000:00:14.0
IOMMU Group 3:
0000:00:16.0
IOMMU Group 4:
0000:00:1c.0
IOMMU Group 5:
0000:00:1d.0
IOMMU Group 6:
0000:01:00.0
0000:01:10.0
0000:01:10.1
0000:01:10.2
0000:01:10.3
```

**Note**: All VFs from a PF will be in the same IOMMU group. This means they must be assigned to the same VM or use ACS for isolation.

### ACS (Access Control Services)

**Purpose**: Prevent DMA transactions between devices that shouldn't communicate

**ACS Capabilities:**
- **Source Validation**: Validate requester ID
- **Translation Blocking**: Block DMA through certain paths
- **Peer-to-Peer Request Blocking**: Block P2P transactions
- **Peer-to-Peer Completion Blocking**: Block completions from P2P

**Check ACS Support:**
```bash
# Check if ACS is supported
lspci -vvv | grep -i acs

# Check ACS capabilities for a device
lspci -s 0000:01:00.0 -vvv | grep -A 5 -i acs
```

**Enable ACS in Kernel:**
```bash
# Enable ACS for PCIe devices
echo "pci=acs_override_downstream" | sudo tee -a /etc/default/grub
sudo update-grub
sudo reboot
```

## 📊 Performance Optimization

### Tuning SR-IOV Performance

#### Host-Level Optimizations

1. **Enable Huge Pages**:
   ```bash
   # Check huge page support
   grep Huge /proc/meminfo
   
   # Allocate huge pages
   echo 1024 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
   
   # Make persistent
   echo "vm.nr_hugepages = 1024" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

2. **CPU Pinning**:
   ```bash
   # Pin VM vCPUs to specific physical CPUs
   virsh vcpupin <vm> <vcpu> <pcpu>
   
   # Example: Pin VM1 vCPU 0 to physical CPU 2
   virsh vcpupin vm1 0 2
   ```

3. **NUMA Optimization**:
   ```bash
   # Check NUMA topology
   numactl --hardware
   
   # Run VM on specific NUMA node
   numactl --cpunodebind=0 --membind=0 qemu-system-x86_64 ...
   ```

4. **Interrupt Affinity**:
   ```bash
   # Set IRQ affinity
   echo 4 | sudo tee /proc/irq/<irq_number>/smp_affinity
   
   # Or use taskset
   taskset -cp <cpu_list> <process>
   ```

#### Guest-Level Optimizations

1. **Enable MSI-X Interrupts**:
   ```bash
   # Check if MSI-X is enabled
   lspci -v | grep -i msix
   
   # In guest kernel, ensure MSI-X is enabled
   grep MSI /proc/interrupts
   ```

2. **Tune Network Stack**:
   ```bash
   # Increase TCP buffer sizes
   echo "net.core.rmem_max=16777216" | sudo tee -a /etc/sysctl.conf
   echo "net.core.wmem_max=16777216" | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   
   # Tune TCP
   echo "net.ipv4.tcp_rmem='4096 87380 16777216'" | sudo tee -a /etc/sysctl.conf
   echo "net.ipv4.tcp_wmem='4096 65536 16777216'" | sudo tee -a /etc/sysctl.conf
   ```

3. **Disable Offloads (if needed)**:
   ```bash
   # Some offloads may interfere with SR-IOV
   ethtool -K eth0 tx off rx off sg off tso off gso off gro off lro off
   ```

4. **Use DPDK in Guest**:
   ```bash
   # Bind VF to DPDK in guest
   sudo dpdk-devbind.py --bind=vfio-pci 0000:00:02.0
   
   # Run DPDK application
   sudo ./dpdk-app --lcore 0-3 --master lcore 0 --pci-whitelist 0000:00:02.0
   ```

### Performance Comparison

| Configuration | Latency | Throughput | CPU Usage | Notes |
|---------------|---------|------------|-----------|-------|
| **Legacy (Emulated NIC)** | High (~50-100μs) | Low (~1-2 Gbps) | Very High | Full emulation |
| **VirtIO (Software)** | Medium (~20-50μs) | Medium (~5-10 Gbps) | High | Paravirtualized |
| **SR-IOV (VF)** | Low (~5-10μs) | High (~10-40 Gbps) | Low | Hardware-assisted |
| **PCI Passthrough** | Lowest (~2-5μs) | Line Rate | None | Full device passthrough |
| **DPDK + SR-IOV** | Near Zero (~1-2μs) | Line Rate | Very Low | Kernel bypass + SR-IOV |

**Benchmark Example (10Gbps NIC):**
```
Configuration:        Latency (μs)  Throughput  CPU Usage
───────────────────────────────────────────────────────────
Emulated NIC:         80-120        1.2 Gbps    80%
VirtIO:               25-45         8.5 Gbps    30%
SR-IOV (VF):          6-12          9.8 Gbps    5%
PCI Passthrough:      2-5           9.9 Gbps    0%
DPDK + SR-IOV:        1-3           14.8 Mpps   2%
```

## 🛠️ Troubleshooting SR-IOV

### Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **SR-IOV not supported** | Device doesn't show SR-IOV capability | Check BIOS settings, device support |
| **IOMMU not enabled** | VFIO binding fails | Enable IOMMU in BIOS and kernel |
| **Driver doesn't support VF** | VF not detected in guest | Use correct VF driver |
| **VF not showing in lspci** | VFs not created | Enable VFs via sriov_numvfs |
| **Performance poor** | High latency, low throughput | Check NUMA, CPU pinning, interrupt affinity |
| **VM can't start** | Error when starting VM | Check IOMMU groups, ACS settings |
| **Network connectivity issues** | Packets dropped | Check VLAN configuration, virtual switch settings |
| **ACS violation** | VF-to-VF communication blocked | Enable ACS or use ACS override |

### Troubleshooting Commands

**Host-Level:**
```bash
# Check IOMMU status
dmesg | grep -i iommu

# Check IOMMU groups
ls -l /sys/kernel/iommu_groups/

# Check PCIe devices
lspci -vvv

# Check SR-IOV status
lspci -s <BDF> -vvv | grep -i sriov

# Check VFs created
lspci | grep -i virtual

# Check VFIO devices
ls /sys/bus/pci/drivers/vfio-pci/

# Check kernel messages
dmesg | tail -20

# Check module loading
lsmod | grep -E "vfio|iommu"
```

**Guest-Level:**
```bash
# Check PCI devices
lspci -vvv

# Check network interfaces
ip a

# Check driver
ethtool -i eth0

# Check kernel messages
dmesg | grep -i sriov
dmesg | grep -i vf

# Check module loading
lsmod | grep -E "ixgbevf|i40evf|mlx5"
```

**VMware ESXi:**
```
# Check SR-IOV status
esxcli network nic sriov get -n vmnic0

# Check VM configuration
vim-cmd vmsvc/get.summary <vmid>

# Check logs
esxcli system coredump partition list
cat /var/log/vmkernel.log | grep -i sriov
```

**Hyper-V:**
```powershell
# Check SR-IOV status
Get-NetAdapterSriov -Name "Ethernet"

# Check VM configuration
Get-VMNetworkAdapter -VMName "VM1" | Select-Object *

# Check event logs
Get-WinEvent -LogName "Hyper-V-Worker" | Where-Object {$_.Message -like "*SR-IOV*"}
```

### Common Error Messages

**Error: "Failed to assign device: IOMMU group not found"**
- **Cause**: IOMMU not enabled or device not in IOMMU group
- **Solution**: Enable IOMMU in BIOS and kernel

**Error: "VFIO: No IOMMU group for device"**
- **Cause**: Device not isolated in IOMMU
- **Solution**: Check IOMMU groups, use ACS override

**Error: "SR-IOV: VF not created"**
- **Cause**: sriov_numvfs not set or driver doesn't support
- **Solution**: Set sriov_numvfs, check driver

**Error: "No space for VF BAR"**
- **Cause**: Not enough PCI address space
- **Solution**: Enable 64-bit BAR support in BIOS

**Error: "Driver does not support SR-IOV"**
- **Cause**: Using wrong driver in guest
- **Solution**: Use correct VF driver (ixgbevf, i40evf, etc.)

## 🎯 Key Takeaways

1. **SR-IOV enables near-native performance** by allowing a physical device to present multiple virtual functions
2. **PF (Physical Function) vs VF (Virtual Function)**: PF manages the device, VFs are lightweight instances for VMs
3. **Hardware Requirements**: SR-IOV-capable device, IOMMU support (VT-d/AMD-Vi), sufficient PCIe lanes
4. **Software Requirements**: SR-IOV support in hypervisor, VF drivers in guest
5. **Performance**: SR-IOV reduces latency from ~50μs to ~5-10μs and improves throughput from ~1-2 Gbps to ~10+ Gbps
6. **Security**: IOMMU and ACS provide isolation between VFs
7. **Optimizations**: Huge pages, CPU pinning, NUMA optimization, interrupt affinity
8. **Use Cases**: Network virtualization, storage virtualization, GPU virtualization, SmartNIC acceleration
9. **Troubleshooting**: Check IOMMU groups, VF creation, driver support
10. **Alternatives**: PCI passthrough (better performance but less flexible), DPDK (kernel bypass + SR-IOV)

## 🔗 Further Reading

- [PCI-SIG SR-IOV Specification](https://pcisig.com/specifications/conventional/sr-iov/)
- [Intel SR-IOV Documentation](https://www.intel.com/content/www/us/en/developer/articles/technical/introduction-to-sr-iov-technology.html)
- [Intel VT-d Documentation](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-virtualization-technology-for-directed-io.html)
- [AMD IOMMU Documentation](https://developer.amd.com/resources/amd-iommu-architecture-guidelines/)
- [Linux VFIO Documentation](https://www.kernel.org/doc/html/latest/driver-api/vfio.html)
- [QEMU SR-IOV Guide](https://wiki.qemu.org/Features/SRIOV)
- [KVM SR-IOV Guide](https://www.linux-kvm.org/page/SR-IOV)
- [VMware SR-IOV Documentation](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.networking.doc/GUID-132485D8-0103-4301-93A5-3A001A4E7E27.html)
- [Microsoft SR-IOV Documentation](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/sriov-overview)
- [DPDK SR-IOV Guide](https://doc.dpdk.org/guides/nics/sriov_drivers.html)
- [SR-IOV in Cloud Computing](https://www.ibm.com/cloud/learn/sr-iov)
- [SR-IOV vs PCI Passthrough](https://www.qemu.org/docs/master/system/invocation.html#hxtool-2)

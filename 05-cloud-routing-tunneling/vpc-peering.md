# VPC Peering

> Connecting Virtual Private Clouds across regions, accounts, and providers

---

## 🎯 Purpose

**VPC Peering** enables direct, private connectivity between two Virtual Private Clouds (VPCs) using their private IP addresses, as if they were part of the same network. This allows resources in different VPCs to communicate without traversing the public internet.

**Key Benefits:**
- ✅ **Private connectivity** - Traffic stays within cloud provider's network
- ✅ **Low latency** - Uses provider's internal backbone
- ✅ **High bandwidth** - Scales with instance size
- ✅ **Cost effective** - Typically cheaper than VPN/Direct Connect
- ✅ **Simple setup** - No gateways or VPN devices required
- ✅ **Cross-account & cross-region** support

## 🏗️ How VPC Peering Works

### Basic Concept

VPC peering creates a **non-transitive** connection between two VPCs. Each VPC maintains its own security groups, network ACLs, and routing tables.

```
┌─────────────────┐       ┌─────────────────┐
│     VPC A        │       │     VPC B        │
│   10.0.0.0/16    │       │   172.16.0.0/16  │
│                 │       │                 │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │ EC2       │  │       │  │ EC2       │  │
│  │ 10.0.0.1 │◄─┼───────►│  │ 172.16.0.1│  │
│  └───────────┘  │       │  └───────────┘  │
│                 │       │                 │
└────────┬────────┘       └────────┬────────┘
         │                             │
         └───────────── Peering Connection ─────┘
           (Private, Direct, Encrypted)
```

### Important Characteristics

| Feature | VPC Peering |
|---------|-------------|
| **Transitive** | ❌ No - A←→B and B←→C doesn't mean A←→C |
| **Overlapping CIDRs** | ❌ Not allowed |
| **Encryption** | ✅ Yes (provider-managed) |
| **Bandwidth** | Up to instance limits |
| **Latency** | Same as intra-region |
| **Security Groups** | ✅ Applied independently |
| **Network ACLs** | ✅ Applied independently |

### Non-Transitive Example

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  VPC A  │─────│  VPC B  │─────│  VPC C  │
│ 10.0.0.0│     │172.16.0.0│     │192.168.0.0│
└─────────┘     └─────────┘     └─────────┘
     ✗               ✓               ✓
     │               │               │
     └───── CANNOT communicate ──────┘

Even though B peers with both A and C, A cannot communicate with C directly.
Solution: Use Transit Gateway for transitive routing.
```

## 🌐 Cloud Provider Implementations

### AWS VPC Peering

#### Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Same Region** | VPCs in same AWS region | Multi-environment isolation |
| **Cross Account** | VPCs in different AWS accounts | Organization-wide networking |
| **Inter-Region** | VPCs in different AWS regions | Disaster recovery, global apps |
| **IPv6** | Peering for IPv6 VPCs | IPv6-only environments |

#### Step-by-Step Setup

**1. Create Peering Connection (Requester):**
```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-1234abcd \
  --peer-vpc-id vpc-5678efgh \
  --peer-owner-id 987654321098 \
  --peer-region us-east-1
```

**2. Accept Peering Connection (Accepter):**
```bash
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-1234567890abcdef0
```

**3. Update Route Tables (Both VPCs):**
```bash
# VPC A: Route to VPC B
aws ec2 create-route \
  --route-table-id rtb-1234abcd \
  --destination-cidr-block 172.16.0.0/16 \
  --vpc-peering-connection-id pcx-1234567890abcdef0

# VPC B: Route to VPC A
aws ec2 create-route \
  --route-table-id rtb-5678efgh \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id pcx-1234567890abcdef0
```

**4. Update Security Groups:**
```bash
# VPC A: Allow traffic from VPC B
aws ec2 authorize-security-group-ingress \
  --group-id sg-1234abcd \
  --protocol tcp \
  --port 22 \
  --cidr 172.16.0.0/16

# VPC B: Allow traffic from VPC A
aws ec2 authorize-security-group-ingress \
  --group-id sg-5678efgh \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/16
```

#### AWS Limitations

| Limitation | Workaround |
|------------|------------|
| Max 125 peering connections per VPC | Use Transit Gateway |
| No transitive peering | Use Transit Gateway |
| No overlapping CIDRs | Use NAT or different CIDRs |
| Inter-region latency | Use same region when possible |

#### Example Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Region: us-east-1                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐       ┌─────────────────┐                │
│  │   Dev VPC        │       │   Prod VPC       │                │
│  │   10.0.0.0/16    │       │   10.1.0.0/16    │                │
│  │                 │       │                 │                │
│  │  ┌───────────┐  │       │  ┌───────────┐  │                │
│  │  │  Dev      │  │       │  │  Prod     │  │                │
│  │  │  Resources│◄─┼───────►│  │  Resources│  │                │
│  │  └───────────┘  │       │  └───────────┘  │                │
│  └────────┬────────┘       └────────┬────────┘                │
│           │                             │                │
│           └────────── Peering Connection ──────────┘        │
│                                                  │                │
│                    pcx-1234567890abcdef0                 │
│                                                  │                │
└─────────────────────────────────────────────────────────────┘
```

#### AWS CLI Reference

```bash
# List all peering connections
aws ec2 describe-vpc-peering-connections

# Describe specific connection
aws ec2 describe-vpc-peering-connections --vpc-peering-connection-ids pcx-1234abcd

# Delete peering connection
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-1234abcd

# Reject peering request
aws ec2 reject-vpc-peering-connection --vpc-peering-connection-id pcx-1234abcd
```

### Azure VNet Peering

#### Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Global Peering** | Cross-region peering | DR, global applications |
| **Remote Peering** | Same-region peering | Multi-tier apps |
| **Same Subscription** | VNets in same subscription | Simple scenarios |
| **Cross Subscription** | VNets in different subscriptions | Enterprise |

#### Step-by-Step Setup

**1. Create Peering (VNet1 → VNet2):**
```bash
az network vnet peering create \
  --name VNet1-To-VNet2 \
  --resource-group myResourceGroup \
  --vnet-name VNet1 \
  --remote-vnet VNet2 \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit
```

**2. Create Reverse Peering (VNet2 → VNet1):**
```bash
az network vnet peering create \
  --name VNet2-To-VNet1 \
  --resource-group myResourceGroup \
  --vnet-name VNet2 \
  --remote-vnet VNet1 \
  --allow-vnet-access \
  --allow-forwarded-traffic
```

#### Azure Peering Options

| Option | Description |
|--------|-------------|
| `--allow-vnet-access` | Allow resources in each VNet to communicate |
| `--allow-forwarded-traffic` | Allow forwarded traffic from VMs in peer VNet |
| `--allow-gateway-transit` | Allow traffic from peer VNet to use this VNet's gateway |
| `--use-remote-gateways` | Use peer VNet's gateway for traffic |

#### Example Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure Region                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐       ┌─────────────────┐                │
│  │   Hub VNet       │       │   Spoke VNet     │                │
│  │   10.0.0.0/16    │       │   172.16.0.0/16  │                │
│  │                 │       │                 │                │
│  │  ┌───────────┐  │       │  ┌───────────┐  │                │
│  │  │  Gateway  │  │       │  │  VMs      │  │                │
│  │  │  Subnet   │  │       │  └───────────┘  │                │
│  │  └───────────┘  │       └─────────────────┘                │
│  └────────┬────────┘                                     │
│           │                                                 │
│           └────────── Peering Connection ──────────┘        │
│                                                  │                │
│  --allow-gateway-transit allows Spoke VNet to use Hub VNet's │
│  gateway for internet access                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Azure CLI Reference

```bash
# List all peering connections
az network vnet peering list --resource-group myResourceGroup --vnet-name VNet1

# Show peering details
az network vnet peering show --name VNet1-To-VNet2 --resource-group myResourceGroup --vnet-name VNet1

# Delete peering
az network vnet peering delete --name VNet1-To-VNet2 --resource-group myResourceGroup --vnet-name VNet1

# Check peering status
az network vnet peering list -g myResourceGroup --vnet-name VNet1 -o table
```

### Google Cloud VPC Network Peering

#### Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Internal** | Same organization, any region | Multi-project apps |
| **Cross Organization** | Different organizations | Partner connectivity |
| **Private Service Connect** | Access Google services | Private Google APIs |

#### Step-by-Step Setup

**1. Create Peering (Project A → Project B):**
```bash
gcloud compute networks peerings create peer-to-b \
  --network vpc-a \
  --peer-network https://www.googleapis.com/compute/v1/projects/project-b/global/networks/vpc-b \
  --auto-create-routes
```

**2. Accept Peering (Project B):**
```bash
gcloud compute networks peerings create peer-to-a \
  --network vpc-b \
  --peer-network https://www.googleapis.com/compute/v1/projects/project-a/global/networks/vpc-a \
  --auto-create-routes
```

#### GCP Peering Features

| Feature | Description |
|---------|-------------|
| **Auto-create routes** | Automatically create routes in both VPCs |
| **Custom routes** | Manually create specific routes |
| **Export/Import** | Control which custom routes are shared |
| **Private Service Connect** | Access Google services via private IP |

#### Example Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Google Cloud Global                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐       ┌─────────────────┐                │
│  │  Project A       │       │  Project B       │                │
│  │  VPC: vpc-a      │       │  VPC: vpc-b      │                │
│  │  10.0.0.0/16    │       │  192.168.0.0/16 │                │
│  │                 │       │                 │                │
│  │  ┌───────────┐  │       │  ┌───────────┐  │                │
│  │  │  GKE      │  │       │  │  GCE      │  │                │
│  │  │  Cluster  │◄─┼───────►│  │  Instances│  │                │
│  │  └───────────┘  │       │  └───────────┘  │                │
│  └────────┬────────┘       └────────┬────────┘                │
│           │                             │                │
│           └────────── VPC Peering ────────────┘         │
│                                                        │
│  --auto-create-routes automatically adds routes in both│
│  VPCs for the peer network CIDR blocks                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### GCP CLI Reference

```bash
# List all peering connections
gcloud compute networks peerings list

# Describe peering
gcloud compute networks peerings describe peer-to-b --network vpc-a

# Delete peering
gcloud compute networks peerings delete peer-to-b --network vpc-a

# Check routes
gcloud compute routes list --filter="network:vpc-a"
```

## 🔧 Advanced VPC Peering Configurations

### Hub-and-Spoke Architecture

Use a central VPC (hub) for shared services, with spoke VPCs for individual applications/environments.

```
┌─────────────────────────────────────────────────────────────┐
│                    Hub-and-Spoke Architecture                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐                                         │
│  │    Hub VPC       │                                         │
│  │   10.0.0.0/16    │                                         │
│  │                 │                                         │
│  │  ┌───────────┐  │    ┌─────────────┐    ┌─────────────┐  │
│  │  │  Shared   │  │    │             │    │             │  │
│  │  │  Services │◄──┼───► Spoke VPC 1 │    │ Spoke VPC 2 │  │
│  │  │  (NAT GW, │  │    │ 172.16.0.0/ │    │ 192.168.0.0/│  │
│  │  │   VPN,    │  │    │    16)      │    │    16)      │  │
│  │  │   DNS,    │  │    └─────────────┘    └─────────────┘  │
│  │  └───────────┘  │                                         │
│  └─────────────────┘                                         │
│                                                             │
│  Limitation: Spoke VPCs CANNOT communicate with each other  │
│  Solution: Use Transit Gateway (AWS) or similar             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Multi-Account Strategy (AWS)

**Organization Structure:**
```
AWS Organization
├── Master Account (Billing, SCPs)
├── Network Account (Transit Gateway, VPC Peering Hub)
│   └── Hub VPC
│       ├── Transit Gateway
│       └── Shared Services (DNS, NTP, Logging)
│
├── Dev Account
│   └── Dev VPC (10.0.0.0/16)
│
├── Prod Account
│   └── Prod VPC (10.1.0.0/16)
│
└── Security Account
    └── Security VPC (10.2.0.0/16)
```

**Peering Setup:**
1. Create peering connections from Hub VPC to each spoke VPC
2. Use Transit Gateway for transitive routing
3. Apply Security Groups and NACLs for isolation

### Cross-Region Disaster Recovery

```
┌─────────────────────┐       ┌─────────────────────┐
│   Primary Region    │       │   DR Region         │
│   (us-east-1)        │       │   (us-west-2)        │
├─────────────────────┤       ├─────────────────────┤
│                     │       │                     │
│  ┌───────────────┐  │       │  ┌───────────────┐  │
│  │ Primary VPC   │  │       │  │ DR VPC        │  │
│  │ 10.0.0.0/16    │◄───────►│  │ 10.0.0.0/16    │  │
│  │               │  │       │  │ (replicated)  │  │
│  │  - Web tier  │  │       │  │  - Web tier   │  │
│  │  - App tier  │  │       │  │  - App tier   │  │
│  │  - DB tier   │  │       │  │  - DB tier    │  │
│  └───────────────┘  │       │  └───────────────┘  │
│                     │       │                     │
│  Inter-Region VPC Peering                             │
│  - Low latency replication                            │
│  - Private connectivity                               │
│  - Automatic failover possible                         │
└─────────────────────┘       └─────────────────────┘
```

**Setup:**
```bash
# Primary region (us-east-1)
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-primary \
  --peer-vpc-id vpc-dr \
  --peer-region us-west-2

# DR region (us-west-2) - accept
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-dr \
  --region us-west-2
```

### Private Services Access

**AWS PrivateLink with VPC Peering:**

```
┌─────────────────────────────────────────────────────────────┐
│                    PrivateLink with VPC Peering                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐                                         │
│  │   Service VPC    │                                         │
│  │   10.0.0.0/16    │                                         │
│  │                 │                                         │
│  │  ┌───────────┐  │                                         │
│  │  │ NLB       │  │                                         │
│  │  │ (Internal)│  │                                         │
│  │  └───────┬─────┘  │                                         │
│  │          │        │                                         │
│  │  ┌───────▼─────┐  │                                         │
│  │  │  Endpoint   │  │                                         │
│  │  │  Service    │  │                                         │
│  │  └───────┬─────┘  │                                         │
│  └──────────┼────────┘                                         │
│             │                                                   │
│             ▼                                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    VPC Peering Connection               │  │
│  └──────────────────────────┬─────────────────────────────┘  │
│                             │                                  │
│  ┌──────────────────────────▼─────────────────────────────┐  │
│  │                 Consumer VPCs                           │  │
│  │  ┌──────────────┐      ┌──────────────┐                │  │
│  │  │ Consumer A   │      │ Consumer B   │                │  │
│  │  │ 172.16.0.0/16│      │ 192.168.0.0/16│                │  │
│  │  └──────────────┘      └──────────────┘                │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
│  Consumers can access the service via PrivateLink endpoint   │
│  Traffic stays within AWS network                            │
└─────────────────────────────────────────────────────────────┘
```

**Benefits:**
- Private access to services (no public internet)
- No NAT required
- Services can be in different accounts
- Traffic stays within AWS backbone

## 🛡️ Security Considerations

### Security Best Practices

1. **Minimize Peering Scope**
   - Only peer VPCs that need to communicate
   - Avoid peering entire organizations by default

2. **Use Network ACLs**
   - Control traffic flow at subnet level
   - Apply least-privilege access

3. **Security Groups**
   - Restrict traffic between peered VPCs
   - Use specific IP ranges, not 0.0.0.0/0

4. **VPC Flow Logs**
   - Enable flow logs for all peered VPCs
   - Monitor traffic patterns

5. **Route Table Controls**
   - Use specific route targets
   - Avoid catch-all routes (0.0.0.0/0)

6. **Encryption**
   - VPC peering is encrypted by default
   - For additional security, use TLS for application traffic

### Security Group Example

**VPC A Security Group (allow from VPC B):**
```json
{
  "IpPermissions": [
    {
      "IpProtocol": "tcp",
      "FromPort": 443,
      "ToPort": 443,
      "IpRanges": [
        {
          "CidrIp": "172.16.0.0/16",
          "Description": "Allow HTTPS from VPC B"
        }
      ]
    }
  ]
}
```

### Network ACL Example

**VPC A Network ACL (allow return traffic):**

| Rule # | Type | Protocol | Port Range | Source | Destination | Action | Allow/Deny |
|--------|------|----------|------------|--------|-------------|--------|------------|
| 100 | Custom | TCP | 1024-65535 | 172.16.0.0/16 | 0.0.0.0/0 | ALLOW | ✅ |
| * | All | All | All | 0.0.0.0/0 | 0.0.0.0/0 | DENY | ❌ |

## 📊 Monitoring and Troubleshooting

### Monitoring Tools

**AWS:**
```bash
# Check peering connection status
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids pcx-1234abcd

# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-1234abcd"

# Check network ACLs
aws ec2 describe-network-acls --filters "Name=vpc-id,Values=vpc-1234abcd"

# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type Subnet \
  --resource-ids subnet-1234abcd \
  --traffic-type ALL \
  --log-group-name VPCFlowLogs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/FlowLogsRole
```

**Azure:**
```bash
# Check peering status
az network vnet peering show -g myRG -n VNet1-To-VNet2 --vnet-name VNet1

# Check effective routes
az network vnet list-effective-route-tables -g myRG --vnet-name VNet1

# Check NSG flow logs
az monitor log-analytics query \
  --workspace myWorkspace \
  --analytics-query "AzureNetworkAnalytics_CL | where TimeGenerated > ago(1h)"
```

**GCP:**
```bash
# Check peering status
gcloud compute networks peerings describe peer-to-b --network vpc-a

# Check firewall rules
gcloud compute firewall-rules list --filter="network:vpc-a"

# Check routes
gcloud compute routes list --filter="network:vpc-a"
```

### Common Issues and Solutions

| Issue | Symptom | Cause | Solution |
|-------|---------|-------|----------|
| **No connectivity** | Can't ping between VPCs | Missing routes | Add routes to both VPC route tables |
| **Security group blocking** | Connection timeout | SG rules too restrictive | Update security groups to allow traffic |
| **NACL blocking** | No traffic between subnets | NACL deny rules | Update NACL to allow traffic |
| **Overlapping CIDRs** | Error creating peering | CIDR blocks overlap | Use different CIDR blocks or NAT |
| **Transitive peering** | A can't reach C | Non-transitive nature | Use Transit Gateway |
| **Peering in wrong state** | Status: pending | Not accepted | Accept peering on accepter side |
| **Route propagation** | Routes not appearing | Auto-create disabled | Enable auto-create routes or add manually |

### Troubleshooting Commands

```bash
# AWS - Test connectivity
aws ec2 describe-instances --instance-ids i-1234abcd --query "Reservations[0].Instances[0].PrivateIpAddress"

# Ping test (requires ICMP allowed in SG)
# On an EC2 instance in VPC A:
ping 172.16.0.100  # Instance in VPC B

# Traceroute
traceroute 172.16.0.100

# Check security groups
aws ec2 describe-security-groups --group-ids sg-1234abcd

# Azure - Test connectivity
az vm list-ip-addresses -g myRG -n myVM --query "[0].virtualMachine.network.privateIps[0].ipAddress" -o tsv

# GCP - Test connectivity
gcloud compute instances describe my-vm --zone us-central1-a --format="value(networkInterfaces[0].networkIP)"
```

## 📈 Performance and Cost

### Performance Considerations

| Factor | Impact | Recommendation |
|--------|--------|----------------|
| **Same Region** | Best performance | Use same region when possible |
| **Cross Region** | Higher latency | Use for DR, not production traffic |
| **Instance Size** | Higher bandwidth | Use larger instances for high traffic |
| **Network Topology** | Affects latency | Place peered VPCs in same AZ when possible |
| **Encryption** | Minimal overhead | Built-in, no configuration needed |

### Cost Analysis

**AWS Pricing (us-east-1 as of 2024):**

| Item | Cost |
|------|------|
| Peering connection | $0.01 per hour per connection |
| Data transfer (same region) | $0.01 per GB |
| Data transfer (cross region) | $0.02 per GB |

**Azure Pricing:**
- VNet peering: Free within same region
- Cross-region peering: ~$0.01/GB data transfer
- Global VNet peering: ~$0.05/GB data transfer

**GCP Pricing:**
- VPC Network Peering: Free
- Egress traffic: $0.01/GB (same region), $0.05-0.12/GB (cross region)

**Cost Optimization Tips:**
1. Use same region peering when possible
2. Minimize cross-region data transfer
3. Use compression for high-volume traffic
4. Monitor data transfer with CloudWatch/Cloud Monitoring
5. Consider Transit Gateway for many VPCs (cost vs. peering limits)

### Throughput Limits

| Instance Type | Max Bandwidth (Gbps) | Max Packets per Second |
|---------------|------------------------|-------------------------|
| t3.micro | 0.5 | 50K |
| m5.large | 10 | 500K |
| c5.xlarge | 10 | 500K |
| m5.2xlarge | 25 | 1.4M |
| c5.4xlarge | 25 | 1.4M |
| m5.8xlarge | 50 | 2.8M |
| c5.9xlarge | 50 | 2.8M |

**Note:** These are theoretical maximums. Actual throughput depends on many factors.

## 🎯 Best Practices

### Design Best Practices

1. **Use Non-Overlapping CIDR Blocks**
   - Plan your IP address space carefully
   - Use RFC 1918 private address ranges
   - Consider future growth

2. **Hub-and-Spoke with Transit Gateway**
   - For >3 VPCs, use Transit Gateway instead of full mesh peering
   - Reduces management overhead
   - Enables transitive routing

3. **Centralize Shared Services**
   - Place shared services (DNS, NTP, logging) in a hub VPC
   - Peer spoke VPCs to hub VPC

4. **Cross-Region for DR Only**
   - Use inter-region peering for disaster recovery
   - Not recommended for production traffic due to latency

5. **Tag Your Resources**
   - Use consistent tagging for VPCs, peering connections, etc.
   - Helps with cost allocation and management

### Security Best Practices

1. **Least Privilege Access**
   - Only allow necessary traffic between peered VPCs
   - Use specific IP ranges in security groups

2. **Network Segmentation**
   - Use different subnets for different tiers
   - Apply appropriate NACLs

3. **Monitor Traffic**
   - Enable VPC Flow Logs for all peered VPCs
   - Set up alerts for unusual traffic patterns

4. **Encrypt Sensitive Data**
   - Use TLS for application traffic
   - Consider MACsec for Layer 2 encryption (if available)

5. **Audit Regularly**
   - Review peering connections periodically
   - Remove unused connections
   - Update security groups and NACLs

### Operational Best Practices

1. **Automate Peering Setup**
   - Use Infrastructure as Code (Terraform, CloudFormation)
   - Automate route table updates

2. **Monitor Connection Status**
   - Set up alerts for peering connection state changes
   - Monitor data transfer volumes

3. **Document Your Network**
   - Maintain a network diagram
   - Document CIDR blocks, peering connections, and routing

4. **Test Failover**
   - Regularly test cross-region connectivity
   - Verify disaster recovery procedures

5. **Plan for Growth**
   - Consider Transit Gateway if you expect >50 VPCs
   - Plan CIDR blocks for future expansion

## 🔗 Further Reading

### AWS
- [AWS VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [VPC Peering Walkthrough](https://docs.aws.amazon.com/vpc/latest/peering/peering-configurations-full-access.html)
- [VPC Peering Limitations](https://docs.aws.amazon.com/vpc/latest/peering/invalid-peering-configurations.html)
- [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html)
- [PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)

### Azure
- [Azure VNet Peering Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [Global VNet Peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview#global-vnet-peering)
- [VNet Peering Troubleshooting](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-troubleshoot)
- [Virtual Network Gateway](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways)

### Google Cloud
- [VPC Network Peering Documentation](https://cloud.google.com/vpc/docs/vpc-peering)
- [Private Service Connect](https://cloud.google.com/vpc/docs/configure-private-service-connect-producer)
- [Shared VPC](https://cloud.google.com/vpc/docs/shared-vpc)
- [VPC Peering Best Practices](https://cloud.google.com/architecture/best-practices-vpc-design)

### Multi-Cloud
- [Multi-Cloud Networking Patterns](https://cloud.google.com/architecture/multi-cloud-networking-patterns)
- [Hybrid and Multi-Cloud Networking](https://aws.amazon.com/solutions/implementations/hybrid-multi-cloud-networking/)
- [VPC Peering Across Cloud Providers](https://www.hashicorp.com/blog/multi-cloud-networking-with-terraform)

## 🧩 Hands-On Labs

### AWS VPC Peering Lab

1. **Create two VPCs** with non-overlapping CIDR blocks
2. **Create peering connection** between them
3. **Accept the peering connection**
4. **Update route tables** in both VPCs
5. **Update security groups** to allow traffic
6. **Launch EC2 instances** in both VPCs
7. **Test connectivity** between instances

### Terraform Example

```hcl
# VPC A
resource "aws_vpc" "vpc_a" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "VPC-A"
  }
}

# VPC B
resource "aws_vpc" "vpc_b" {
  cidr_block = "10.1.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "VPC-B"
  }
}

# Peering Connection
resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.vpc_a.id
  peer_vpc_id   = aws_vpc.vpc_b.id
  peer_owner_id = data.aws_caller_identity.current.account_id
  auto_accept   = true
  tags = {
    Name = "VPC-A-to-VPC-B"
  }
}

# Route from A to B
resource "aws_route" "a_to_b" {
  route_table_id            = aws_vpc.vpc_a.main_route_table_id
  destination_cidr_block    = aws_vpc.vpc_b.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}

# Route from B to A
resource "aws_route" "b_to_a" {
  route_table_id            = aws_vpc.vpc_b.main_route_table_id
  destination_cidr_block    = aws_vpc.vpc_a.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}

# Security Group for VPC A
resource "aws_security_group" "vpc_a" {
  name        = "allow-from-vpc-b"
  description = "Allow traffic from VPC B"
  vpc_id      = aws_vpc.vpc_a.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.vpc_b.cidr_block]
  }
}
```

## 🎯 Key Takeaways

1. **VPC peering enables private connectivity** between VPCs without public internet
2. **Peering is non-transitive** - Use Transit Gateway for transitive routing
3. **CIDRs must not overlap** - Plan your IP address space carefully
4. **AWS, Azure, and GCP all support peering** with some differences
5. **Update route tables and security groups** for traffic to flow
6. **Monitor your peering connections** for performance and security
7. **Use hub-and-spoke for many VPCs** to reduce management overhead
8. **Cross-region peering is good for DR** but has higher latency
9. **Cost depends on data transfer** - Minimize cross-region traffic
10. **Automate with IaC** - Use Terraform, CloudFormation, or similar

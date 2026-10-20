## AWS VPC Peering — Student Project Report

**Name:** Sushant Balunda
**Batch:** B.Tech CSE (Core), JECRC University
**Project Title:** AWS VPC Peering (Project 01 / Oct 2026)
**Region:** Asia Pacific (Mumbai) — `ap-south-1`

### 1. Architecture & Resource Inventory

| Resource / Name | Type | CIDR / IP Address |
| --- | --- | --- |
| `ewb-vpc-a` | VPC A | `10.10.0.0/16`<br> |
| `ewb-a-public` | Public Subnet (VPC A) | `10.10.1.0/24`<br> |
| `ewb-a-private` | Private Subnet (VPC A) | `10.10.2.0/24`<br> |
| `ewb-vpc-b` | VPC B | `10.20.0.0/16`<br> |
| `ewb-b-private` | Private Subnet (VPC B) | `10.20.1.0/24`<br> |
| `ewb-bastion` | EC2 (Public) | `10.10.1.10` (Auto-assign Public IP: Enabled)

 |
| `ewb-app-a` | EC2 (Private) | `10.10.2.10` (No Public IP)

 |
| `ewb-app-b` | EC2 (Private) | `10.20.1.10` (No Public IP)

 |
| `ewb-a-b-peer` | Peering Connection | Active

 |

### 2. Route Tables & Subnet Associations

| Route Table | Associated Subnet | Destination | Target | Purpose |
| --- | --- | --- | --- | --- |
| `ewb-rt-a-public`<br> | `ewb-a-public`<br> | `0.0.0.0/0`<br> | `ewb-igw-a`<br> | Bastion internet access

 |
| `ewb-rt-a-public`<br> | `ewb-a-public`<br> | `10.20.0.0/16`<br> | `pcx-...`<br> | Bastion SSH to App-B

 |
| `ewb-rt-a-private`<br> | `ewb-a-private`<br> | `10.20.0.0/16`<br> | `pcx-...`<br> | App-A to VPC B

 |
| `ewb-rt-b-private`<br> | `ewb-b-private`<br> | `10.10.0.0/16`<br> | `pcx-...`<br> | Return path to VPC A

 |

### 3. Security Group Rules (Inbound)

* **`ewb-sg-bastion`**: Allows `SSH` (TCP 22) from the local administrator's public IP.


* **`ewb-sg-app-a`**: Allows `SSH` (TCP 22) from `10.10.1.10/32` (Bastion). Allows `Custom ICMP - IPv4` (Echo Request) from `10.20.1.10/32` (App-B).


* **`ewb-sg-app-b`**: Allows `SSH` (TCP 22) from `10.10.1.10/32` (Bastion). Allows `Custom ICMP - IPv4` (Echo Request) from `10.10.2.10/32` (App-A).



### 4. Connectivity Explanations & Test Results

* **Forward Path (`ewb-app-a` $\rightarrow$ `ewb-app-b`):** Traffic originates from `10.10.2.10` destined for `10.20.1.10`. The private route table in VPC A successfully directs the `10.20.0.0/16` traffic to the active peering connection. The `ewb-sg-app-b` security group permits the inbound ICMP request from `10.10.2.10/32`. *Result: 0% packet loss.*


* **Return Path (`ewb-app-b` $\rightarrow$ `ewb-app-a`):** Traffic originates from `10.20.1.10` destined for `10.10.2.10`. The private route table in VPC B directs `10.10.0.0/16` traffic back across the peering connection. The `ewb-sg-app-a` security group permits the inbound ICMP request from `10.20.1.10/32`. *Result: 0% packet loss.*


* **Controlled Failure & Recovery:** Temporarily deleting the inbound ICMP Echo Request rule from `ewb-sg-app-b` resulted in 100% packet loss from App-A. This confirms that an Active peering connection does not guarantee traffic flow without explicit, stateful firewall permissions at the destination instance. Restoring the exact rule immediately recovered the connection to 0% packet loss.



### 5. Cleanup & Learnings

All lab-specific instances, security groups, peer connections, route tables, subnets, internet gateways, and VPCs were deleted to prevent unintended AWS charges. This architecture demonstrated that VPC peering is non-transitive and requires precise, bidirectional routing and tightly scoped security group rules to securely connect isolated workloads using private IP addresses.

(Combine this text with your captured screenshots matching the sections above, save as `SushantBalunda_EWB_VPC_Peering.pdf`, and your university submission is complete.)

---

## GitHub Repository `README.md`

```markdown
# AWS VPC Peering & Private Workload Connectivity

## Project Overview
This project demonstrates the implementation of a secure, private network connection between two completely isolated Amazon Virtual Private Clouds (VPCs) in the AWS `ap-south-1` (Mumbai) region. It proves that internal instances can communicate securely via private IPv4 addresses without traversing the public internet.

## Architecture
- **VPC A (`10.10.0.0/16`)**: Houses a public Bastion host (`10.10.1.10`) for secure SSH administration and a private application server `App-A` (`10.10.2.10`).
- **VPC B (`10.20.0.0/16`)**: Houses a fully isolated application server `App-B` (`10.20.1.10`) with no Internet Gateway attached.
- **Connection**: A direct AWS VPC Peering Connection linking the two non-overlapping CIDR blocks.

## Key Technical Configurations
1. **Routing**: Custom private route tables on both sides actively direct traffic destined for the remote VPC's CIDR directly into the peering connection.
2. **Security Groups (Stateful Firewalls)**: Inbound rules strictly permit `TCP 22` (SSH) only from the Bastion's private IP (`10.10.1.10/32`), and ICMP Echo Requests are explicitly restricted to the exact `/32` private IP of the peered instance.
3. **Administration**: Implemented SSH ProxyJump via the Windows `~/.ssh/config` file, allowing seamless local terminal access to fully private instances through the Bastion host.

## Validation & Testing
- ✅ **Baseline Failure**: Verified 100% packet loss prior to peering activation.
- ✅ **Forward & Reverse Path**: Achieved 0% packet loss during bidirectional ICMP testing across the peering link.
- ✅ **Controlled Isolation**: Proved network isolation by temporarily removing stateful ICMP rules to simulate and recover from a connection failure.

```

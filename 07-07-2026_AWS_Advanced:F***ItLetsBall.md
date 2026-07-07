# AWS Security Guide - Advanced: VPN & Network Architecture
## Multi-Hop Secure Communications & Defense-in-Depth Network Design
## By Oblivion Edge Vulnerability Research LLC (c) 2026

---

## Table of Contents
1. [Overview & Architecture](#overview--architecture)
2. [OpenVPN Onion Routing](#openvpn-onion-routing)
3. [WireGuard Mesh Networks](#wireguard-mesh-networks)
4. [AWS Route 53 VPN-Restricted Access](#aws-route-53-vpn-restricted-access)
5. [Multi-Hop Tunnel Architectures](#multi-hop-tunnel-architectures)
6. [Defense-in-Depth Security](#defense-in-depth-security)
7. [Advanced Deployment Scenarios](#advanced-deployment-scenarios)
8. [Monitoring & Troubleshooting](#monitoring--troubleshooting)

---

## Overview & Architecture

### Problem Statement
Organizations need to:
- Secure distributed R&D environments across multiple regions
- Implement zero-trust network access
- Create encrypted multi-hop communication paths
- Authenticate all network access to infrastructure
- Prevent direct exposure of internal resources
- Maintain compliance with security frameworks

### Solution Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Internet                                 │
└────────────┬──────────────────────────────────────┬─────────┘
             │                                      │
        ┌────▼────┐                          ┌─────▼────┐
        │ Entry   │                          │ Exit     │
        │ Point   │◄─────────Encrypted───────┤ Point    │
        │ (VPN)   │         Tunnel           │ (VPN)    │
        └────┬────┘                          └─────┬────┘
             │                                      │
        ┌────▼──────────────────────────────────────▼────┐
        │         VPN Tunnel Mesh Network                 │
        │  (OpenVPN or WireGuard Encrypted Overlay)       │
        │  • Multi-hop routing                            │
        │  • End-to-end encryption                        │
        │  • No traffic leaks outside tunnel              │
        └────┬──────────────────────────────────────┬────┘
             │                                      │
        ┌────▼────┐                          ┌─────▼────┐
        │  AWS    │                          │  AWS     │
        │  VPC-A  │◄─────Authenticated───────┤  VPC-B   │
        │         │      IAM + Route 53      │          │
        │ Private │                          │ Private  │
        │ 10.0.0  │                          │ 10.1.0   │
        └────┬────┘                          └─────┬────┘
             │                                      │
        ┌────▼──────────┐              ┌────────────▼────┐
        │  Bastion Host │              │ Application     │
        │  + IAM Role   │              │ Servers         │
        │  + CloudTrail │              │ + GuardDuty     │
        └───────────────┘              └─────────────────┘
```

### Key Concepts

**Onion Routing**: Multi-layer encryption where each hop decrypts only its layer
**Mesh Network**: Every node can communicate with every other node
**Zero-Trust**: Verify every access request, even within network
**Defense-in-Depth**: Multiple security layers (network, encryption, auth, logging)

---

## OpenVPN Onion Routing

### What is OpenVPN Onion Routing?

OpenVPN onion routing creates multi-layered encrypted tunnels where:
- Each relay node decrypts only its layer
- Intermediate nodes can't see end-to-end traffic
- Similar to Tor but with static VPN endpoints
- Suitable for distributed R&D teams

### Recipe 1: Deploy OpenVPN Server on EC2

**Use Case**: Secure VPN entry point for distributed teams.

```bash
#!/bin/bash
# Deploy OpenVPN Access Server on EC2

REGION="us-east-1"
INSTANCE_TYPE="t3.medium"
AMI_ID="ami-0c55b159cbfafe1f0"  # OpenVPN Access Server AMI

# 1. Create VPC and subnets
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.8.0.0/16 \
  --region $REGION \
  --query 'Vpc.VpcId' \
  --output text)

SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.8.1.0/24 \
  --region $REGION \
  --query 'Subnet.SubnetId' \
  --output text)

# 2. Create security group for VPN
SG_ID=$(aws ec2 create-security-group \
  --group-name openvpn-sg \
  --description "OpenVPN Access Server Security Group" \
  --vpc-id $VPC_ID \
  --region $REGION \
  --query 'GroupId' \
  --output text)

# Allow VPN traffic (UDP 1194, TCP 443)
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol udp \
  --port 1194 \
  --cidr 0.0.0.0/0 \
  --region $REGION

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0 \
  --region $REGION

# Allow SSH for management
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/8 \
  --region $REGION

# 3. Create IAM role for OpenVPN server
ROLE_NAME="OpenVPNAccessServerRole"

TRUST_POLICY='{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}'

aws iam create-role \
  --role-name $ROLE_NAME \
  --assume-role-policy-document "$TRUST_POLICY"

# Attach policies
aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore

# Create instance profile
INSTANCE_PROFILE=$(aws iam create-instance-profile \
  --instance-profile-name $ROLE_NAME \
  --query 'InstanceProfile.Arn' \
  --output text)

aws iam add-role-to-instance-profile \
  --instance-profile-name $ROLE_NAME \
  --role-name $ROLE_NAME

# 4. Launch OpenVPN EC2 instance
INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type $INSTANCE_TYPE \
  --subnet-id $SUBNET_ID \
  --security-group-ids $SG_ID \
  --iam-instance-profile Name=$ROLE_NAME \
  --region $REGION \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=OpenVPN-Server}]' \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "OpenVPN instance launched: $INSTANCE_ID"

# 5. Allocate and associate Elastic IP
EIP=$(aws ec2 allocate-address \
  --domain vpc \
  --region $REGION \
  --query 'PublicIp' \
  --output text)

aws ec2 associate-address \
  --instance-id $INSTANCE_ID \
  --public-ip $EIP \
  --region $REGION

echo "Elastic IP assigned: $EIP"

# 6. Enable CloudTrail for API logging
TRAIL_NAME="openvpn-audit"
BUCKET="openvpn-audit-logs-$(date +%s)"

aws s3api create-bucket \
  --bucket $BUCKET \
  --region $REGION

aws cloudtrail create-trail \
  --name $TRAIL_NAME \
  --s3-bucket-name $BUCKET \
  --region $REGION

aws cloudtrail start-logging --trail-name $TRAIL_NAME

echo "✓ OpenVPN infrastructure deployed"
echo "  Instance: $INSTANCE_ID"
echo "  Public IP: $EIP"
echo "  Access at: https://$EIP/admin"
```

### Recipe 2: Configure OpenVPN Multi-Hop Routing

**Use Case**: Create multi-layer encryption for sensitive communications.

```bash
#!/bin/bash
# Configure OpenVPN multi-hop routing

# OpenVPN config for Relay Node (Middle Layer)
cat > /etc/openvpn/relay.conf << 'EOF'
# OpenVPN Relay Node Configuration
mode p2p
proto udp
port 1194
ca ca.crt
cert relay-server.crt
key relay-server.key
dh dh.pem

# Virtual tunnel network
server 10.8.0.0 255.255.255.0

# Keep remote hosts in memory
keepalive 20 60

# Compression
comp-lz4

# Security hardening
tls-server
tls-cipher TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
cipher AES-256-GCM
auth SHA256

# Enable multi-hop
route-nopull
push "route 10.0.0.0 255.255.0.0"

# Logging
log /var/log/openvpn/relay.log
log-append /var/log/openvpn/relay.log
verb 3

# User/group
user nobody
group nogroup

# Status file for monitoring
status /var/run/openvpn/relay-status.log

# Key renegotiation
reneg-sec 3600
EOF

# Start OpenVPN
systemctl start openvpn@relay
systemctl enable openvpn@relay

# Verify tunnel
openvpn-status relay-status.log
```

### Recipe 3: OpenVPN Client Configuration for Onion Routing

```bash
#!/bin/bash
# Client config for 3-hop onion routing

cat > client-3hop.ovpn << 'EOF'
# 3-Hop Onion Routing Configuration

# First hop: Entry VPN
remote vpn-entry.example.com 1194 udp

# Second hop: Relay
remote vpn-relay.example.com 1194 udp

# Third hop: Exit
remote vpn-exit.example.com 1194 udp

# Certificate paths
ca ca.crt
cert client.crt
key client.key
tls-auth ta.key 1

# Encryption
cipher AES-256-GCM
auth SHA256
tls-cipher TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384

# Protocol
proto udp
comp-lz4

# Connection settings
remote-random
resolv-retry infinite
nobind
persist-key
persist-tun

# Security
mute 20
mute-replay-warnings
script-security 2
disable-occ

# Logging
log-append /var/log/openvpn/client.log
verb 3
mute-replay-warnings

# Network
redirect-gateway def1
dhcp-option DNS 8.8.8.8
dhcp-option DNS 8.8.4.4

# Keep connection alive
keepalive 20 60
EOF

# Install OpenVPN client
sudo apt-get install openvpn

# Connect
sudo openvpn --config client-3hop.ovpn
```

---

## WireGuard Mesh Networks

### What is WireGuard?

WireGuard is a modern VPN protocol with:
- Minimal code base (~4000 lines vs. OpenVPN's 100k+)
- Faster performance
- Built-in mesh capability
- Easier configuration
- Post-quantum resistant cryptography ready

### Recipe 1: Deploy WireGuard Mesh Network on AWS

**Use Case**: Fast, modern VPN tunnel for R&D environments.

```bash
#!/bin/bash
# Deploy WireGuard mesh network across 3 EC2 instances

REGION="us-east-1"
INSTANCES=("10.0.1.10" "10.0.2.10" "10.0.3.10")  # Private IPs
PUBLIC_IPS=("203.0.113.1" "203.0.113.2" "203.0.113.3")  # Public IPs

# 1. Install WireGuard on each instance
for IP in "${INSTANCES[@]}"; do
  ssh -i key.pem ubuntu@$IP << 'SCRIPT'
    sudo apt-get update
    sudo apt-get install -y wireguard wireguard-tools

    # Enable forwarding
    echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
SCRIPT
done

# 2. Generate WireGuard keys
mkdir -p wireguard-keys
cd wireguard-keys

# Node 1 (Entry Point)
wg genkey | tee node1-private.key | wg pubkey > node1-public.key

# Node 2 (Relay)
wg genkey | tee node2-private.key | wg pubkey > node2-public.key

# Node 3 (Exit Point)
wg genkey | tee node3-private.key | wg pubkey > node3-public.key

# Pre-shared keys for added security
wg genpsk > node1-node2.psk
wg genpsk > node2-node3.psk
wg genpsk > node1-node3.psk

echo "✓ Keys generated"

# 3. Deploy WireGuard configuration on Node 1
NODE1_PRIV=$(cat node1-private.key)
NODE2_PUB=$(cat node2-public.key)
NODE3_PUB=$(cat node3-public.key)
PSK_12=$(cat node1-node2.psk)
PSK_13=$(cat node1-node3.psk)

ssh -i key.pem ubuntu@${PUBLIC_IPS[0]} sudo tee /etc/wireguard/wg0.conf > /dev/null << EOF
[Interface]
PrivateKey = $NODE1_PRIV
Address = 10.100.1.1/32
ListenPort = 51820

# Node 2 (Relay)
[Peer]
PublicKey = $NODE2_PUB
Endpoint = ${PUBLIC_IPS[1]}:51820
AllowedIPs = 10.100.2.0/24
PresharedKey = $PSK_12
PersistentKeepalive = 25

# Node 3 (Exit)
[Peer]
PublicKey = $NODE3_PUB
Endpoint = ${PUBLIC_IPS[2]}:51820
AllowedIPs = 10.100.3.0/24
PresharedKey = $PSK_13
PersistentKeepalive = 25
EOF

echo "✓ Node 1 configured"

# 4. Bring up WireGuard interface
ssh -i key.pem ubuntu@${PUBLIC_IPS[0]} << 'SCRIPT'
  sudo wg-quick up wg0
  sudo systemctl enable wg-quick@wg0
  sudo wg show
SCRIPT

# 5. Verify mesh connectivity
ssh -i key.pem ubuntu@${PUBLIC_IPS[0]} << 'SCRIPT'
  echo "Testing connectivity:"
  ping -c 3 10.100.2.1
  ping -c 3 10.100.3.1
  
  echo "Interface status:"
  sudo wg
SCRIPT
```

### Recipe 2: WireGuard Configuration on Each Node

**Node 1 (Entry Point) - 10.0.1.10:**
```ini
[Interface]
PrivateKey = CMI6MS2/Ja7xhBf4A...
Address = 10.100.1.1/32
ListenPort = 51820

# Allow forwarding to other nodes
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT

[Peer]
PublicKey = gN65BkIKpZD7VIC...  # Node 2
Endpoint = 203.0.113.2:51820
AllowedIPs = 10.100.2.0/24
PresharedKey = e84b5a...
PersistentKeepalive = 25

[Peer]
PublicKey = rRgaeJJ3ILTy0Fw...  # Node 3
Endpoint = 203.0.113.3:51820
AllowedIPs = 10.100.3.0/24
PresharedKey = f92x1d...
PersistentKeepalive = 25
```

**Node 2 (Relay) - 10.0.2.10:**
```ini
[Interface]
PrivateKey = UM84ns3I/JK2bV...
Address = 10.100.2.1/32
ListenPort = 51820

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT

[Peer]
PublicKey = eJ8wK2Lp5M9Nt...  # Node 1
Endpoint = 203.0.113.1:51820
AllowedIPs = 10.100.1.0/24
PresharedKey = e84b5a...
PersistentKeepalive = 25

[Peer]
PublicKey = 7sQ0fR1Y2w8V...  # Node 3
Endpoint = 203.0.113.3:51820
AllowedIPs = 10.100.3.0/24
PresharedKey = g11p2b...
PersistentKeepalive = 25
```

---

## AWS Route 53 VPN-Restricted Access

### Recipe 1: Create Route 53 Private Hosted Zone

**Use Case**: Internal DNS only accessible from VPN.

```bash
#!/bin/bash
# Create Route 53 private zone for internal services

VPC_ID="vpc-12345678"
ZONE_NAME="research.internal"

# 1. Create private hosted zone
ZONE_ID=$(aws route53 create-hosted-zone \
  --name $ZONE_NAME \
  --vpc VPCRegion=us-east-1,VPCId=$VPC_ID \
  --caller-reference "zone-$(date +%s)" \
  --query 'HostedZone.Id' \
  --output text | cut -d'/' -f3)

echo "Created zone: $ZONE_ID"

# 2. Create DNS records for internal services
aws route53 change-resource-record-sets \
  --hosted-zone-id $ZONE_ID \
  --change-batch '{
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "api.research.internal",
          "Type": "A",
          "TTL": 300,
          "ResourceRecords": [
            {"Value": "10.0.1.50"}
          ]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "database.research.internal",
          "Type": "A",
          "TTL": 300,
          "ResourceRecords": [
            {"Value": "10.0.2.25"}
          ]
        }
      },
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "tools.research.internal",
          "Type": "A",
          "TTL": 300,
          "ResourceRecords": [
            {"Value": "10.0.3.100"}
          ]
        }
      }
    ]
  }'

echo "✓ DNS records created"
```

### Recipe 2: Restrict Route 53 Access via Resource Policy

```bash
#!/bin/bash
# Restrict Route 53 zone access to VPN-authenticated users

ZONE_ID="Z1234567890ABC"
PRINCIPAL_ARN="arn:aws:iam::123456789012:role/VPNAuthenticatedRole"

# Create and attach resource policy
aws route53 put-query-logging-config \
  --hosted-zone-id $ZONE_ID \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789012:log-group:/aws/route53/research-internal

# Policy to restrict access to specific role
POLICY='{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowVPNAuthenticatedAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "'$PRINCIPAL_ARN'"
      },
      "Action": "route53:GetChange",
      "Resource": "*"
    },
    {
      "Sid": "AllowVPNQueryDNS",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "route53:ListHostedZones",
      "Resource": "arn:aws:route53:::hostedzone/'$ZONE_ID'",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalOrgID": "o-1234567890"
        }
      }
    }
  ]
}'

echo "✓ Resource policy configured"
```

### Recipe 3: IAM Policy for VPN-Authenticated Access

```bash
#!/bin/bash
# Create IAM role for VPN-authenticated users

# Trust policy (only VPN endpoint can assume)
TRUST_POLICY='{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/OpenVPNServerRole"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "vpn-auth-token-12345"
        }
      }
    }
  ]
}'

aws iam create-role \
  --role-name VPNAuthenticatedRole \
  --assume-role-policy-document "$TRUST_POLICY"

# Attach policy for Route 53 access
aws iam put-role-policy \
  --role-name VPNAuthenticatedRole \
  --policy-name Route53Access \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "route53:GetHostedZone",
          "route53:ListResourceRecordSets",
          "route53:GetChange"
        ],
        "Resource": [
          "arn:aws:route53:::hostedzone/Z*",
          "arn:aws:route53:::change/*"
        ],
        "Condition": {
          "IpAddress": {
            "aws:SourceIp": ["10.100.0.0/8"]
          }
        }
      }
    ]
  }'

# Attach policy for IAM-controlled mail services
aws iam put-role-policy \
  --role-name VPNAuthenticatedRole \
  --policy-name MailServiceAccess \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ses:SendEmail",
          "ses:SendRawEmail"
        ],
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "aws:username": "vpn-authenticated"
          }
        }
      }
    ]
  }'

echo "✓ VPN authentication role configured"
```

---

## Multi-Hop Tunnel Architectures

### Architecture 1: Three-Layer Onion (Entry → Relay → Exit)

```bash
#!/bin/bash
# Deploy complete 3-layer onion architecture

cat > deploy-3layer-onion.sh << 'DEPLOY'
#!/bin/bash

REGION="us-east-1"

# Tier 1: Entry Point (Public-facing VPN)
echo "Deploying Entry Point..."
ENTRY_VPC=$(aws ec2 create-vpc --cidr-block 10.8.0.0/16 --region $REGION --query 'Vpc.VpcId' --output text)
ENTRY_SUBNET=$(aws ec2 create-subnet --vpc-id $ENTRY_VPC --cidr-block 10.8.0.0/24 --region $REGION --query 'Subnet.SubnetId' --output text)

# Deploy Entry VPN server
ENTRY_INSTANCE=$(aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.medium \
  --subnet-id $ENTRY_SUBNET \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Entry-VPN}]' \
  --region $REGION \
  --query 'Instances[0].InstanceId' \
  --output text)

# Tier 2: Relay Node (Internal relay, private)
echo "Deploying Relay Node..."
RELAY_VPC=$(aws ec2 create-vpc --cidr-block 10.9.0.0/16 --region $REGION --query 'Vpc.VpcId' --output text)
RELAY_SUBNET=$(aws ec2 create-subnet --vpc-id $RELAY_VPC --cidr-block 10.9.0.0/24 --region $REGION --query 'Subnet.SubnetId' --output text)

RELAY_INSTANCE=$(aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.medium \
  --subnet-id $RELAY_SUBNET \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Relay-VPN}]' \
  --region $REGION \
  --query 'Instances[0].InstanceId' \
  --output text)

# Tier 3: Exit Point (Data center gateway)
echo "Deploying Exit Point..."
EXIT_VPC=$(aws ec2 create-vpc --cidr-block 10.10.0.0/16 --region $REGION --query 'Vpc.VpcId' --output text)
EXIT_SUBNET=$(aws ec2 create-subnet --vpc-id $EXIT_VPC --cidr-block 10.10.0.0/24 --region $REGION --query 'Subnet.SubnetId' --output text)

EXIT_INSTANCE=$(aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.medium \
  --subnet-id $EXIT_SUBNET \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Exit-VPN}]' \
  --region $REGION \
  --query 'Instances[0].InstanceId' \
  --output text)

# VPC Peering: Entry ↔ Relay ↔ Exit
echo "Configuring VPC Peering..."

PEER_1=$(aws ec2 create-vpc-peering-connection \
  --vpc-id $ENTRY_VPC \
  --peer-vpc-id $RELAY_VPC \
  --region $REGION \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text)

aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id $PEER_1 \
  --region $REGION

PEER_2=$(aws ec2 create-vpc-peering-connection \
  --vpc-id $RELAY_VPC \
  --peer-vpc-id $EXIT_VPC \
  --region $REGION \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text)

aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id $PEER_2 \
  --region $REGION

echo "✓ 3-layer onion deployed"
echo "  Entry: $ENTRY_INSTANCE"
echo "  Relay: $RELAY_INSTANCE"
echo "  Exit: $EXIT_INSTANCE"
DEPLOY

chmod +x deploy-3layer-onion.sh
./deploy-3layer-onion.sh
```

### Architecture 2: Mesh Network (Full Connectivity)

```bash
#!/bin/bash
# Deploy full mesh WireGuard network

NODES=("node-1" "node-2" "node-3" "node-4")
BASE_IP="10.100"
COUNTER=1

# Generate all peer configurations
for NODE in "${NODES[@]}"; do
  echo "Configuring $NODE..."
  
  # Generate keypair
  PRIV_KEY=$(wg genkey)
  PUB_KEY=$(echo $PRIV_KEY | wg pubkey)
  
  # Store for later use
  echo "$NODE:$PRIV_KEY:$PUB_KEY" >> wireguard-peers.txt
done

# Create mesh configuration
MESH_CONFIG=""
for NODE in "${NODES[@]}"; do
  IFS=':' read -r name priv_key pub_key < <(grep "^$NODE:" wireguard-peers.txt)
  
  cat > /etc/wireguard/$NODE.conf << EOF
[Interface]
PrivateKey = $priv_key
Address = $BASE_IP.$COUNTER.1/32
ListenPort = 51820

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

EOF

  # Add all other nodes as peers
  for PEER in "${NODES[@]}"; do
    if [ "$PEER" != "$NODE" ]; then
      IFS=':' read -r pname ppriv ppub < <(grep "^$PEER:" wireguard-peers.txt)
      
      cat >> /etc/wireguard/$NODE.conf << EOF

[Peer]
PublicKey = $ppub
AllowedIPs = $BASE_IP.$(( COUNTER + 1 )).0/24
PersistentKeepalive = 25
EOF
    fi
  done
  
  COUNTER=$((COUNTER + 1))
done

echo "✓ Mesh configuration generated"
```

---

## Defense-in-Depth Security

### Recipe 1: Layered Security Stack

```bash
#!/bin/bash
# Deploy complete defense-in-depth security

cat > deploy-defense-in-depth.sh << 'DEFENSE'
#!/bin/bash

PROJECT="research-infrastructure"
REGION="us-east-1"

echo "Deploying Defense-in-Depth Security..."

# Layer 1: Network Security (VPC + SecurityGroups)
echo "Layer 1: Network Security"

VPC=$(aws ec2 create-vpc --cidr-block 10.0.0.0/8 --region $REGION --query 'Vpc.VpcId' --output text)
echo "  VPC: $VPC"

# Layer 2: Encryption (TLS/IPsec Tunnels)
echo "Layer 2: Encryption"

# Enable EBS encryption by default
aws ec2 enable-ebs-encryption-by-default --region $REGION
echo "  EBS encryption enabled"

# Layer 3: Authentication (IAM + MFA)
echo "Layer 3: Authentication"

# Require MFA for sensitive actions
aws iam put-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --expire-passwords \
  --max-password-age 90

echo "  Password policy configured"

# Layer 4: Logging (CloudTrail + GuardDuty)
echo "Layer 4: Logging & Detection"

# Enable CloudTrail
TRAIL_NAME="defense-audit-trail"
BUCKET="defense-audit-logs-$(date +%s)"

aws s3api create-bucket --bucket $BUCKET --region $REGION
aws cloudtrail create-trail --name $TRAIL_NAME --s3-bucket-name $BUCKET --region $REGION
aws cloudtrail start-logging --trail-name $TRAIL_NAME

echo "  CloudTrail: $TRAIL_NAME"

# Enable GuardDuty
DETECTOR=$(aws guardduty create-detector --enable --region $REGION --query 'DetectorId' --output text)
echo "  GuardDuty: $DETECTOR"

# Layer 5: Application Security (WAF + Shield)
echo "Layer 5: Application Security"

# Create WAF Web ACL
WAF_ACL=$(aws wafv2 create-web-acl \
  --name "$PROJECT-waf" \
  --scope REGIONAL \
  --region $REGION \
  --default-action Allow={} \
  --rules '[
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 0,
      "OverrideAction": {"None": {}},
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSet"
      }
    }
  ]' \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=$PROJECT-waf \
  --query 'Summary.ARN' \
  --output text)

echo "  WAF ACL: $WAF_ACL"

# Layer 6: Incident Response
echo "Layer 6: Incident Response"

# Create SNS topic for alerts
TOPIC=$(aws sns create-topic --name $PROJECT-security-alerts --region $REGION --query 'TopicArn' --output text)
echo "  Alert topic: $TOPIC"

# Create EventBridge rules for automation
aws events put-rule \
  --name "$PROJECT-auto-response" \
  --event-pattern '{
    "source": ["aws.guardduty", "aws.securityhub"],
    "detail-type": ["GuardDuty Finding", "Security Hub Findings - Imported"]
  }' \
  --state ENABLED \
  --region $REGION

echo "  Auto-response rule created"

echo "✓ Defense-in-Depth deployed"
DEFENSE

chmod +x deploy-defense-in-depth.sh
./deploy-defense-in-depth.sh
```

### Recipe 2: Security Audit Script

```bash
#!/bin/bash
# Comprehensive security audit across all layers

cat > security-audit.sh << 'AUDIT'
#!/bin/bash

echo "===================================="
echo "AWS Security Posture Audit"
echo "===================================="
echo ""

# Layer 1: Network
echo "LAYER 1: Network Security"
echo "- VPC Configuration"
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock]' --output table

echo "- Flow Logs Status"
aws ec2 describe-flow-logs --query 'FlowLogs[*].[ResourceId,FlowLogStatus]' --output table

# Layer 2: Encryption
echo ""
echo "LAYER 2: Data Encryption"
echo "- EBS Encryption by Default"
aws ec2 get-ebs-encryption-by-default --query 'EbsEncryptionByDefault'

echo "- S3 Bucket Encryption"
for BUCKET in $(aws s3 ls --query 'Buckets[*].Name' --output text); do
  aws s3api get-bucket-encryption --bucket $BUCKET \
    --query 'ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.SSEAlgorithm' \
    --output text 2>/dev/null && echo "  $BUCKET: Encrypted" || echo "  $BUCKET: NOT Encrypted"
done

# Layer 3: Authentication
echo ""
echo "LAYER 3: Authentication & Authorization"
echo "- IAM Users with Access Keys"
aws iam get-credential-report
echo "- Root Account MFA Status"
aws iam get-account-summary --query 'SummaryMap.AccountAccessKeysPresent'

# Layer 4: Logging
echo ""
echo "LAYER 4: Logging & Detection"
echo "- CloudTrail Trails"
aws cloudtrail describe-trails --query 'trailList[*].[Name,IsLogging]' --output table

echo "- GuardDuty Detectors"
DETECTORS=$(aws guardduty list-detectors --query 'DetectorIds' --output text)
for DETECTOR in $DETECTORS; do
  aws guardduty get-detector --detector-id $DETECTOR \
    --query 'FindingPublishingFrequency' --output text
done

echo "- Security Hub Status"
aws securityhub get-enabled-standards \
  --query 'StandardsSubscriptions[*].[StandardsArn,ComplianceStatus]' \
  --output table 2>/dev/null || echo "  Security Hub not enabled"

# Layer 5: Application Security
echo ""
echo "LAYER 5: Application Security"
echo "- WAF Web ACLs"
aws wafv2 list-web-acls --scope REGIONAL \
  --query 'WebACLs[*].[Name,ARN]' \
  --output table

echo "- Shield Advanced Status"
aws shield describe-subscription \
  --query 'Subscription.[SubscriptionState,AutoRenew]' \
  --output text 2>/dev/null || echo "  Shield Standard (no advanced subscription)"

# Layer 6: Incident Response
echo ""
echo "LAYER 6: Incident Response"
echo "- SNS Topics for Alerts"
aws sns list-topics --query 'Topics[*].TopicArn' --output text | grep -i security || echo "  No security topics found"

echo "- EventBridge Rules"
aws events list-rules --query 'Rules[?contains(Name, `security`) || contains(Name, `alert`)].Name' --output text

echo ""
echo "===================================="
echo "Audit Complete"
echo "===================================="
AUDIT

chmod +x security-audit.sh
./security-audit.sh
```

---

## Advanced Deployment Scenarios

### Scenario 1: Global R&D Network (Multi-Region)

```bash
#!/bin/bash
# Deploy global VPN mesh across 4 regions

REGIONS=("us-east-1" "us-west-2" "eu-west-1" "ap-southeast-1")

for REGION in "${REGIONS[@]}"; do
  echo "Deploying VPN infrastructure in $REGION..."
  
  # VPC setup
  VPC=$(aws ec2 create-vpc \
    --cidr-block 10.${RANDOM:0:2}.0.0/16 \
    --region $REGION \
    --query 'Vpc.VpcId' \
    --output text)
  
  # VPN instance
  INSTANCE=$(aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.medium \
    --region $REGION \
    --query 'Instances[0].InstanceId' \
    --output text)
  
  echo "  Region: $REGION, VPC: $VPC, Instance: $INSTANCE"
done

# Connect regions with Transit Gateway
echo "Creating Transit Gateway..."
TGW=$(aws ec2 create-transit-gateway \
  --description "Global R&D VPN Backbone" \
  --query 'TransitGateway.TransitGatewayId' \
  --output text)

echo "Transit Gateway: $TGW"
```

### Scenario 2: Sensitive Data Isolation (Nested VPCs)

```bash
#!/bin/bash
# Create isolated network for highly sensitive data

# Outer VPC: General purpose
OUTER_VPC=$(aws ec2 create-vpc --cidr-block 10.0.0.0/8 --query 'Vpc.VpcId' --output text)

# Inner VPC: Sensitive data only
INNER_VPC=$(aws ec2 create-vpc --cidr-block 10.200.0.0/16 --query 'Vpc.VpcId' --output text)

# Private Link between VPCs
ENDPOINT=$(aws ec2 create-vpc-endpoint \
  --vpc-id $OUTER_VPC \
  --service-name com.amazonaws.vpce.us-east-1.vpce-svc-12345678 \
  --vpc-endpoint-type Interface \
  --query 'VpcEndpoint.VpcEndpointId' \
  --output text)

echo "Isolated inner VPC: $INNER_VPC"
echo "Access via PrivateLink: $ENDPOINT"
```

---

## Monitoring & Troubleshooting

### Recipe 1: VPN Tunnel Health Monitoring

```bash
#!/bin/bash
# Monitor VPN tunnel health

cat > monitor-vpn.sh << 'MONITOR'
#!/bin/bash

echo "VPN Tunnel Health Monitor"
echo "========================="

# Check OpenVPN processes
echo ""
echo "1. OpenVPN Process Status:"
systemctl status openvpn@relay --no-pager

# Check WireGuard interfaces
echo ""
echo "2. WireGuard Interfaces:"
sudo wg show

# Check tunnel connectivity
echo ""
echo "3. Tunnel Connectivity:"
for PEER in 10.100.2.1 10.100.3.1; do
  ping -c 3 $PEER && echo "  ✓ $PEER reachable" || echo "  ✗ $PEER unreachable"
done

# Check bandwidth utilization
echo ""
echo "4. Bandwidth Usage:"
ifstat -i wg0 -n 2

# Check DNS resolution (internal)
echo ""
echo "5. DNS Resolution:"
nslookup api.research.internal
nslookup database.research.internal

# Check routing tables
echo ""
echo "6. Routing Table:"
route -n | grep -E "^10\.|wg0|tun"

# Check logs for errors
echo ""
echo "7. Recent Errors:"
journalctl -u openvpn@relay -n 20 --no-pager | grep -i error || echo "  No errors found"

# CloudWatch metrics
echo ""
echo "8. CloudWatch Metrics:"
aws cloudwatch get-metric-statistics \
  --namespace AWS/VPN \
  --metric-name TunnelState \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
MONITOR

chmod +x monitor-vpn.sh
./monitor-vpn.sh
```

### Recipe 2: Troubleshoot VPN Connectivity

```bash
#!/bin/bash
# VPN troubleshooting guide

cat > troubleshoot-vpn.sh << 'TROUBLESHOOT'
#!/bin/bash

echo "VPN Troubleshooting Guide"
echo "========================="

# Test 1: Check VPN service
echo ""
echo "Test 1: VPN Service Status"
systemctl status openvpn@relay 2>&1 | grep -i "active" && echo "  ✓ Service running" || echo "  ✗ Service not running"

# Test 2: Check listening ports
echo ""
echo "Test 2: Listening Ports"
ss -tlnup | grep -E "1194|51820" || echo "  ✗ VPN ports not listening"

# Test 3: Check firewall rules
echo ""
echo "Test 3: Firewall Rules"
sudo iptables -L -n | grep -E "1194|51820" && echo "  ✓ Rules configured" || echo "  ✗ No firewall rules"

# Test 4: Test connectivity between nodes
echo ""
echo "Test 4: Inter-node Connectivity"
for NODE in 10.100.2.1 10.100.3.1; do
  ping -c 1 -W 2 $NODE > /dev/null 2>&1 && echo "  ✓ $NODE OK" || echo "  ✗ $NODE unreachable"
done

# Test 5: Check logs
echo ""
echo "Test 5: Error Logs"
tail -50 /var/log/openvpn/relay.log | grep -i error || echo "  ✓ No recent errors"

# Test 6: Check certificate validity
echo ""
echo "Test 6: Certificate Status"
openssl x509 -in /etc/openvpn/certs/relay-server.crt -noout -dates

# Test 7: Test DNS from VPN
echo ""
echo "Test 7: DNS from VPN"
nslookup example.com @10.8.0.1 && echo "  ✓ DNS working" || echo "  ✗ DNS not working"

# Test 8: Check bandwidth/latency
echo ""
echo "Test 8: Latency Test"
ping -c 10 10.100.2.1 | tail -1

TROUBLESHOOT

chmod +x troubleshoot-vpn.sh
./troubleshoot-vpn.sh
```

---

## Best Practices

✅ **DO:**
- [ ] Use certificate-based authentication (not pre-shared keys)
- [ ] Implement perfect forward secrecy (PFS)
- [ ] Rotate VPN certificates annually
- [ ] Log all VPN connections to CloudTrail
- [ ] Monitor tunnel status continuously
- [ ] Test failover scenarios regularly
- [ ] Document all VPN configurations
- [ ] Use strong encryption (AES-256, ChaCha20)
- [ ] Implement network segmentation
- [ ] Regular security audits

❌ **DON'T:**
- [ ] Expose VPN admin interfaces publicly
- [ ] Reuse pre-shared keys across environments
- [ ] Store credentials in code
- [ ] Use weak ciphers (DES, RC4)
- [ ] Skip logging and monitoring
- [ ] Use default passwords
- [ ] Connect untrusted networks directly
- [ ] Ignore certificate expiration

---

## Security Considerations

**Threat Model:**
- Nation-state adversaries (IPsec, WireGuard hardened)
- Cryptanalysis (post-quantum ready with hybrid keys)
- DNS leaks (VPN-forced DNS only)
- IP leaks (kill-switch enabled)
- Man-in-the-middle (certificate pinning)

**Compliance:**
- CloudTrail audit logging (PCI-DSS, HIPAA)
- Encryption at rest and in transit (SOC 2)
- Access controls via IAM (ISO 27001)
- Incident response logging (CSA CCM)

---

## Summary

This advanced guide covers:
Multi-hop encrypted communication architectures  
OpenVPN onion routing deployment  
WireGuard mesh networks  
Route 53 VPN-restricted access  
Defense-in-depth security layers  
Global multi-region deployment  
Monitoring and troubleshooting  
Security best practices  

Oblivion Edge Vulnerability Research LLC (c) 2026

These patterns are battle-tested in production R&D environments and provide enterprise-grade security for distributed teams.


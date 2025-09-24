# AWS Infrastructure Architecture Diagram (Free Tier Compatible)



## Architecture Overview (Free Tier Compatible)

### 🏗️ **Infrastructure Components**

**VPC (Virtual Private Cloud)**
- CIDR: 10.0.0.0/16
- DNS hostnames and support enabled
- Spans multiple Availability Zones

**Subnets**
- **Public Subnet** (10.0.1.0/24): Web server and jump host with internet access
- **Private Subnet 1** (10.0.2.0/24): General private resources (isolated)
- **Private Subnet 2** (10.0.3.0/24): Database subnet (isolated)

**Compute Resources**
- **EC2 Web Server**: t3.micro instance running Apache HTTP server
- **EC2 Jump Host**: t3.micro bastion host for secure private access
- **RDS PostgreSQL**: db.t3.micro database instance

### 🔒 **Security Architecture**

**Web Security Group**
- Inbound: HTTP (80) and HTTPS (443) from anywhere
- Outbound: All traffic allowed

**Jump Host Security Group**
- Inbound: SSH (22) from anywhere (can be restricted to your IP)
- Outbound: All traffic allowed

**Database Security Group**
- Inbound: PostgreSQL (5432) from Web Security Group
- Inbound: SSH (22) from Jump Host Security Group
- Outbound: All traffic allowed

### 🌐 **Network Flow**

1. **Web Traffic**: Internet → Internet Gateway → Web Server (HTTP/HTTPS)
2. **Database Access**: Web Server → RDS PostgreSQL (Port 5432)
3. **Admin Access**: Internet → Jump Host (SSH) → Private Resources (SSH)
4. **Private Subnets**: No direct internet access (isolated for security)

### 🛡️ **Security Features**

- **Network Isolation**: Database completely isolated in private subnets
- **Jump Host Pattern**: Secure bastion for private resource administration
- **Security Group Rules**: Restrict access between tiers
- **No NAT Gateway**: Private resources have no outbound internet (cost savings)
- **RDS Encryption**: Storage encryption enabled
- **Credential Security**: Database password stored in variables

### 💰 **Cost Optimization**

- **$0/month**: All resources use AWS Free Tier
- **No NAT Gateway**: Saves ~$45/month
- **Jump Host**: Free alternative for private access
- **t3.micro instances**: Free tier eligible





# AWS Infrastructure Architecture Diagram (Free Tier Compatible)

```mermaid
graph TB
    %% Internet and External Access
    Internet[🌐 Internet<br/>0.0.0.0/0]
    
    %% VPC Container
    subgraph VPC["🏢 VPC (10.0.0.0/16)"]
        %% Internet Gateway
        IGW[🚪 Internet Gateway<br/>main-igw]
        
        %% Availability Zones
        subgraph AZ1["📍 Availability Zone 1"]
            %% Public Subnet
            subgraph PubSub["🌐 Public Subnet<br/>10.0.1.0/24"]
                WebServer[💻 EC2 Web Server<br/>t3.micro<br/>Apache HTTP]
                JumpHost[🔑 Jump Host<br/>t3.micro<br/>Bastion]
            end
        end
        
        subgraph AZ2["📍 Availability Zone 2"]
            %% Private Subnet 1
            subgraph PrivSub1["🔒 Private Subnet<br/>10.0.2.0/24"]
                PrivateRes1[📦 Private Resources<br/>(No Internet Access)]
            end
        end
        
        subgraph AZ3["📍 Availability Zone 3"]
            %% Private Subnet 2 (DB)
            subgraph PrivSub2["🔒 Private DB Subnet<br/>10.0.3.0/24"]
                RDS[(🗄️ RDS PostgreSQL<br/>db.t3.micro<br/>maindb)]
            end
        end
        
        %% Route Tables
        PubRT[📋 Public Route Table<br/>→ 0.0.0.0/0 via IGW]
        PrivRT[📋 Private Route Table<br/>→ No Internet Route<br/>Isolated]
        
        %% Security Groups
        subgraph SG["🛡️ Security Groups"]
            WebSG[🌐 Web Security Group<br/>Inbound: 80,443 from 0.0.0.0/0<br/>Outbound: All traffic]
            JumpSG[🔑 Jump Host Security Group<br/>Inbound: 22 from 0.0.0.0/0<br/>Outbound: All traffic]
            DBSG[🔒 Database Security Group<br/>Inbound: 5432 from Web SG<br/>Inbound: 22 from Jump SG<br/>Outbound: All traffic]
        end
        
        %% DB Subnet Group
        DBSubnetGroup[📊 DB Subnet Group<br/>Private Subnets]
    end
    
    %% Connections
    Internet --> IGW
    IGW --> PubSub
    PubSub --> PubRT
    PrivSub1 --> PrivRT
    PrivSub2 --> PrivRT
    
    %% Security Group Associations
    WebServer -.-> WebSG
    JumpHost -.-> JumpSG
    RDS -.-> DBSG
    
    %% DB Subnet Group Association
    DBSubnetGroup -.-> PrivSub1
    DBSubnetGroup -.-> PrivSub2
    RDS -.-> DBSubnetGroup
    
    %% Traffic Flow
    WebServer -->|Port 5432| RDS
    JumpHost -->|SSH Port 22| PrivateRes1
    JumpHost -->|SSH Port 22| RDS
    WebSG -->|Allows DB Access| DBSG
    JumpSG -->|Allows SSH Access| DBSG
    
    %% User Access
    Internet -->|HTTP/HTTPS| WebServer
    Internet -->|SSH| JumpHost
    
    %% Styling
    classDef vpc fill:#e1f5fe,stroke:#01579b,stroke-width:3px
    classDef public fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef private fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    classDef security fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef database fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef jump fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    
    class VPC vpc
    class PubSub,WebServer,PubRT public
    class PrivSub1,PrivSub2,PrivRT,PrivateRes1 private
    class WebSG,DBSG,SG security
    class RDS,DBSubnetGroup database
    class JumpHost,JumpSG jump
```

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

## 🎨 **Excalidraw Recreation Guide**

### **Layout Structure:**
```
┌─────────────────────────────────────────────────────────────┐
│                        🌐 Internet                          │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────┴───────────────────────────────────┐
│                    VPC (10.0.0.0/16)                       │
│                                                             │
│  ┌─────────────────────┴─────────────────┐                 │
│  │         Internet Gateway              │                 │
│  └─────────────────────┬─────────────────┘                 │
│                        │                                   │
│  ┌─────────────────────┴─────────────────┐                 │
│  │      Public Subnet (10.0.1.0/24)     │                 │
│  │                                       │                 │
│  │  ┌─────────────┐    ┌─────────────┐   │                 │
│  │  │ Web Server  │    │ Jump Host   │   │                 │
│  │  │   (EC2)     │    │ (Bastion)   │   │                 │
│  │  │ HTTP/HTTPS  │    │    SSH      │   │                 │
│  │  └─────────────┘    └─────────────┘   │                 │
│  └─────────────────────┬─────────────────┘                 │
│                        │                                   │
│                        │ SSH Access                        │
│                        ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │           Private Subnets (10.0.2-3.0/24)              │ │
│  │                                                         │ │
│  │  ┌─────────────────────────────────────────────────────┐ │ │
│  │  │            PostgreSQL Database                      │ │ │
│  │  │               (RDS)                                 │ │ │
│  │  │            🔒 No Internet                           │ │ │
│  │  └─────────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### **Excalidraw Elements:**
1. **Large Rectangle**: VPC container (blue border)
2. **Cloud Shape**: Internet (gray)
3. **Rectangles**: Subnets (green for public, orange for private)
4. **Small Rectangles**: EC2 instances and RDS
5. **Arrows**: Traffic flow (solid for allowed, dashed for restricted)
6. **Shield Icons**: Security groups
7. **Labels**: Component names and IP ranges

### **Color Scheme:**
- **Blue**: VPC boundary
- **Green**: Public subnet and resources
- **Orange**: Private subnets and resources
- **Red**: Security group shields
- **Purple**: Database resources
- **Light Blue**: Jump host (special role)

This architecture provides enterprise-level security patterns while remaining completely free tier compatible!



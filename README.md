<div align="center">

# HotelOps Hospitality Management Cloud

### Enterprise Hotel Management Platform · Deployed on Amazon Web Services

**B.Tech CSE 2024–2028 · Semester IV · ITM Skills University**

---

**Live URL** · `http://32.236.113.239` · **Region** · `ap-southeast-2 Sydney` · **Cost** · `USD 0.00`

---

</div>

## What is HotelOps?

HotelOps is a centralized cloud platform that replaces disconnected hotel management spreadsheets and manual workflows with a fully automated, secure, and monitored AWS infrastructure.

The platform manages room inventory, guest records, bookings, and operational reporting — all running on AWS with Nginx, MySQL, Docker, CloudWatch monitoring, automated backups, and role-based access control.

---

## Architecture

```
                        ┌─────────────────────────────────────────────────┐
         Internet ───►  │                 HotelOps-VPC                    │
                        │                 10.0.0.0/16                     │
                        │                                                 │
    Internet Gateway    │   ┌─────────────────────────────────────────┐   │
    HotelOps-IGW  ───►  │   │           Public Subnet                  │   │
                        │   │           10.0.1.0/24                    │   │
                        │   │                                          │   │
                        │   │   ┌──────────────────────────────────┐   │   │
                        │   │   │      HotelOps-WebServer           │   │   │
                        │   │   │      Ubuntu 26.04 · t3.micro      │   │   │
                        │   │   │      32.236.113.239               │   │   │
                        │   │   │                                   │   │   │
                        │   │   │   Nginx :80    MySQL :3306        │   │   │
                        │   │   │   Docker :8080 CloudWatch         │   │   │
                        │   │   │   IAM    Cron  Git                │   │   │
                        │   │   └──────────────────────────────────┘   │   │
                        │   └─────────────────────────────────────────┘   │
                        │                                                 │
                        │   ┌─────────────────────────────────────────┐   │
                        │   │           Private Subnet                 │   │
                        │   │           10.0.2.0/24                    │   │
                        │   │           Reserved for RDS               │   │
                        │   └─────────────────────────────────────────┘   │
                        └─────────────────────────────────────────────────┘
                                  Security Group · HotelOps-Web-SG
                                  22 · 80 · 443 · 8080
```

---

## Tech Stack

| Layer | Technology | Configuration |
|---|---|---|
| Cloud Provider | Amazon Web Services | ap-southeast-2 · Sydney |
| Compute | EC2 t3.micro | 2 vCPU · 1 GB RAM · 8 GB gp3 |
| Operating System | Ubuntu 26.04 LTS | ami-06259b63260eddc13 |
| Web Server | Nginx | Port 80 · /var/www/html |
| Database | MySQL 8.0 | hotelops_db · 3 relational tables |
| Containers | Docker | hotelops-container · Port 8080 |
| Monitoring | AWS CloudWatch | CPU + Network · 80% alarm |
| Security | AWS IAM | 3 users · 2 groups · custom policy |
| Automation | Bash + Cron | Hourly health · Daily backup |
| Deployment | Git + SCP | Version control + file transfer |

---

## Features

### Networking
- VPC `HotelOps-VPC` with CIDR `10.0.0.0/16` — 65,536 available IPs
- Public subnet `10.0.1.0/24` for compute · Private subnet `10.0.2.0/24` reserved for DB
- Internet Gateway `HotelOps-IGW` with route `0.0.0.0/0`
- Security group `HotelOps-Web-SG` — ports 22, 80, 443, 8080

### Compute & Linux
- EC2 `HotelOps-WebServer` — Ubuntu 26.04 LTS · t3.micro
- SSH key-pair authentication via `HotelOps-Key.pem`
- Linux user management — `hotelops-manager` + `hotelops-staff` in `hotelops-team` group
- File permissions — `chmod 770 /opt/hotelops`
- System logs via `journalctl` and `/var/log/auth.log`

### Web Server
- Nginx serving the HotelOps hotel management portal on port 80
- 7-page dashboard — Dashboard, Bookings, Rooms, Guests, Reports, AWS Status, IAM
- Responsive HTML5 + CSS3 — no framework dependency

### Database
- MySQL with `hotelops_db` — rooms, guests, bookings tables
- Foreign key constraints enforce referential integrity
- Dedicated `hotelops_user` with least-privilege access to `hotelops_db` only

### Containerization
- Docker Engine installed and configured
- `hotelops-container` running Nginx image from Docker Hub
- Port mapping `8080:80` — accessible at `http://32.236.113.239:8080`

### Monitoring
- CloudWatch dashboard `HotelOps-Dashboard` — CPUUtilization + NetworkIn graphs
- Alarm `HotelOps-CPU-Alarm` — triggers at 80% CPU for 2 consecutive periods

### Automation
- `hotelops_health.sh` — logs CPU, memory, disk, service status every hour
- `hotelops_backup.sh` — mysqldump at midnight, 7-day auto-purge retention
- Both scheduled via `crontab` — zero manual intervention required

### Security
- IAM RBAC — three roles with tiered permissions
- Custom `HotelOps-Manager-Policy` written in JSON
- MySQL user restricted to single database
- SSH key-pair only — no password authentication

---

## Database Schema

```sql
CREATE TABLE rooms (
    room_id          INT AUTO_INCREMENT PRIMARY KEY,
    room_number      VARCHAR(10)    NOT NULL,
    room_type        VARCHAR(50),
    price_per_night  DECIMAL(10,2),
    status           VARCHAR(20)    DEFAULT 'available'
);

CREATE TABLE guests (
    guest_id    INT AUTO_INCREMENT PRIMARY KEY,
    full_name   VARCHAR(100)  NOT NULL,
    email       VARCHAR(100),
    phone       VARCHAR(20),
    created_at  TIMESTAMP     DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE bookings (
    booking_id    INT AUTO_INCREMENT PRIMARY KEY,
    guest_id      INT,
    room_id       INT,
    check_in      DATE,
    check_out     DATE,
    total_amount  DECIMAL(10,2),
    status        VARCHAR(20)  DEFAULT 'confirmed',
    FOREIGN KEY (guest_id) REFERENCES guests(guest_id),
    FOREIGN KEY (room_id)  REFERENCES rooms(room_id)
);
```

---

## IAM Access Control

```
AWS Account
│
├── hotelops-admin  ──►  HotelOps-Admins   ──►  EC2 Full Access
│                                               CloudWatch Full Access
│
├── hotelops-manager ──► HotelOps-Managers ──►  EC2 Describe (Read Only)
│                                               CloudWatch Read Only
│
└── hotelops-staff  ──►  HotelOps-Staff    ──►  CloudWatch Read Only
```

| User | Group | Access Level |
|---|---|---|
| `hotelops-admin` | HotelOps-Admins | Full EC2 + CloudWatch — manage everything |
| `hotelops-manager` | HotelOps-Managers | Read-only EC2 + CloudWatch — view only |
| `hotelops-staff` | HotelOps-Staff | CloudWatch read only — monitoring only |

---

## Automation Scripts

**Health Monitoring** — `/home/ubuntu/hotelops_health.sh` — runs every hour

```bash
#!/bin/bash
echo "=== HotelOps Health Report ===" >> /home/ubuntu/health.log
echo "Timestamp : $(date)" >> /home/ubuntu/health.log
echo "CPU       : $(top -bn1 | grep 'Cpu(s)' | awk '{print $2}')%" >> /home/ubuntu/health.log
echo "Memory    : $(free -m | awk 'NR==2{printf "%s/%s MB", $3,$2}')" >> /home/ubuntu/health.log
echo "Disk      : $(df -h / | awk 'NR==2{print $5}') used" >> /home/ubuntu/health.log
echo "Nginx     : $(systemctl is-active nginx)" >> /home/ubuntu/health.log
echo "MySQL     : $(systemctl is-active mysql)" >> /home/ubuntu/health.log
echo "Docker    : $(systemctl is-active docker)" >> /home/ubuntu/health.log
echo "---" >> /home/ubuntu/health.log
```

**Database Backup** — `/home/ubuntu/hotelops_backup.sh` — runs every midnight

```bash
#!/bin/bash
DATE=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="/opt/hotelops/backups"
mysqldump -u hotelops_user -pHotelOps@2024 hotelops_db \
  > $BACKUP_DIR/hotelops_backup_$DATE.sql
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
echo "Backup completed: $DATE" >> /home/ubuntu/backup.log
```

**Cron Schedule**

```
# Health check — every hour at minute 0
0 * * * *  /home/ubuntu/hotelops_health.sh

# Database backup — every day at midnight
0 0 * * *  /home/ubuntu/hotelops_backup.sh
```

---

## Cost Analysis

| Service | Usage | Free Tier Cost | Production Cost |
|---|---|---|---|
| EC2 t3.micro | 750 hrs/month | USD 0.00 | USD 7.59/mo |
| EBS 8 GB gp3 | Block storage | USD 0.00 | USD 0.64/mo |
| Amazon VPC | 1 VPC + subnets | USD 0.00 | USD 0.00/mo |
| CloudWatch | 5 metrics + 1 alarm | USD 0.00 | USD 3.00/mo |
| Data Transfer | < 1 GB outbound | USD 0.00 | USD 0.90/mo |
| IAM | 3 users + 2 groups | USD 0.00 | USD 0.00/mo |
| **Total** | | **USD 0.00** | **USD 12.13/mo** |

> Free Tier confirmed via AWS Billing Dashboard — June 2026  
> Production estimate with RDS + Load Balancer — approximately USD 80/month  
> Reserved Instances (1-year) reduce production cost by up to 72%

---

## Infrastructure Reference

```
Instance Name   HotelOps-WebServer
Instance ID     i-02725eee03f1c0b35
Instance Type   t3.micro
AMI             Ubuntu 26.04 LTS (ami-06259b63260eddc13)
Public IP       32.236.113.239
Private IP      10.0.1.23
VPC             HotelOps-VPC  (10.0.0.0/16)
Public Subnet   Public-Subnet  (10.0.1.0/24)
Private Subnet  Private-Subnet (10.0.2.0/24)
Security Group  HotelOps-Web-SG
Key Pair        HotelOps-Key (RSA .pem)
Region          ap-southeast-2 (Sydney, Australia)
Storage         8 GB gp3 EBS
Monthly Cost    USD 0.00 (Free Tier)
```

---

## Quick Deploy

```bash
# Connect
ssh -i HotelOps-Key.pem ubuntu@32.236.113.239

# Setup
sudo apt update && sudo apt upgrade -y

# Nginx
sudo apt install nginx -y && sudo systemctl enable nginx
sudo cp index.html /var/www/html/index.html

# MySQL
sudo apt install mysql-server -y
sudo mysql < database/schema.sql

# Docker
sudo apt install docker.io -y
docker run -d -p 8080:80 --name hotelops-container nginx

# Automation
chmod +x scripts/*.sh
crontab -e
```

---

## Project Structure

```
hotelops-aws/
├── web/
│   └── index.html                 HotelOps management dashboard
├── scripts/
│   ├── hotelops_health.sh         Hourly server health monitoring
│   └── hotelops_backup.sh         Daily MySQL backup automation
├── database/
│   └── schema.sql                 MySQL schema and sample data
├── logs/
│   ├── health.log                 Auto-generated by health script
│   └── backup.log                 Auto-generated by backup script
└── README.md
```

---

## Documentation

| File | Description |
|---|---|
| `Project_Report.docx` | Full 20-step implementation with objectives, outcomes, and screenshots |
| `Viva_Notes.docx` | Study guide with 50+ Q&A across 10 topics — VPC, EC2, Linux, Nginx, MySQL, Docker, CloudWatch, Automation, IAM, Cost |
| `README.md` | This file |

---

<div align="center">

**HotelOps Hospitality Management Cloud**

B.Tech CSE 2024–2028 · Semester IV AWS Case Study · ITM Skills University

Built on Amazon Web Services · ap-southeast-2 Sydney · Ubuntu 26.04 · Nginx · MySQL · Docker

</div>

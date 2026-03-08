# Azure Book Review App — 3-Tier Architecture

A full-stack Book Review application deployed on Microsoft Azure using a 3-tier architecture.

## Live Demo
![App Running in Browser](screenshots/book-review-app-running.png)

## Architecture
```
Browser → Public Load Balancer (port 80)
  → Nginx on web-vm (web-subnet)
    → Next.js frontend (port 3000)
    → /api/* → Internal Load Balancer (10.0.3.10:3001)
      → Node.js/Express backend (app-subnet)
        → Azure MySQL Flexible Server (db-subnet)
```

## Tech Stack
- **Frontend:** Next.js 15, React, Tailwind CSS
- **Backend:** Node.js, Express.js, Sequelize ORM
- **Database:** Azure Database for MySQL Flexible Server
- **Infrastructure:** Azure VNet, NSGs, Public & Internal Load Balancers, Ubuntu 24.04 VMs

---

## Task 1 — VNet and Subnet Setup

### Resource Group
![Resource Group](screenshots/01_resource_group.png)

### VNet Overview
![VNet Overview](screenshots/02_vnet_overview.png)

### Subnets List
![Subnets List](screenshots/03_vnet_subnets_list.png)

---

## Task 2 — NSGs and Load Balancers

### Web NSG Inbound Rules
![Web NSG](screenshots/04_web_nsg_inbound_rules.png)

### App NSG Inbound Rules
![App NSG](screenshots/05_app_nsg_inbound_rules.png)

### DB NSG Inbound Rules
![DB NSG](screenshots/06_db_nsg_inbound_rules.png)

### Public Load Balancer Overview
![Public LB Overview](screenshots/07_public_lb_overview.png)

### Public LB Frontend IP
![Public LB Frontend IP](screenshots/08_public_lb_frontend_ip.png)

### Public LB Backend Pool
![Public LB Backend Pool](screenshots/09_public_lb_backend_pool.png)

### Public LB Load Balancing Rule
![Public LB Rule](screenshots/11_public_lb_lb_rule.png)

### Internal Load Balancer Overview
![Internal LB Overview](screenshots/12_internal_lb_overview.png)

### Internal LB Frontend IP
![Internal LB Frontend IP](screenshots/13_internal_lb_frontend_ip.png)

### Internal LB Backend Pool
![Internal LB Backend Pool](screenshots/14_internal_lb_backend_pool.png)

### Internal LB Health Probe
![Internal LB Health Probe](screenshots/15_internal_lb_health_probe.png)

### Internal LB Load Balancing Rule
![Internal LB Rule](screenshots/16_internal_lb_lb_rule.png)

---

## Task 3 — VM and Application Deployment

### Web VM Overview
![Web VM Overview](screenshots/17_web_vm_overview.png)

### Web VM Networking Settings
![Web VM Networking](screenshots/18_web_vm_networking_settings.png)

### Web VM in Backend Pool
![Web VM Backend Pool](screenshots/19_web_vm_in_backend_pool.png)

### Web VM PM2 Running
![Web VM PM2](screenshots/20_web_vm_pm2_running.png)

### Web VM Nginx Status
![Web VM Nginx](screenshots/21_web_vm_nginx_status.png)

### Web VM Curl Localhost
![Web VM Curl](screenshots/22_web_vm_curl_localhost.png)

### App VM Overview
![App VM Overview](screenshots/23_app_vm_overview.png)

### App VM Networking Settings
![App VM Networking](screenshots/24_app_vm_networking_settings.png)

### App VM in Backend Pool
![App VM Backend Pool](screenshots/25_app_vm_in_backend_pool..png)

### App VM PM2 Running
![App VM PM2](screenshots/26_app_vm_pm2_running.png)

### App VM Curl Localhost 3001
![App VM Curl](screenshots/27_app_vm_curl_localhost_3001.png)

### App Running in Browser
![App in Browser](screenshots/28_app_running_in_browser.png)

---

## Task 4 — Azure MySQL Setup

### MySQL Server Overview
![MySQL Overview](screenshots/29_mysql_server_overview.png)

### MySQL Server Networking
![MySQL Networking](screenshots/30_mysql_server_networking.png)

### MySQL Private DNS Zone
![MySQL DNS](screenshots/32_mysql_private_dns_zone.png)

### MySQL Connected with SSL Logs
![MySQL SSL Logs](screenshots/33_mysql_db_connected_ssl_logs.png)

### MySQL Schema Tables Created
![MySQL Tables](screenshots/34_mysql_schema_tables_created.png)

---

## Azure Resources
- Resource Group: book-review-rg (Canada Central)
- VNet: 10.0.0.0/16 with 3 subnets
  - web-subnet: 10.0.1.0/24
  - app-subnet: 10.0.3.0/24
  - db-subnet: 10.0.5.0/24
- Public LB: routes port 80 to web-vm
- Internal LB: routes port 3001 to app-vm
- MySQL: Private VNet integration, SSL enforced, Zone-Redundant HA

---

## Setup

### Frontend (web-vm)
```bash
cd frontend
npm install
cp .env.local.example .env.local
# Edit .env.local with your Public LB IP
export NODE_OPTIONS="--max-old-space-size=1024"
npm run build
pm2 start npm --name 'frontend' -- start -- -p 3000
pm2 startup && pm2 save
```

### Backend (app-vm)
```bash
cd backend
npm install
cp .env.example .env
# Edit .env with your Azure MySQL credentials
pm2 start src/server.js --name 'backend'
pm2 startup && pm2 save
```

### Nginx (web-vm)
```bash
sudo cp nginx/default.conf /etc/nginx/sites-available/default
sudo nginx -t
sudo systemctl reload nginx
```

---

## Key Pitfalls Discovered During Deployment
1. NSG source IP requires /32 suffix
2. VM creation — set NIC NSG to None, subnet NSG handles it
3. Private IP must be set to Static before adding public IP
4. Do not change Static IP and add public IP in same save operation
5. Backend pool becomes empty after VM restart — always verify
6. Next.js 15 builds require 4 GiB RAM minimum — resize VM before building
7. NEXT_PUBLIC_API_URL must be public IP not private VNet IP
8. Nginx must proxy /api/* to backend — browser cannot reach private IPs
9. ALLOWED_ORIGINS must match exactly how browser sends origin header
10. Duplicate email causes 500 error on registration — use different email
11. MySQL client must be installed separately with apt install
12. ISP IP changes frequently — update NSG SSH rules when SSH stalls
13. Use -p flag for mysql password prompt to avoid history exposure
14. pm2 save must be run after pm2 startup or apps won't start on reboot

---

## Security
- App tier not publicly accessible
- DB tier only accessible from app-subnet on port 3306
- SSH restricted to admin IP only via NSG
- MySQL SSL enforced on all connections
- No credentials committed to source code
- Nginx reverse proxy keeps backend IP private

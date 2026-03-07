# Azure Book Review App — 3-Tier Architecture

A full-stack Book Review application deployed on Microsoft Azure using a 3-tier architecture.

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

## Azure Resources
- Resource Group: book-review-rg (Canada Central)
- VNet: 10.0.0.0/16 with 3 subnets
  - web-subnet: 10.0.1.0/24
  - app-subnet: 10.0.3.0/24
  - db-subnet: 10.0.5.0/24
- Public LB: routes port 80 to web-vm
- Internal LB: routes port 3001 to app-vm
- MySQL: Private VNet integration, SSL enforced, Zone-Redundant HA

## Key Pitfalls Discovered During Deployment
1. NSG source IP requires /32 suffix
2. VM creation — set NIC NSG to None, subnet NSG handles it
3. Private IP must be set to Static before adding public IP
4. Do not change Static IP and add public IP in same save operation
5. Backend pool becomes empty after VM restart — always verify
6. Next.js 15 builds require 4 GiB RAM minimum
7. NEXT_PUBLIC_API_URL must be public IP not private VNet IP
8. Nginx must proxy /api/* to backend — browser cannot reach private IPs
9. ALLOWED_ORIGINS must match exactly how browser sends origin header
10. Duplicate email causes 500 error on registration

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

## Security
- App tier not publicly accessible
- DB tier only accessible from app-subnet on port 3306
- SSH restricted to admin IP only via NSG
- MySQL SSL enforced on all connections
- No credentials committed to source code
- Nginx reverse proxy keeps backend IP private

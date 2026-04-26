# Turf Booking Platform — Infrastructure

This repository contains the deployment configuration for the Turf Booking Platform microservices architecture.

## Repository Structure

```
├── docker-compose.yml       → Development compose (builds from local source)
├── docker-compose.prod.yml  → Production compose (uses Docker Hub images)
├── .env.example             → Template for required environment variables
├── ansible/
│   ├── ansible.cfg
│   ├── inventory/hosts.ini
│   └── playbooks/
│       ├── deploy.yml       → Deploy the application stack
│       └── setup-server.yml → One-time server provisioning
├── Jenkinsfile              → CI/CD for infrastructure changes
└── README.md
```

## Services Deployed

| Service | Image | Port |
|---------|-------|------|
| MongoDB | `mongo:latest` | 27017 |
| Redis | `redis:alpine` | 6380 → 6379 |
| API Gateway | `prateekkumaryadav/turf-api-gateway` | 5000 |
| Auth Service | `prateekkumaryadav/turf-auth-service` | 5001 |
| Turf Service | `prateekkumaryadav/turf-turf-service` | 5002 |
| Booking Service | `prateekkumaryadav/turf-booking-service` | 5003 |
| Frontend | `prateekkumaryadav/turf-frontend` | 80, 5173 |

## Quick Start (Development)

```bash
# Copy and configure environment variables
cp .env.example .env
# Edit .env with your values

# Start all services (builds from local source)
docker compose up -d

# Seed the database
cd ../Turf-Booking-Platform-Backend
MONGO_URI=mongodb://localhost:27017/turf_booking node seed.js
```

## Production Deployment

```bash
# One-time server setup
ansible-playbook ansible/playbooks/setup-server.yml

# Deploy
ansible-playbook ansible/playbooks/deploy.yml
```

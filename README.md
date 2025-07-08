# Attendance Backend - Employee Attendance Tracking System

A comprehensive backend system for tracking employee attendance with features for check-in/check-out via QR codes or phone, user management, departments, positions, statistics, real-time dashboard monitoring, and office geolocation binding.

## Features

- **Attendance Tracking**: Check-in/check-out via QR code or phone
- **User Management**: Complete CRUD operations for employees
- **Departments & Positions**: Organizational structure management
- **Statistics & Reports**: Comprehensive attendance analytics
- **Real-time Dashboard**: Live monitoring with Server-Sent Events (SSE)
- **Geolocation**: Office location binding for attendance validation
- **Authentication**: JWT tokens with Google OAuth integration
- **QR Code Generation**: Individual QR codes for each employee
- **Excel Import/Export**: Bulk employee data operations
- **Multi-language Support**: Configurable language settings

## Technology Stack

- **Go 1.23** - Main programming language
- **Gin** - Web framework
- **PostgreSQL** - Primary database
- **Redis** - Caching layer
- **Bun ORM** - Database operations
- **JWT** - Authentication
- **Swagger** - API documentation
- **Docker** - Containerization
- **Google OAuth** - Third-party authentication

## Project Structure

```
├── cmd/main.go              # Application entry point
├── foundation/web/          # Web framework wrappers
├── internal/
│   ├── auth/               # JWT authentication
│   ├── commands/           # Database migrations
│   ├── controller/http/v1/ # HTTP controllers
│   ├── entity/            # Data models
│   ├── middleware/        # Middleware functions
│   ├── repository/postgres/ # Database repositories
│   ├── router/            # Routing configuration
│   └── service/           # Business logic
├── docs/                  # Swagger documentation
├── scripts/               # Deployment scripts
├── config.yaml           # Configuration file
├── docker-compose.yml    # Docker composition
└── Dockerfile           # Docker image configuration
```

## Prerequisites

Before running the project locally, ensure you have:

- **Docker & Docker Compose** installed
- **PostgreSQL** installed
- **Go 1.23+** (for local development)
- **Git** for version control

### Installing Docker (Ubuntu/Debian)

```bash
sudo apt update
sudo apt upgrade
sudo apt install docker.io docker-compose -y
```

### Installing PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

## Local Development Setup

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd attendance_backend
```

### 2. Database Setup

Create PostgreSQL user and database:

```bash
sudo -i -u postgres
psql
CREATE USER your_user WITH PASSWORD 'your_password';
CREATE DATABASE attendances;
GRANT ALL PRIVILEGES ON DATABASE attendances TO your_user;
\q
exit
```

### 3. Configuration

Edit the `config.yaml` file with your settings:

```yaml
db_username: "your_user"
db_password: "your_password"
db_host: "localhost"
db_name: "attendances"
default_lang: "uz"  # or "jp", "en", etc.
port: "5432"
disable_tls: true
base_url: "http://localhost:8080/api/v1"
jwt_key: "attendancePanel"
google_client_id: "your_google_client_id"
google_client_secret: "your_google_client_secret"
google_redirect_url: "your_redirect_url"
frontend_url: "your_frontend_url"
```

### 4. Run the Application

```bash
# Using Makefile
make run

# Or directly with Go
go run cmd/main.go
```

The server will start on `http://localhost:8080`

## Docker Deployment

### Local Docker Setup

```bash
docker-compose up -d
```

# Complete AWS Deployment Guide - Backend & Frontend

This guide covers the complete deployment process for both backend and frontend applications on AWS EC2, designed for backend developers who also handle DevOps responsibilities.

## 1. AWS EC2 Instance Setup

### Launch Instance
1. Log in to your AWS Console
2. Navigate to EC2 Dashboard → Launch Instance
3. Select **Ubuntu 20.04 LTS** as the AMI
4. Choose instance type (t2.micro for testing, t3.medium+ for production)
5. Configure storage (minimum 20GB recommended)

### Security Group Configuration
Configure inbound rules to allow:
- **Port 22** (SSH) - Your IP only
- **Port 80** (HTTP) - 0.0.0.0/0
- **Port 443** (HTTPS) - 0.0.0.0/0
- **Port 5432** (PostgreSQL) - Your IP only
- **Port 8080** (Backend API) - 127.0.0.1 only (internal)
- **Port 3000** (Frontend) - 127.0.0.1 only (internal)

### Connect to Instance
```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

## 2. Server Preparation

### Update System Packages
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Docker and Docker Compose
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker ubuntu

# Install Docker Compose
sudo apt install docker-compose -y

# Verify installation
docker --version
docker-compose --version
```

### Install Nginx
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Install SSL Certificate Tool
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Install Additional Tools
```bash
# Install Git for cloning repositories
sudo apt install git -y

# Install Node.js and npm (if needed for frontend builds)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## 3. Domain and DNS Configuration

### Set Up DNS Records
Before proceeding, ensure you have:
- **A record** for your API domain pointing to your EC2 public IP
- **A record** for your frontend domain pointing to your EC2 public IP

Example:
- `api.yourdomain.com` → `your-ec2-public-ip`
- `app.yourdomain.com` → `your-ec2-public-ip`

## 4. Nginx Configuration

### Backend Proxy Configuration
```bash
sudo nano /etc/nginx/sites-available/backend.conf
```

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
        proxy_buffering off;
    }
}
```

### Frontend Proxy Configuration
```bash
sudo nano /etc/nginx/sites-available/frontend.conf
```

```nginx
server {
    listen 80;
    server_name app.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
        proxy_buffering off;
        
        # WebSocket support (if your frontend uses WebSockets)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Enable Sites and Test Configuration
```bash
# Enable the sites
sudo ln -s /etc/nginx/sites-available/backend.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload nginx
sudo systemctl reload nginx
```

## 5. SSL Certificate Setup

### Backend SSL
```bash
sudo certbot --nginx -d api.yourdomain.com
```

### Frontend SSL
```bash
sudo certbot --nginx -d app.yourdomain.com
```

### Auto-renewal Setup
```bash
# Test auto-renewal
sudo certbot renew --dry-run

# Certbot automatically sets up a cron job, but you can verify:
sudo crontab -l
```

## 6. Backend Deployment

### Clone and Setup Backend
```bash
# Clone your repository
git clone https://github.com/yourusername/your-backend-repo.git
cd your-backend-repo

# Copy and configure environment file
cp .env.example .env
sudo nano .env
```

### Configure Environment Variables
Update your `.env` file with production values:
```env
NODE_ENV=production
PORT=8080
DATABASE_URL=postgresql://username:password@localhost:5432/dbname
JWT_SECRET=your-super-secret-jwt-key
API_URL=https://api.yourdomain.com
FRONTEND_URL=https://app.yourdomain.com
```

### Build and Deploy Backend
```bash
# Build and start services
sudo docker-compose build
sudo docker-compose up -d

# Check if services are running
sudo docker-compose ps
sudo docker-compose logs
```

## 7. Frontend Deployment

### Method 1: Local Build and Push to Docker Hub

#### On Your Local Machine:
```bash
# Navigate to frontend project
cd your-frontend-project

# Update environment variables for production
# Create or update .env.production
echo "REACT_APP_API_URL=https://api.yourdomain.com" > .env.production
echo "REACT_APP_ENVIRONMENT=production" >> .env.production

# Build Docker image
docker build -t yourusername/your-frontend:latest .

# Push to Docker Hub
docker login
docker push yourusername/your-frontend:latest
```

#### On Your EC2 Server:
```bash
# Pull the latest image
docker pull yourusername/your-frontend:latest

# Stop existing container (if any)
docker stop frontend-app || true
docker rm frontend-app || true

# Run new container
docker run -d \
  --name frontend-app \
  -p 3000:80 \
  --restart unless-stopped \
  yourusername/your-frontend:latest

# Verify container is running
docker ps
docker logs frontend-app
```

### Method 2: Direct Build on Server

```bash
# Clone frontend repository
git clone https://github.com/yourusername/your-frontend-repo.git
cd your-frontend-repo

# Install dependencies and build
npm install
npm run build

# Create Dockerfile for production (if not exists)
cat > Dockerfile << EOF
FROM nginx:alpine
COPY build/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Create nginx configuration for the container
cat > nginx.conf << EOF
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files \$uri \$uri/ /index.html;
    }
}
EOF

# Build and run
docker build -t frontend-app .
docker run -d --name frontend-app -p 3000:80 --restart unless-stopped frontend-app
```

## 8. Database Setup (if using PostgreSQL)

### Install PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Configure Database (if occured some troubleshots)
```bash
# Switch to postgres user
sudo -u postgres psql

# Create database and user
CREATE DATABASE your_database_name;
CREATE USER your_username WITH ENCRYPTED PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE your_database_name TO your_username;
\q
```

## 9. Monitoring and Maintenance

### Check Application Status
```bash
# Check Docker containers
sudo docker ps -a

# Check nginx status
sudo systemctl status nginx

# Check backend logs (attendance_backend)
sudo docker logs attendance_backend

# Check frontend logs
sudo docker logs attendance_frontend

# Check nginx error logs
sudo tail -f /var/log/nginx/error.log
```

### Backend Container Management
```bash
# View backend container status
sudo docker-compose ps

# Stop backend services
sudo docker-compose down

# Restart backend services
sudo docker-compose up -d

# View backend logs in real-time
sudo docker-compose logs -f
```

### Frontend Container Management
```bash
# Check frontend container
sudo docker ps | grep attendance_frontend

# Stop frontend container
sudo docker stop attendance_frontend

# Remove frontend container
sudo docker rm attendance_frontend

# Start new frontend container
sudo docker run -d -p 3000:80 --name attendance_frontend --restart unless-stopped yourusername/attendance_frontend:latest
```

### Update Applications

#### Backend Updates
```bash
cd ~/attendance_backend
git pull origin main
sudo docker-compose down
sudo docker-compose build
sudo docker-compose up -d
```

#### Frontend Updates
```bash
# Pull latest frontend image
sudo docker pull yourusername/attendance_frontend:latest

# Stop and remove old container
sudo docker stop attendance_frontend
sudo docker rm attendance_frontend

# Remove old image (optional)
sudo docker rmi $(sudo docker images yourusername/attendance_frontend -q | tail -n +2)

# Start new container
sudo docker run -d -p 3000:80 --name attendance_frontend --restart unless-stopped yourusername/attendance_frontend:latest
```

## Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google+ API
4. Go to "Credentials" → "Create Credentials" → "OAuth 2.0 Client ID"
5. Configure OAuth consent screen
6. Add authorized redirect URIs:
   - `http://localhost:8080/api/v1/auth/google/callback` (development)
   - `https://your-api-domain.com/api/v1/auth/google/callback` (production)
7. Add authorized JavaScript origins:
   - `http://localhost:3000` (development)
   - `https://your-frontend-domain.com` (production)
8. Copy Client ID and Client Secret to your `config.yaml`

## API Documentation

### Authentication

- `POST /api/v1/sign-in`  
  User login.  
  Access: Open

- `POST /api/v1/refresh-token`  
  JWT token refresh.  
  Access: Open

- `POST /api/v1/logout`  
  User logout (token invalidation).  
  Access: Open

- `GET /api/v1/auth/google`  
  Redirect to Google OAuth.  
  Access: Open

- `GET /api/v1/auth/google/callback`  
  Callback for Google OAuth.  
  Access: Open

- `GET /api/v1/auth/google/force-select`  
  Google OAuth with forced account selection.  
  Access: Open

### Files

- `GET /media/*filepath`  
  Get a file (e.g. images).  
  Access: Open

- `HEAD /media/*filepath`  
  Check for file existence.  
  Access: Open

### Users

- `GET /api/v1/user/list`  
  List of users.  
  Access: Only for Admin

- `GET /api/v1/user/:id`  
  Detailed information about the user by ID.  
  Access: Only for Admin

- `GET /api/v1/user/qrcodehimself`  
  Get a QR code for yourself (by your EmployeeID).  
  Access: Only for Employee

- `GET /api/v1/user/qrcode`  
  Get a QR code by EmployeeID (any user).  
  Access: Only for Admin

- `GET /api/v1/user/qrcodelist`  
  Get a list of QR codes for all employees.  
  Access: Only for Admin

- `GET /api/v1/user/export_employee`  
  Export employees to Excel.  
  Access: Admin Only

- `GET /api/v1/user/export_template`  
  Download template for importing employees.  
  Access: Admin Only

- `POST /api/v1/user/create`  
  Create a user.  
  Access: Admin Only  
  Validation: Email, phone, Latin

- `POST /api/v1/user/create_excell`  
  Bulk create users from Excel.  
  Access: Admin Only

- `PATCH /api/v1/user/:id`  
  Update individual user fields.  
  Access: Admin Only  
  Validation: Email, phone, Latin

- `DELETE /api/v1/user/:id`  
  Delete a user.  
  Access: Admin Only

- `GET /api/v1/user/statistics`  
  Get user statistics.  
  Access: Any authorized

- `GET /api/v1/user/monthly`  
  Monthly user statistics.  
  Access: Any authorized

- `GET /api/v1/user/dashboard`  
  Employee dashboard.  
  Access: Any authorized

- `GET /api/v1/user/dashboardlist`  
  SSE (Server-Sent Events) for real-time dashboard.  
  Access: Open (but there may be a check inside the controller)

### Departments

- `GET /api/v1/department/list`  
  List of departments.  
  Access: Admin, Dashboard

- `GET /api/v1/department/:id`  
  Department details.  
  Access: Admin only

- `POST /api/v1/department/create`  
  Create a department.  
  Access: Admin only

- `PATCH /api/v1/department/:id`  
  Update a department (partially).  
  Access: Admin only

- `DELETE /api/v1/department/:id`  
  Delete a department.  
  Access: Admin Only

### Positions

- `GET /api/v1/position/list`  
  List of positions.  
  Access: Admin, Dashboard

- `GET /api/v1/position/:id`  
  Post details of a position.  
  Access: Admin Only

- `POST /api/v1/position/create`  
  Create a position.  
  Access: Admin Only

- `PUT /api/v1/position/:id`  
  Full update of a position.  
  Access: Admin Only

- `PATCH /api/v1/position/:id`  
  Partial update of a position.  
  Access: Admin Only

- `DELETE /api/v1/position/:id`  
  Delete a position.  
  Access: Admin Only

### Company Info

- `GET /api/v1/company_info/list`  
  Get information about a company.  
  Access: Admin Only

- `PUT /api/v1/company_info/:id`  
  Update all company information.  
  Access: Admin Only  
  Validation: Latin

### Attendance

- `GET /api/v1/attendance/list`  
  Attendance list.  
  Access: Admin, Employee, Dashboard

- `GET /api/v1/attendance/:id`  
  Attendance details by ID.  
  Access: Admin Only

- `GET /api/v1/attendance/history`  
  Attendance history by ID.  
  Access: Admin Only

- `POST /api/v1/attendance/createbyphone`  
  Check in by phone.  
  Access: Any authorized

- `POST /api/v1/attendance/createbyqrcode`  
  Check in by QR code.  
  Access: Any authorized

- `PATCH /api/v1/attendance/exitbyphone`  
  Mark exit by phone (check-out).  
  Access: Any authorized

- `PUT /api/v1/attendance/:id`  
  Full update of attendance record.  
  Access: Only for Admin  
  Validation: Latin

- `PATCH /api/v1/attendance/:id`  
  Partial update of attendance record.  
  Access: Only for Admin  
  Validation: Latin

- `DELETE /api/v1/attendance/:id`  
  Delete attendance record.  
  Access: Only for Admin

- `GET /api/v1/attendance`  
  General attendance statistics.  
  Access: Only for Admin

- `GET /api/v1/attendance/piechart`  
  Statistics for pie chart.  
  Access: Admin Only

- `GET /api/v1/attendance/barchart`  
  Bar chart statistics.  
  Access: Admin Only

- `GET /api/v1/attendance/graph`  
  Attendance graph.  
  Access: Admin

## Database Schema

### Main Tables

- **users** - Employee information
- **attendance** - Attendance records
- **attendance_period** - Work periods
- **department** - Organizational departments
- **position** - Job positions
- **company_info** - Company settings

### Migrations

Database migrations are handled automatically on application startup via `migrate.go`.

## Security & Authentication

### JWT Authentication
- RSA signed JWT tokens
- Role-based access control
- Available roles: EMPLOYEE, ADMIN, DASHBOARD, QRCODE

### Middleware
- Authentication middleware for protected routes
- Role-based authorization middleware
- CORS handling

## Container Management

### Check Running Containers

```bash
sudo docker ps -a
```

### Backend Updates

```bash
sudo docker-compose down
git pull origin main
sudo docker-compose build
sudo docker-compose up -d
```

### Frontend Updates (if applicable)

```bash
# Locally with correct environment variables
docker compose down
docker compose build

# Tag and push to Docker Hub
docker tag attendance_frontend-react-attendance-app:latest your-dockerhub-username/attendance_frontend:latest
docker push your-dockerhub-username/attendance_frontend:latest

# On server
sudo docker ps -a
sudo docker stop attendance_frontend
sudo docker rm attendance_frontend

# Remove old image
sudo docker images
sudo docker rmi <image_id>

# Pull and run new version
sudo docker pull your-dockerhub-username/attendance_frontend:latest
sudo docker run -d -p 3000:80 --name attendance_frontend your-dockerhub-username/attendance_frontend:latest
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Development Notes

- The application uses Server-Sent Events (SSE) for real-time dashboard updates
- Geolocation validation ensures employees check-in from the correct office location
- Each employee gets a unique QR code for quick check-in/check-out
- Excel import/export functionality supports bulk employee operations
- Multi-language support is configured via the `default_lang` setting

## Troubleshooting

### Common Issues

1. **Database connection failed**: Check PostgreSQL service and credentials
2. **Port already in use**: Ensure no other services are using port 8080
3. **Docker build fails**: Check Docker daemon status and available disk space
4. **Google OAuth not working**: Verify client ID, secret, and redirect URLs

### Logs

View application logs:

```bash
# Docker logs
sudo docker logs attendance_backend

# Direct logs (if running locally)
tail -f logs/app.log
```
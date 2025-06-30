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

## 🛠 Technology Stack

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

### Production Deployment

For production deployment, update your code and restart containers:

```bash
# Stop containers
sudo docker-compose down

# Pull latest changes
git pull origin main

# Rebuild and start
sudo docker-compose build
sudo docker-compose up -d
```

## AWS Deployment Guide

### 1. EC2 Instance Setup

1. Launch Ubuntu 20.04 EC2 instance
2. Configure Security Groups:
   - Port 5432 (PostgreSQL)
   - Port 8080 (Backend API)
   - Port 3000 (Frontend)
   - Port 80, 443 (HTTP/HTTPS)
3. Connect via SSH:

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

### 2. Server Preparation

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker-compose -y

# Install Nginx
sudo apt install nginx -y

# Install Certbot for SSL
sudo apt install certbot python3-certbot-nginx -y
```

### 3. Nginx Configuration

Create backend configuration:

```bash
sudo nano /etc/nginx/sites-enabled/backend.conf
```

```nginx
server {
    server_name your-api-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
    }
}
```

### 4. Enable SSL

```bash
sudo certbot --nginx -d your-api-domain.com
```

### 5. Deploy Application

```bash
cd attendance_backend
sudo docker-compose build
sudo docker-compose up -d
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

### Authentication Endpoints

- `POST /api/v1/sign-in` - User login
- `POST /api/v1/refresh-token` - Refresh JWT token
- `GET /api/v1/auth/google` - Google OAuth login

### User Management

- `GET /api/v1/user/list` - Get users list
- `POST /api/v1/user/create` - Create new user
- `GET /api/v1/user/statistics` - Get user statistics
- `GET /api/v1/user/qrcode` - Generate user QR code

### Attendance

- `GET /api/v1/attendance/list` - Get attendance records
- `POST /api/v1/attendance/createbyphone` - Check-in via phone
- `POST /api/v1/attendance/createbyqrcode` - Check-in via QR code
- `PATCH /api/v1/attendance/exitbyphone` - Check-out via phone

### Departments & Positions

- `GET /api/v1/department/list` - Get departments
- `POST /api/v1/department/create` - Create department
- `GET /api/v1/position/list` - Get positions

### Real-time Dashboard

- `GET /api/v1/user/dashboardlist` - SSE stream for dashboard

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
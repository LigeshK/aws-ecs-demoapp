# ECS Workshop Application

A fully containerized full-stack web application designed to demonstrate deployment on AWS Elastic Container Service (ECS). It features a React-based frontend and a Node.js backend, showcasing modern containerization techniques and an automated CI/CD pipeline for seamless deployment to ECS.

## 🏗️ Architecture Overview

This application consists of two main components:

- **Frontend**: Static HTML/CSS/JavaScript served by Nginx with API proxy configuration
- **Backend**: Node.js/Express REST API with health check endpoints
- **CI/CD**: GitHub Actions workflow for automated Docker image building and ECS deployment

```
aws-ecs-demoapp/
├── frontend/
│   ├── index.html          # Main application page
│   ├── style.css           # Application styling
│   ├── script.js           # Frontend JavaScript logic
│   ├── nginx.conf          # Nginx proxy configuration
│   ├── Dockerfile          # Frontend container definition
│   └── .dockerignore       # Docker build exclusions
├── backend/
│   ├── index.js            # Express server with API endpoints
│   ├── package.json        # Node.js dependencies
│   ├── Dockerfile          # Backend container definition
│   └── .dockerignore       # Docker build exclusions
└── .github/
    └── workflows/
        └── deploy.yml      # CI/CD pipeline configuration
```

## 🚀 Application Components

### Frontend Service

- **Technology**: Nginx serving static assets
- **Port**: 80 (containerized), 8080 (local testing)
- **Features**:
  - Responsive web interface
  - API proxy to backend service
  - ECS service discovery integration

### Backend Service

- **Technology**: Node.js with Express framework
- **Port**: 3000
- **Endpoints**:
  - `GET /api/health` - Health check endpoint returning service status
- **Features**:
  - CORS enabled for cross-origin requests
  - JSON API responses
  - Container health monitoring

### CI/CD Pipeline

- **Trigger**: Push to main branch
- **Actions**:
  - Build Docker images for frontend and backend
  - Push images to Amazon ECR
  - Update ECS services with new deployments
- **Registry**: Single ECR repository with tagged images (`frontend-latest`, `backend-latest`)

## 📋 Prerequisites

### Local Development Environment

- **Docker**: Container runtime and image building
- **Node.js v18+**: Backend development and testing
- **AWS CLI**: AWS service interaction and authentication
- **Git**: Version control and repository management

### AWS Infrastructure Requirements

- ECS Cluster with network and application stacks deployed
- ECR repository (`my-app-repo`) for container images
- ECS services (`frontend-service`, `backend-service`) configured
- Proper IAM roles and security groups

### GitHub Configuration

Required repository secrets:

- `AWS_ACCOUNT_ID`: Your AWS account identifier
- `AWS_ACCESS_KEY_ID`: AWS access credentials
- `AWS_SECRET_ACCESS_KEY`: AWS secret credentials

### Local Testing - Docker

1. **Build and run frontend container**

   ```bash
   docker build -t test-frontend .
   docker run -d -p 8080:80 --name frontend test-frontend
   ```

2. **Test frontend (backend will show "Error" status initially)**

   ```bash
   curl http://localhost:8080
   # Or visit http://localhost:8080 in your browser
   ```

3. **Build and run backend container**

   ```bash
   cd ../backend
   docker build -t test-backend .
   docker run -d -p 3000:3000 --name backend test-backend
   ```

4. **Verify backend health endpoint**

   ```bash
   curl http://localhost:3000/api/health
   # Should return: {"status":"ok"}
   ```

5. **Clean up after testing**
   ```bash
   # Stop containers
   docker stop frontend backend
   docker rm frontend backend
   ```

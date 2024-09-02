# DevopsProject
### Docker deployment =>  React - API(DRF)-Database-Traefik to VPS server with SSL/https.





<p align="center"> <img src="https://github.com/DevRajib/3.DevopsProject/blob/main/Devops.png" align="center" height="250" </p>






# 3-Tier Docker Deployment Using GitHub Actions


# 3-Tier Docker Deployment with GitHub Actions


To set up a three-tier Docker deployment (React frontend, Django Rest Framework API, and a Database) using GitHub Actions, you'll want to automate the deployment process to build and push Docker images to a container registry, followed by deploying these containers to a server or cloud service. Here is a guide to achieve this:

# Overview of the setup:
### 1.Frontend: React.js
### 2.Backend: Django Rest Framework (DRF)
### 3.Database: PostgreSQL (or any preferred database)


# Steps Involved:

### 1.Dockerize the Application: Create Dockerfile for each tier (Frontend, API, and Database).
### 2.Compose Docker Setup: Use docker-compose to manage multi-container deployment.
### 3.Set up GitHub Actions: Define CI/CD pipelines in a GitHub Actions workflow file to automate building, pushing, and deploying the Docker images.


## Step 1: Project Structure

```bash
my_project/
│
├── frontend/           # React app
│   ├── public/
│   ├── src/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
├── backend/            # Django + DRF
│   ├── app/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── manage.py
├── db/                 # Database
│   └── Dockerfile
├── docker-compose.yml  # Docker Compose file
└── .github/
    └── workflows/
        └── ci.yml      # GitHub Actions workflow file
```

## Step 2: Dockerfiles for Each Service

### Frontend (React) Dockerfile

```Dockerfile
# frontend/Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package.json ./
COPY yarn.lock ./

RUN yarn install

COPY . ./

RUN yarn build

EXPOSE 3000

CMD ["yarn", "start"]
```

### Backend (Django + DRF) Dockerfile

```Dockerfile
# backend/Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app.wsgi:application"]
```

### Database Dockerfile (optional if using a prebuilt image like PostgreSQL)

```Dockerfile
# db/Dockerfile (Example using PostgreSQL)
FROM postgres:13-alpine
```

## Step 3: Docker Compose File

```yaml
# docker-compose.yml
version: '3'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://user:password@db:5432/mydatabase

  db:
    image: postgres:13-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Step 4: GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    services:
      db:
        image: postgres:13-alpine
        env:
          POSTGRES_USER: user
          POSTGRES_PASSWORD: password
          POSTGRES_DB: mydatabase
        ports:
          - 5432:5432

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Cache Docker layers
      uses: actions/cache@v2
      with:
        path: /tmp/.buildx-cache
        key: ${{ runner.os }}-buildx-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-buildx-

    - name: Build and push Docker images
      uses: docker/build-push-action@v2
      with:
        context: .
        push: false
        tags: username/myapp:latest

    - name: Deploy to server
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        HOST: ${{ secrets.HOST }}
      run: |
        ssh -o StrictHostKeyChecking=no user@$HOST "
          cd /path/to/project &&
          docker-compose down &&
          docker-compose up -d --build
        "
```
Explanation:
Docker Build and Push: The workflow builds Docker images for the frontend and backend and pushes them to Docker Hub.
Deploy: The deployment step connects to your server via SSH and pulls the latest Docker images, then restarts the services using docker-compose.
Secrets Configuration:
You need to configure the following secrets in your GitHub repository:

DOCKER_USERNAME: Your Docker Hub username.
DOCKER_PASSWORD: Your Docker Hub password.
SSH_USER: SSH user to connect to the server.
SERVER_IP: IP address of your server.
Step 4: Setting up the Server
Ensure Docker and docker-compose are installed on your server. The server should be set up to receive the Docker images and run them.

Additional Considerations:
Database Migrations: Automate running migrations after deployment.
Environment Variables: Use .env files or GitHub Secrets for securely passing environment variables (e.g., database credentials).
By following this guide, you will have a fully automated CI/CD pipeline that builds, pushes, and deploys your three-tier Docker application via GitHub Actions.
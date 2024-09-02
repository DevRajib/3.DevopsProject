# DevopsProject
### Docker deployment =>  React - API(DRF)-Database-Traefik to VPS server with SSL/https.





<p align="center"> <img src="https://github.com/DevRajib/3.DevopsProject/blob/main/Devops.png" align="center" height="250" </p>






# 3-Tier Docker Deployment Using GitHub Actions


# 3-Tier Docker Deployment with GitHub Actions

This guide will help you set up a 3-tier Docker deployment (React app, Django REST Framework (DRF) API, and a database) using GitHub Actions.

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

## GitHub Secrets

Add the following secrets to your GitHub repository:

- `SSH_PRIVATE_KEY`: The private key to access your remote server.
- `HOST`: The hostname or IP address of your remote server.


# Step by Step instruction coming soon

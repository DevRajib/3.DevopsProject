
# 3-Tier Docker Deployment Using GitHub Actions

This guide outlines a basic setup for a 3-tier Docker deployment using GitHub Actions. The tiers include:
1. **Frontend**: React application
2. **Backend**: API (e.g., Node.js)
3. **Database**: PostgreSQL

The GitHub Actions workflow will automate the build, push, and deployment of your Docker containers to a server.

## Step 1: Prepare Dockerfiles

### Frontend (React)

Create a `Dockerfile` in your React project:

```Dockerfile
# Dockerfile for React App
FROM node:14-alpine as build
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Backend (API)

Create a `Dockerfile` for your backend (Node.js example):

```Dockerfile
# Dockerfile for Node.js API
FROM node:14-alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Database

For the database, you can use a standard Docker image such as PostgreSQL or MySQL. No custom `Dockerfile` is necessary.

## Step 2: Docker Compose for Local Development

Create a `docker-compose.yml` file to orchestrate the multi-container application:

```yaml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://user:password@db:5432/dbname

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## Step 3: GitHub Actions Workflow

Create a `.github/workflows/deploy.yml` file to automate building and deploying your Docker containers.

Example workflow:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    services:
      docker:
        image: docker:19.03.12
        options: --privileged
        ports:
          - 80:80

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push frontend
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/react-frontend:latest ./frontend
          docker push ${{ secrets.DOCKER_USERNAME }}/react-frontend:latest

      - name: Build and push backend
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/node-backend:latest ./backend
          docker push ${{ secrets.DOCKER_USERNAME }}/node-backend:latest

      - name: Deploy to server
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          ssh -i <path_to_key> user@your_server "docker-compose -f /path/to/docker-compose.yml pull && docker-compose -f /path/to/docker-compose.yml up -d"
```

## Step 4: Set Up Secrets

In your GitHub repository, set up the following secrets:

- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub password or access token.
- `SSH_PRIVATE_KEY`: Your private SSH key for accessing the server.

## Step 5: Server Setup

Ensure your server has Docker and Docker Compose installed, and that your `docker-compose.yml` file is in place. The final deployment step will SSH into your server, pull the latest Docker images, and bring up the containers.

## Step 6: Test and Monitor

Once the GitHub Actions workflow is set up, push to the `main` branch and monitor the workflow in the Actions tab of your GitHub repository. You should see your containers built, pushed, and deployed automatically.

### Additional Enhancements

- Add automatic database migrations.
- Implement health checks for your containers.
- Set up rollback strategies in case of failures.

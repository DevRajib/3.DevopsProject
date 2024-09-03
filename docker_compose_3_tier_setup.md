
# Docker Compose Setup for 3-Tier Architecture (React, Django, Postgres, Traefik)

This guide walks you through deploying a 3-tier application using Docker, Traefik for reverse proxy, and Let's Encrypt for SSL.

## Project Directory Structure

```plaintext
my-app/
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
├── db/
├── traefik/
│   └── traefik.toml
└── docker-compose.yml
```

### Step 1: Create Dockerfile for Frontend (React app)
Inside the `frontend/` directory, create a `Dockerfile`:

```Dockerfile
# Frontend Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npx", "serve", "-s", "build"]
```

### Step 2: Create Dockerfile for Backend (Django REST Framework)
Inside the `backend/` directory, create a `Dockerfile`:

```Dockerfile
# Backend Dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt ./
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myproject.wsgi:application"]
```

### Step 3: Create `traefik.toml` Configuration
Inside the `traefik/` directory, create a `traefik.toml` file:

```toml
[entryPoints]
  [entryPoints.http]
    address = ":80"
  [entryPoints.https]
    address = ":443"

[certificatesResolvers.myresolver.acme]
  email = "your-email@example.com"
  storage = "acme.json"
  [certificatesResolvers.myresolver.acme.httpChallenge]
    entryPoint = "http"

[http.routers]
  [http.routers.frontend]
    rule = "Host(\`decorationbd.com\`)"
    entryPoints = ["https"]
    service = "frontend"
    [http.routers.frontend.tls]
      certResolver = "myresolver"

  [http.routers.backend]
    rule = "Host(\`api.decorationbd.com\`)"
    entryPoints = ["https"]
    service = "backend"
    [http.routers.backend.tls]
      certResolver = "myresolver"

[providers.docker]
  exposedByDefault = false
```

### Step 4: Create `docker-compose.yml`
Here's the boilerplate `docker-compose.yml`:

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v2.9
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.http.address=:80"
      - "--entrypoints.https.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=your-email@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik/acme.json:/letsencrypt/acme.json
      - ./traefik/traefik.toml:/etc/traefik/traefik.toml

  frontend:
    build: ./frontend
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(\`decorationbd.com\`)"
      - "traefik.http.routers.frontend.entrypoints=https"
      - "traefik.http.routers.frontend.tls.certresolver=myresolver"
    networks:
      - web

  backend:
    build: ./backend
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.backend.rule=Host(\`api.decorationbd.com\`)"
      - "traefik.http.routers.backend.entrypoints=https"
      - "traefik.http.routers.backend.tls.certresolver=myresolver"
    depends_on:
      - db
    networks:
      - web
      - internal

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - internal

networks:
  web:
    external: true
  internal:
    driver: bridge

volumes:
  db_data:
```

### Step 5: Configure Environment Variables
Make sure to create a `.env` file to manage sensitive data like database credentials, API keys, etc.

### Step 6: Set Up Traefik & SSL
1. Create an empty `acme.json` file in the `traefik/` directory:
   ```bash
   touch traefik/acme.json
   chmod 600 traefik/acme.json
   ```

2. Adjust permissions for Traefik to manage the `acme.json` file.

### Step 7: Deploy to VPS
1. Install Docker and Docker Compose on your VPS.
2. Copy your project to the VPS.
3. Run the following commands:

```bash
docker-compose up -d
```

Traefik will automatically handle the SSL certificates and routing.

### Final Notes
- Replace `your-email@example.com` with your actual email address for Let's Encrypt.
- Ensure that DNS records for `decorationbd.com` and `api.decorationbd.com` are pointed to your VPS IP.

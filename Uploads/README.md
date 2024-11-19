## Containerization Documentation for the Application

### Overview

This documentation provides detailed instructions on how to containerize a 3-tier application comprising the Frontend, Backend, and Database components using Docker. The backend uses a remote MongoDB database, which eliminates the need to containerize the database. This guide includes Dockerfile configurations, Docker Compose setup, and testing instructions.

---

### Prerequisites

- Docker installed on the local machine. You can download Docker from [here](https://www.docker.com/products/docker-desktop).
- Docker Compose installed. You can install it by following [this guide](https://docs.docker.com/compose/install/).

---

### 1. Dockerfiles for Each Component

Each application component (Frontend, Backend, and Database) requires its own container configuration. Below are the necessary configurations.

#### Frontend Dockerfile

```Dockerfile
# Use an official Node.js runtime as a parent image
FROM node:23-alpine3.19 AS build

# Set the working directory
WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the app
RUN npm run build

# Serve the app using a lightweight web server
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80 for the web server
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### Backend Dockerfile

```Dockerfile
# Use a Node.js base image
FROM node:23.1.0-alpine3.19

# Set the working directory
WORKDIR /app

# Copy package files and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the rest of the app files
COPY . .

# Expose the port used by the backend
EXPOSE 3000

# Command to start the app
CMD ["npm", "run", "start"]
```

**Note:** The Database component is for local testing purposes only, as the production backend connects to a remote MongoDB service.

---

### 2. Docker Compose File

The `docker-compose.yml` file orchestrates the setup of the three components (Frontend, Backend, Database). Below is the example Docker Compose configuration.

```yaml
services:
  frontend:
    build:
      context: ./../../fullstack-todo-list/Frontend
      dockerfile: Dockerfile.frontend
    container_name: todo-frontend
    ports:
      - "80:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./../../fullstack-todo-list/Backend
      dockerfile: Dockerfile.backend
    container_name: todo-backend
    ports:
      - "3000:3000"
```

### 3. Setup Instructions

Follow these steps to build, run, and stop the containers.

#### Step 1: Clone the repository

```bash
git clone https://github.com/your-repository-url.git
cd your-repository-directory
```

#### Step 2: Build the containers

```bash
docker-compose build
```

#### Step 3: Run the containers

```bash
docker-compose up
```

This will start the containers and run the services as defined in the `docker-compose.yml`.

#### Step 4: Stop the containers

```bash
docker-compose down
```

This command will stop and remove all the containers.

---

### 4. Network and Security Configurations

- **Ports Exposed:**

  - Frontend: Port 80 is exposed to the host.
  - Backend: Port 3000 is exposed for API calls.
  - Database (if used locally): Port 27017 for MongoDB.

- **Security Considerations:**
  - **MONGO_URI:** The `MONGO_URI` should not contain sensitive information (like database credentials) in plain text. It’s best to use environment variables or a secrets manager for secure configuration.
  - Ensure that Docker containers are not exposed to the public unless necessary. Use firewall rules or VPNs for sensitive services.

---

### 5. Troubleshooting Guide

Here are some common issues and how to resolve them:

#### Issue 1: Containers cannot communicate with each other

- Ensure the services are all on the same Docker network.
- Verify that the correct ports are exposed and mapped in `docker-compose.yml`.

#### Issue 2: Backend cannot connect to the MongoDB service

- Ensure the `MONGO_URI` in the backend service is correctly configured to point to the remote MongoDB server.
- Check MongoDB credentials and access permissions if applicable.

#### Issue 3: Frontend is not accessible

- Check if the `nginx` server is running correctly and has access to the build directory.
- Verify that port 80 is correctly mapped.

---

### 6. Container Testing Script

To verify the containers are running correctly and communicate seamlessly, you can run the following tests:

#### Test Frontend

```bash
curl http://localhost:80
```

- Expected: You should see the frontend application’s homepage.

#### Test Backend API

```bash
curl http://localhost:3000/api/health
```

- Expected: A successful response indicating the backend is running (e.g., `{"status": "ok"}`).

#### Test Database Connection

Since we use a remote MongoDB database, the connection can be tested by running a simple query from the backend container or directly from the MongoDB client.

```bash
docker exec -it <backend-container-id> bash
curl http://localhost:3000/api/database-test
```

- Expected: A successful response confirming the database connection.

---

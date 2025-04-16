# Nexlayer Quickstart Guide

Welcome to Nexlayer! This guide will help you set up, configure, and deploy cutting-edge full-stack web applications effortlessly.

---

## Overview

Nexlayer simplifies the development-to-deployment process with:

- **Automated Docker Image Management**: GitHub Actions for streamlined image building and pushing.
- **Seamless Service Integration**: Effortless configuration of inter-service communication.
- **Hassle-Free Deployment**: Single YAML templates to deploy complex stacks with ease.

---

## 1. Using the Nexlayer Template Builder (Web Interface)

### Step 1: Select Your Stack

1. Log in to your Nexlayer account and select **Deploy a New Template**.
2. Browse through the available stack templates under "Choose your fullstack template."
   - Options include:
     - **MERN Stack**: MongoDB, Express, React, Node.js
     - **MEVN Stack**: MongoDB, Express, Vue.js, Node.js
     - **PERN Stack**: PostgreSQL, Express, React, Node.js
     - **MEAN Stack**: MongoDB, Express, Angular, Node.js
     - **MNFA Stack**: MongoDB, Neo4j, FastAPI, Angular
3. Select the template that matches your application requirements to proceed to the next step.

### Step 2: Configure Your Application

1. After selecting your stack, proceed to the **Configure** step.
2. Customize the following settings:
   - **Pods**: Configure the services (e.g., frontend, backend, database).
   - **Environment Variables**: Add or adjust runtime variables for each service.
     - Example:
       - **MongoDB** pod:
         - `MONGO_INITDB_ROOT_USERNAME`: Set the database root username.
         - `MONGO_INITDB_ROOT_PASSWORD`: Set the root password.
         - `MONGO_INITDB_DATABASE`: Specify the database name.
       - Other variables as needed.
   - **Ports**: Map container ports to service ports as required by your application.
3. Use the visual flow diagram to ensure proper connections between services (e.g., database -> backend -> frontend).
4. Save the configuration by naming your template appropriately (e.g., `my-mern-todo-app.yaml`).

### Step 3: Deploy Your Application

1. Click the **Deploy** button in the Template Builder to launch your application.
2. Monitor deployment progress in the Nexlayer Dashboard.
   - Confirm the services are running as expected.
   - Test connections between the frontend, backend, and database.
3. Your application is now live and accessible via the configured endpoints.

---

## 2. Workflow for Docker Image Creation

To integrate Docker images into Nexlayer:

### 1. Set Up GitHub Actions Workflow

Create a `.github/workflows/docker-publish.yml` file.
Example Github Actions Workflow for a MERN Todo App repository:

```yaml
name: Build and Push MERN Docker Images to GHCR

on:
  workflow_dispatch: # Enable manual trigger
    inputs:
      image_tag: # Input parameter for the image tag
        description: "The tag for the Docker images"
        required: true
        default: "latest"

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write # Required to push images to GHCR

    steps:
      # Step 1: Checkout the repository
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Log in to GitHub Container Registry
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Step 3: Convert repository owner to lowercase (docker image names must all be lowercase)
      - name: Set repository owner lowercase
        id: owner_lowercase
        run: echo "owner_lowercase=$(echo '${{ github.repository_owner }}' | tr '[:upper:]' '[:lower:]')" >> $GITHUB_ENV

      # Step 4: Build the React app
      - name: Install and Build React App
        run: |
          cd react/mern-todo-list
          npm ci # Install dependencies (prefer ci for CI environments)
          npm run build # Build the React app
          cd ../../ # Return to home directory

      # Step 5: Set React image tag
      - name: Set React Image Tag
        id: react_tag
        run: echo "image_tag=${{ github.event.inputs.image_tag }}" >> $GITHUB_ENV

      # Step 6: Build and Push React Docker Image
      - name: Build and Push React App Image
        uses: docker/build-push-action@v5
        with:
          context: react/mern-todo-list
          platforms: linux/amd64
          push: true
          tags: ghcr.io/${{ env.owner_lowercase }}/mern-react-todo:${{ env.image_tag }}

      # Step 7: Set Express image tag
      - name: Set Express Image Tag
        id: express_tag
        run: echo "image_tag=${{ github.event.inputs.image_tag }}" >> $GITHUB_ENV

      # Step 8: Build and Push Express Docker Image
      - name: Build and Push Express App Image
        uses: docker/build-push-action@v5
        with:
          context: express
          platforms: linux/amd64
          push: true
          tags: ghcr.io/${{ env.owner_lowercase }}/mern-express-todo:${{ env.image_tag }}

      # Step 9: Set Mongo image tag
      - name: Set Mongo Image Tag
        id: mongo_tag
        run: echo "image_tag=${{ github.event.inputs.image_tag }}" >> $GITHUB_ENV

      # Step 10: Build and Push Mongo Docker Image
      - name: Build and Push Mongo Image
        uses: docker/build-push-action@v5
        with:
          context: mongoDB
          platforms: linux/amd64
          push: true
          tags: ghcr.io/${{ env.owner_lowercase }}/mern-mongo-todo:${{ env.image_tag }}
```

---

## 3. Configuring the YAML Template to Use Private Images

Here’s an example of a Nexlayer YAML template for a MERN stack utilizing images pulled from the Github Container Registry:

### **Registry Configuration**

```yaml
application:
  template:
    name: "mongodb-express-react-nodejs"
    deploymentName: "My MERN Stack"
    registryLogin:
      registry: ghcr.io
      username: <Github Username>
      personalAccessToken: <GitHub Read:Packages Personal Access Token>
    pods:
      - type: database
        exposeOn80: false
        name: mongoDB
        tag: ghcr.io/<Github Lowercase Username>/mern-mongo:v0.01
        privateTag: true
        vars:
          - key: MONGO_INITDB_ROOT_USERNAME
            value: mongo
          - key: MONGO_INITDB_ROOT_PASSWORD
            value: passw0rd
          - key: MONGO_INITDB_DATABASE
            value: todo
      - type: express
        exposeOn80: false
        name: express
        tag: ghcr.io/<Github Lowercase Username>/mern-express:v0.01
        privateTag: true
        ports:
          - name: express
            containerPort: 3000
            servicePort: 3000
      - type: nginx
        exposeOn80: true
        name: react
        tag: ghcr.io/<Github Lowercase Username>/mern-react:v0.01
        privateTag: true
        vars:
          - key: EXPRESS_URL
            value: BACKEND_CONNECTION_URL
        ports:
          - name: react
            containerPort: 80
            servicePort: 80
```

---

## 4. Monitor Deployment Progress

- Use the **Nexlayer Dashboard** to track real-time progress.

---

## 5. Troubleshooting

### Common Issues

1. **YAML Errors**:

   - Validate syntax before deployment.
   - Avoid comments, this can cause JSON parsing issues.

2. **Docker Image Push Issues**:

   - Ensure lowercase usernames.
   - Verify Personal Access Token permissions.

3. **Deployment Delays**:
   - Check Docker Hub or GHCR connectivity.

---

## 6. Key Features Recap

- **Prebuilt Templates**: Save time with ready-to-deploy configurations.
- **Customizable Workflows**: Flexibility to integrate any stack or runtime.
- **Simplified Deployment**: YAML-driven process reduces complexity.

---

## 7. Need Support?

- Visit the [Nexlayer Documentation](https://docs.nexlayer.io).
- Contact Support: [support@nexlayer.io](mailto:support@nexlayer.io).

---

With Nexlayer, deploying full-stack web applications is fast, scalable, and efficient. Start building today! 🚀

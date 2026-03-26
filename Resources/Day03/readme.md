# Day 3/40 - Multi Stage Docker Build - Docker Tutorial For Beginners - CKA Full Course 2024 ☸️


## Check out the video below for Day3 👇

[![Day 2/40 - How To Dockerize a Project - CKA Full Course 2024](https://img.youtube.com/vi/ajetvJmBvFo/sddefault.jpg)](https://youtu.be/ajetvJmBvFo)

# Pre-requisites ( If you have followed Day2 video and/or already have Docker Setup, then skip this step)

## If you would like to use docker and Kubernetes sandbox environment , you can use below:
```
https://labs.play-with-docker.com/
https://labs.play-with-k8s.com/
```

## Download Docker desktop client
```
https://www.docker.com/products/docker-desktop/
```

# Getting started with the demo

- Clone the below sample repository, or you can use any web application that you have

```
git clone https://github.com/piyushsachdeva/todoapp-docker.git
```

- cd into the directory
```
cd todoapp-docker/
```
- Create an empty file with the name Dockerfile
```
touch Dockerfile
```

- Using the text editor of your choice, paste the below content:
Note: Details about the below Dockerfile have already been shared in the video
```
FROM node:18-alpine AS installer
WORKDIR /app
COPY package*.json ./
RUN npm install 
COPY . .
RUN npm run build
FROM nginx:latest AS deployer
COPY --from=installer /app/build /usr/share/nginx/html
```

- Build the docker image using the application code and Dockerfile

```
docker build -t todoapp-docker .
```
- Verify the image has been created and stored locally using the below command:
```
docker images
```

- Create a public repository on hub.docker.com and push the image to remote repo
```
docker login
docker tag todoapp-docker:latest username/new-reponame:tagname
docker images
docker push username/new-reponame:tagname
```

- To pull the image to another environment, you can use the below command
```
docker pull username/new-reponame:tagname
```

- To start the docker container, use the below command

```
docker run -dp 3000:80 username/new-reponame:tagname
```

- Verify your app. If you have followed the above steps correctly, your app should be listening on localhost:3000
- To enter(exec) into the container, use the below command

```
docker exec -it containername sh
or
docker exec -it containerid sh
```
- To view docker logs

```
docker logs containername
or
docker logs containerid
```

- To view the content of Docker container
```
docker inspect
```

- Cleanup the old docker images from local repo using below command:

```
docker image rm image-id
```


## Best Practices

Great — these are **real production-level Docker + React optimization practices** (exactly what companies expect in DevOps/Frontend interviews). I’ll explain each one clearly and then go **deeper into React-specific Docker caching + optimization**, which is where most people lack understanding.

---

# 🔥 1. Run Containers as Non-Root User (USER instruction)

### ❌ Problem:

By default, Docker containers run as **root**, which is a security risk.

### ✅ Solution:

Use a non-root user:

```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

### 💡 Why important:

* Prevents privilege escalation
* Required in **Kubernetes production environments**
* Security best practice (DevSecOps)

---

# ⚡ 2. Make Docker Image Lightweight

### ❌ Problem:

Large images = slow builds + slow deployments

### ✅ Solutions:

#### Use minimal base images:

```dockerfile
FROM node:18-alpine
```

Instead of:

```dockerfile
FROM node:18
```

#### Use multi-stage builds:

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
```

### 💡 Benefits:

* Removes dev dependencies
* Final image is **very small (~20–30MB)**

---

# ⚡ 3. Use Docker Build Cache (VERY IMPORTANT)

### 🧠 Core Concept:

Docker caches layers. If nothing changes in a layer → it reuses it.

### ❌ Bad Dockerfile:

```dockerfile
COPY . .
RUN npm install
```

👉 Any code change → npm install runs again (slow)

---

### ✅ Optimized Dockerfile:

```dockerfile
COPY package.json package-lock.json ./
RUN npm install   # cached

COPY . .          # app code changes here
```

### 💡 Why:

* Dependencies change less often than code
* Saves **huge build time**

---

### 🔁 Use External Cache:

```bash
docker build -t myapp . --cache-from myapp:latest
```

👉 Reuses previous image layers

---

# 🔐 4. Use Secrets for Sensitive Data

### ❌ Never do:

```dockerfile
ENV API_KEY=123456
```

### ✅ Use Docker secrets / runtime injection:

#### Example (Docker run):

```bash
docker run --env-file .env myapp
```

#### In Kubernetes:

* Use **Secrets**
* Mount as environment variables

### 💡 Why:

* Prevents secrets leaking in image layers
* Safer for GitHub / CI/CD

---

# ⚡ 5. Reduce Layers + Use Efficient RUN

### ❌ Bad:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
```

### ✅ Good:

```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

### 💡 Why:

* Fewer layers → smaller image
* Cleaner builds

---

# 🚀 6. Advanced: Mount Cache in Docker Build (BuildKit)

### Enable BuildKit:

```bash
DOCKER_BUILDKIT=1 docker build .
```

### Example:

```dockerfile
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

### 💡 Why:

* Reuses npm cache across builds
* Faster CI/CD pipelines

---

# ⚛️ React App Docker Optimization (MOST IMPORTANT FOR YOU)

Now let’s go **deep into React-specific optimization** 👇

---

# 🔥 7. React Build Optimization Strategy

### 🚀 Best Practice Flow:

```
Code → Build → Static Files → Serve via Nginx
```

---

## ✅ Optimized Dockerfile for React

```dockerfile
# Stage 1: Build React App
FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

# ⚡ 8. Use `npm ci` instead of `npm install`

### Why:

* Faster
* Deterministic builds
* Uses `package-lock.json`

---

# ⚡ 9. Optimize React Build Size

### In React:

#### Enable production build:

```bash
npm run build
```

#### Techniques:

* Code splitting (`React.lazy`)
* Tree shaking
* Remove unused libraries

---

# ⚡ 10. Use .dockerignore (CRITICAL)

### Example:

```
node_modules
.git
build
Dockerfile
README.md
```

### 💡 Why:

* Reduces build context size
* Faster builds

---

# ⚡ 11. Nginx Optimization for React

### Add caching headers:

```nginx
location /static/ {
    expires 1y;
    add_header Cache-Control "public";
}
```

### 💡 Why:

* Browser caching → faster load

---

# ⚡ 12. Layer Ordering Strategy (Interview Gold)

Order matters:

```
1. Base image
2. Dependencies (package.json)
3. Install deps
4. Copy source code
5. Build
```

👉 This maximizes caching

---

# ⚡ 13. CI/CD Optimization

In pipelines:

* Cache:

  * `node_modules`
  * `.npm`
* Use:

```bash
docker build --cache-from
```

---

# ⚡ 14. Use Distroless Images (Advanced)

Instead of nginx:

```dockerfile
FROM gcr.io/distroless/nodejs
```

### 💡 Benefits:

* Ultra secure
* Very small
* No shell (harder to attack)

---

# ⚡ 15. Use Compression

Enable gzip in nginx:

```nginx
gzip on;
gzip_types text/css application/javascript;
```

---

# 🧠 Final Summary (Interview Ready Answer)

If interviewer asks:

👉 *“How do you optimize Docker for React apps?”*

You say:

* Use **multi-stage builds**
* Use **alpine images**
* Optimize **layer caching (copy package.json first)**
* Use **npm ci**
* Use **.dockerignore**
* Serve via **nginx with caching**
* Use **BuildKit cache mounts**
* Keep containers **non-root**
* Avoid secrets in image
* Minimize layers

---

# 🚀 Pro Tip (Real Industry Insight)

Most beginners fail here:

👉 They rebuild dependencies every time
👉 They ship node_modules in production
👉 They use node instead of nginx for serving

**You now understand production-level optimization.**

---


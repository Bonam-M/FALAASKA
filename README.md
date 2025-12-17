# ALAASKA Overview and Deployment Guide

## 📋 Table of Contents
- [Overview](#overview)
- [1. Prerequisistes](#1-prerequisites)
- [2. Deployment Instructions (Ubuntu)](#2-deployment-instructions)
  - [2.1 Install Dependencies](#21-install-dependencies)
  - [2.2 Clone and Setup Application](#22-clone-and-setup-application)
  - [2.3 Configure Environment Variables](#23-configure-environment-variables)
  - [2.4 Build and run Docker containers](#24-build-and-run-docker-containers)
  - [2.5 Start Application](#25-start-application-)
- [3. Required Manual Steps](#4-required-manual-steps)
- [4. Troubleshooting](#6-troubleshooting)

---

## Overview
**ALAASKA** (Adaptive Learning for All through AI-Powered Student Knowledge Assessment) is a full-stack web application that delivers intelligent, personalized tutoring experiences powered by a fine-tuned language model. Designed with a strong pedagogical foundation, ALAASKA simulates the behavior of a supportive tutor who uses microlearning materials like flashcards, guiding questions, and mini-quizzes to promote intuitive problem solving and learner autonomy. 

### UI Overview
<img src="https://github.com/Bonam-M/FALAASKA/blob/main/frontend/src/assets/alaaska_hw.png" alt="Platform Overview" width="600" />
<br>

The system supports persistent multi-session tutoring through a sidebar-based conversation manager, markdown-rendered messaging, and token-authenticated communication between the client and server. Users can log in, start conversations about specific assignments, submit conversations for review, manage their past conversations, and engage in real-time learning conversations with the model. ALAASKA is a research project that can serves as a framework for researchers, educators, or developers building AI-powered adaptive learning tools with pedagogical constraints.  
<br>

### Implementation Overview
ALAASKA is built with:
- **Frontend**: React 18.3.1
- **Backend**: FastAPI with Python 3.10.16
- **Database**: MongoDB Community Edition 7.0
- **AI**: OpenAI GPT integration
- **Authentication**: Auth0 
- **Deployment**: Docker

---

## 1. Prerequisites

### Required Software Versions
- **Node.js**: v24.4.1
- **npm**: 11.4.2
- **Python**: 3.10.16
- **MongoDB**: Community Edition 7.0
- **Docker**: Docker Desktop 29.1.3

### Required Environment Variables
You will need the following:
- OPENAI_API_KEY=[To be provided], MODEL_ID=[To be provided], SUMMARIZE_MODEL_ID=[To be provided]
- MONGODB_URL=[Local instance to be defined], MONGODB_CLIENT=[Database name to be defined]
- AUTH0_DOMAIN=[To be provided], AUTH0_CLIENT_ID=[To be provided], AUTH0_API_AUDIENCE=[To be provided]
- ALGORITHM=[To be provided]
- BACKEND_URL=[To be defined], FRONTEND_UR=[To be defined], Backend SECRET_KEY=[To be defined]

---

## 2. Deployment Instructions (Ubuntu)

### 2.1 Install Software Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Uninstall all conflicting packages
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1)

#apt might report that you have none of these packages installed. All good!

# Install Docker (Skip if not needed)
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

# Install the Docker packages (latest version)
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify that Docker is running
sudo systemctl status docker
# Or
docker compose --version

# OPTIONAL: Verify that Docker installation was successful 
sudo docker run hello-world

```

### 2.2 Clone and Setup Application
```bash
# Clone repository
git clone THIS_REPO_URL falaaska_app
cd falaaska_app

# Verify that you have frontend/Dockerfile, backend/dockerfile, 
# docker-compose.yml and nginx.conf files in the project directory

```
### OPTIONAL: Add restart policies to docker-compose.yml
```yaml
services:
  backend:
    build: ./backend
    restart: unless-stopped

  frontend:
    build: ./frontend
    restart: unless-stopped
```

### 2.3 Configure Environment Variables

**Backend create (.env) file:**
```bash
cd backend
cat > .env << 'EOF'
MONGODB_URL=mongodb://YOUR_SERVER_IP:27017 [OR To be defined]
MONGODB_CLIENT=[To be defined]
OPENAI_API_KEY=your_openai_key_here [To be provided]
MODEL_ID=[To be provided]
SUMMARIZE_MODEL_ID=[To be provided]
AUTH0_DOMAIN=your_domain.auth0.com [To be provided]
AUTH0_CLIENT_ID=[To be provided]
AUTH0_API_AUDIENCE=your_api_audience [To be provided]
ALGORITHM=RS256
SECRET_KEY=generate_64_char_random_string_here
FRONTEND_URL=http://YOUR_SERVER_IP:3000
BACKEND_URL=http://YOUR_SERVER_IP:8000
EOF
```

**Frontend (.env):**
```bash
cd ../frontend  
cat > .env << 'EOF'
REACT_APP_AUTH0_DOMAIN=your_domain.auth0.com [To be provided]
REACT_APP_AUTH0_CLIENT_ID=your_client_id_here [To be provided]
REACT_APP_AUTH0_API_AUDIENCE=your_api_audience [To be provided]
REACT_APP_API_URL=http://YOUR_SERVER_IP:8000
```

### 2.4 Build and run Docker containers
```bash
cd <PROJECT DIRECTORY>
docker compose up -d --build

# Check running containers
docker compose ps

# OPTIONAL: Enable Docker to start on boot
sudo systemctl enable docker
```

### 2.5 Start Application (Verify deployment)

```bash
# Visit your app on a web browser
# Frontend: `http://YOUR_SERVER_IP:3000`

# Reboot the server to ensure containers restart automatically
```bash
sudo reboot

# Stop containers 
docker compose down

# Restart containers 
docker compose restart

#Test URLs:
- Frontend: `http://YOUR_SERVER_IP:3000`
- Backend API: `http://YOUR_SERVER_IP:8000/docs`
```
---

## 3. Required Manual Steps

### 3.1 Update Environment Variables
Replace all placeholders with actual values:
- `YOUR_SERVER_IP` → Server IP address
- `your_openai_key_here` → OpenAI API key
- `your_domain.auth0.com` → Auth0 domain
- `your_client_id_here` → Auth0 client ID  
- `your_api_audience` → Auth0 API audience
- `generate_64_char_random_string_here` → Generate with: `openssl rand -hex 32`

### 3.2 Auth0 Configuration
In Auth0 dashboard, set:
- **Allowed Callback URLs**: `PROD FRONTEND AND BACKEND URLS`
- **Allowed Web Origins**: `PROD FRONTEND AND BACKEND URLS`
- **Allowed Logout URLs**: `PROD FRONTEND AND BACKEND URLS`

---

## 4. Troubleshooting
```bash
# View logs
docker compose logs -f
```
---

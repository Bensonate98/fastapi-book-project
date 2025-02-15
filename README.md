# FastAPI Book Project

## Overview
This is a FastAPI-based application for managing books. The project includes a fully functional API with a CI/CD pipeline using GitHub Actions and deployment on an AWS EC2 instance with Nginx as a reverse proxy.

## Features
- RESTful API for book management
- CI/CD pipeline for automated testing and deployment
- Deployed on AWS EC2 with Nginx as a reverse proxy
- Secure and scalable architecture

## Technologies Used
- **Backend:** FastAPI, Uvicorn
- **Database:** SQLite (or PostgreSQL for production)
- **CI/CD:** GitHub Actions
- **Deployment:** AWS EC2, Nginx, Systemd
- **Testing:** Pytest

---

## Getting Started
### Prerequisites
Ensure you have the following installed:
- Python 3.10+
- pip & virtualenv
- Git
- AWS EC2 instance (Ubuntu 20.04 or later)
- SSH access to the instance

### Local Setup
1. **Clone the repository:**
   ```sh
   git clone https://github.com/bensonate98/fastapi-book-project.git
   cd fastapi-book-project
   ```

2. **Create a virtual environment and install dependencies:**
   ```sh
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Run the application locally:**
   ```sh
   uvicorn main:app --host 127.0.0.1 --port 8000 --reload
   ```

4. **Access the API:**
   Open your browser and go to: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## Deployment Guide
### 1. Set Up AWS EC2 Instance
1. **Launch an EC2 instance** (Ubuntu 20.04 recommended)
2. **Connect to the instance via SSH:**
   ```sh
   ssh -i your-key.pem ubuntu@your-ec2-public-ip
   ```

### 2. Configure the Server
1. **Update packages and install dependencies:**
   ```sh
   sudo apt update && sudo apt install -y python3-pip python3-venv git nginx
   ```

2. **Clone the repository:**
   ```sh
   git clone https://github.com/bensonate98/fastapi-book-project.git
   cd fastapi-book-project
   ```

3. **Set up the virtual environment:**
   ```sh
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

4. **Create a systemd service for FastAPI:**
   ```sh
   sudo nano /etc/systemd/system/fastapi.service
   ```
   Add the following content:
   ```ini
   [Unit]
   Description=FastAPI App
   After=network.target

   [Service]
   User=ubuntu
   Group=ubuntu
   WorkingDirectory=/home/ubuntu/fastapi-book-project
   ExecStart=/home/ubuntu/fastapi-book-project/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
   Save and exit.

5. **Enable and start the service:**
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl enable fastapi
   sudo systemctl start fastapi
   ```

6. **Check service status:**
   ```sh
   sudo systemctl status fastapi
   ```

### 3. Configure Nginx as a Reverse Proxy
1. **Create an Nginx configuration file:**
   ```sh
   sudo nano /etc/nginx/sites-available/fastapi
   ```
   Add the following content:
   ```nginx
   server {
       listen 80;
       server_name your-ec2-public-ip;

       location / {
           proxy_pass http://127.0.0.1:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```
   Save and exit.

2. **Enable the configuration and restart Nginx:**
   ```sh
   sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/
   sudo systemctl restart nginx
   ```

3. **Allow HTTP traffic:**
   ```sh
   sudo ufw allow 'Nginx Full'
   ```

### 4. CI/CD Pipeline
#### Continuous Integration (CI) - Testing
Create `.github/workflows/test.yml`:
```yaml
name: CI Pipeline

on:
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

#### Continuous Deployment (CD) - Auto Deploy on Push to Main
Create `.github/workflows/deploy.yml`:
```yaml
name: CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH into Server and Deploy
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ubuntu
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /home/ubuntu/fastapi-book-project
            git pull origin main
            source venv/bin/activate
            pip install -r requirements.txt
            sudo systemctl restart fastapi
```

### 5. Add Secrets to GitHub
Go to **GitHub → Repository → Settings → Secrets → Actions**:
- `SERVER_IP`: `your-ec2-public-ip`
- `SSH_PRIVATE_KEY`: Paste the content of `ben.pem`

---

## Testing & Verification
1. **Check running services:**
   ```sh
   sudo systemctl status fastapi
   sudo systemctl status nginx
   ```
2. **Test the API:**
   Open: `http://your-ec2-public-ip/docs`

---

## Conclusion
🎉 Your FastAPI application is now live with CI/CD and Nginx!

If you have any issues, feel free to open an issue in the repository. 🚀


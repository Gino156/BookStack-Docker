# 📚 BookStack Docker Setup on WSL2 (Ubuntu)

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![BookStack](https://img.shields.io/badge/BookStack-0284C7?style=for-the-badge&logo=bookstack&logoColor=white)](https://www.bookstackapp.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)

A streamlined installation guide and `docker-compose` configuration for deploying [BookStack](https://github.com/linuxserver/docker-bookstack) using Docker on Windows Subsystem for Linux (WSL2 - Ubuntu).

---

## 📋 Prerequisites

Before starting, ensure you have:
* **WSL2 (Ubuntu)** installed and configured on Windows.
* **Docker** installed either via Docker Desktop (with WSL2 backend integration) or natively inside Ubuntu WSL.

---

## 🚀 Installation & Setup Steps

### 1. Start Docker Service
Open your WSL Ubuntu terminal and start the Docker daemon:

```bash
sudo service docker start
Enter your sudo password when prompted.
```

2. Create the Project Directory
Create a dedicated folder for your BookStack deployment and navigate into it:

```bash
mkdir -p ~/bookstack && cd ~/bookstack
```
3. Generate an Application Key (APP_KEY)
BookStack requires an encryption key before it can start. Generate one by running:

```bash
docker run -it --rm --entrypoint /bin/bash lscr.io/linuxserver/bookstack:latest appkey
```

Copy the generated output string (e.g., base64:UJGtYKyNuJWPps6WU4ij0/ZMToEviZJgjh82DqC6ij0=).

4. Create and Configure docker-compose.yml
Open the file editor:

```bash
nano docker-compose.yml
```
Paste the following configuration into the file (make sure to update the APP_KEY line with your generated key):

```bash
version: "3.8"

services:
  bookstack:
    image: lscr.io/linuxserver/bookstack:latest
    container_name: bookstack
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Manila
      - APP_URL=http://localhost:6875
      - APP_KEY=base64:UJGtYKyNuJWPps6WU4ij0/ZMToEviZJgjh82DqC6ij0=
      - DB_HOST=bookstack_db
      - DB_PORT=3306
      - DB_USERNAME=bookstack
      - DB_PASSWORD=secretpassword
      - DB_DATABASE=bookstackapp
    volumes:
      - ./config:/config
    ports:
      - 6875:80
    restart: unless-stopped
    depends_on:
      - bookstack_db

  bookstack_db:
    image: lscr.io/linuxserver/mariadb:latest
    container_name: bookstack_db
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Manila
      - MYSQL_ROOT_PASSWORD=rootsecretpassword
      - MYSQL_DATABASE=bookstackapp
      - MYSQL_USER=bookstack
      - MYSQL_PASSWORD=secretpassword
    volumes:
      - ./db_config:/config
    restart: unless-stopped
```
Save and exit nano by pressing Ctrl + O, Enter, then Ctrl + X.

5. Launch the Containers
Start BookStack and MariaDB in detached mode:

```bash
docker compose up -d
```
6. Verify Installation
To monitor initialization and view logs, execute:

```bash
docker logs -f bookstack
```

Accessing BookStack
Once the containers are initialized, open your browser in Windows and navigate to:

URL: http://localhost:6875

Default Credentials:

Email: admin@admin.com

Password: password

⚠️ Security Notice: Change the default administrator email and password immediately after logging in by visiting Edit Profile / Settings.

🛠️ Management Commands
Stop containers:

```bash
docker compose down
```
Restart containers:

```bash
docker compose restart
```
Clear application cache (troubleshooting):

```bash
docker exec -it bookstack php /app/www/artisan cache:clear
docker exec -it bookstack php /app/www/artisan view:clear
```
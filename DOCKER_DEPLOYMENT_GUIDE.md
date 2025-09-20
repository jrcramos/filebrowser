# Docker Deployment Guide for File Browser

This guide will help you deploy File Browser using Docker, even if you've never used Docker before. File Browser is a web-based file manager that allows you to access and manage your files through a web interface.

## What is Docker?

Docker is a platform that allows you to run applications in containers. Think of containers as lightweight, portable packages that include everything needed to run an application. This makes deployment much easier and more consistent across different systems.

## Prerequisites

### Install Docker

You need to have Docker installed on your system. Here's how to install it on different platforms:

#### Windows
1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Run the installer and follow the setup wizard
3. Restart your computer when prompted
4. Launch Docker Desktop and wait for it to start

#### macOS
1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Drag Docker to your Applications folder
3. Launch Docker from Applications and follow the setup

#### Linux (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install Docker
sudo apt install docker.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (optional, to avoid using sudo)
sudo usermod -aG docker $USER
```

### Verify Docker Installation

Open a terminal (Command Prompt on Windows) and run:
```bash
docker --version
```

You should see something like `Docker version 20.10.x, build...`

## Quick Start - Deploy File Browser

### Method 1: Using Bind Mounts (Recommended for Specific Folders)

This method directly mounts your local folder (like `Y:\media`) into the container.

#### For Windows Users (Your Y:\media folder case):

```bash
docker run -d \
  --name filebrowser \
  -v Y:\media:/srv \
  -v filebrowser_database:/database \
  -v filebrowser_config:/config \
  -p 8080:80 \
  --restart unless-stopped \
  filebrowser/filebrowser
```

#### For Linux/macOS Users:

```bash
docker run -d \
  --name filebrowser \
  -v /path/to/your/files:/srv \
  -v filebrowser_database:/database \
  -v filebrowser_config:/config \
  -p 8080:80 \
  --restart unless-stopped \
  filebrowser/filebrowser
```

Replace `/path/to/your/files` with the actual path to your files (e.g., `/home/username/Documents`).

#### Command Explanation:
- `-d`: Run container in background (detached mode)
- `--name filebrowser`: Give the container a name for easy reference
- `-v Y:\media:/srv`: Mount your Y:\media folder to the container's /srv directory
- `-v filebrowser_database:/database`: Create a Docker volume for the database
- `-v filebrowser_config:/config`: Create a Docker volume for configuration
- `-p 8080:80`: Map port 8080 on your computer to port 80 in the container
- `--restart unless-stopped`: Automatically restart the container if it stops
- `filebrowser/filebrowser`: The Docker image to use

### Method 2: Using Docker Volumes Only

This method creates Docker-managed storage volumes:

```bash
docker run -d \
  --name filebrowser \
  -v filebrowser_data:/srv \
  -v filebrowser_database:/database \
  -v filebrowser_config:/config \
  -p 8080:80 \
  --restart unless-stopped \
  filebrowser/filebrowser
```

With this method, you'll need to copy files into the volume or use the web interface to upload them.

## Accessing File Browser

1. Wait a few seconds for the container to start
2. Open your web browser and go to: `http://localhost:8080`
3. You should see the File Browser login page

## First Time Setup

### Getting Initial Login Credentials

To see the initial admin password, check the container logs:

```bash
docker logs filebrowser
```

Look for a line that shows the admin password. It will look something like:
```
2024/01/01 12:00:00 admin login created with the password: RANDOM_PASSWORD_HERE
```

### Login
- **Username**: `admin`
- **Password**: Use the password from the logs above

> **Important**: The randomly generated password is only shown once in the logs. Make sure to note it down or change it immediately after logging in.

### Change Default Password (Recommended)

1. Log in with the admin credentials
2. Click on "Settings" (gear icon) in the top right
3. Go to "Profile Settings" or "User Management"
4. Change your password to something secure and memorable

## Managing Your Container

### View Container Status
```bash
docker ps
```

### Stop the Container
```bash
docker stop filebrowser
```

### Start the Container Again
```bash
docker start filebrowser
```

### View Container Logs
```bash
docker logs filebrowser
```

### Remove the Container (if you want to start fresh)
```bash
# Stop the container first
docker stop filebrowser

# Remove the container
docker rm filebrowser

# Note: This won't delete your volumes/data
```

## Advanced Configuration

### Using Docker Compose (Recommended for Production)

Create a file named `docker-compose.yml`:

```yaml
version: '3.8'

services:
  filebrowser:
    image: filebrowser/filebrowser
    container_name: filebrowser
    ports:
      - "8080:80"
    volumes:
      - Y:\media:/srv                    # Windows path example
      - filebrowser_database:/database
      - filebrowser_config:/config
    restart: unless-stopped

volumes:
  filebrowser_database:
  filebrowser_config:
```

Then run:
```bash
docker-compose up -d
```

### Custom Port

To use a different port (e.g., 3000 instead of 8080):
```bash
docker run -d \
  --name filebrowser \
  -v Y:\media:/srv \
  -v filebrowser_database:/database \
  -v filebrowser_config:/config \
  -p 3000:80 \
  --restart unless-stopped \
  filebrowser/filebrowser
```

Then access it at `http://localhost:3000`.

## Troubleshooting

### Container Won't Start

1. **Check if port is already in use**:
   ```bash
   # Windows
   netstat -an | findstr :8080
   
   # Linux/macOS
   netstat -tlnp | grep :8080
   ```
   If something is using port 8080, use a different port like `-p 3000:80`.

2. **Check Docker logs**:
   ```bash
   docker logs filebrowser
   ```

### Permission Issues (Linux/macOS)

If you get permission errors with bind mounts:

```bash
# Make sure the directory is readable/writable
chmod 755 /path/to/your/files

# Or change ownership to Docker user (UID 1000)
sudo chown -R 1000:1000 /path/to/your/files
```

### Can't Access Web Interface

1. **Verify container is running**:
   ```bash
   docker ps
   ```
   
2. **Check if the port is correctly mapped**:
   The output should show something like `0.0.0.0:8080->80/tcp`

3. **Try accessing from different addresses**:
   - `http://localhost:8080`
   - `http://127.0.0.1:8080`
   - `http://YOUR_LOCAL_IP:8080`

### Windows Path Issues

If you have issues with Windows paths:

1. **Use forward slashes**:
   ```bash
   -v C:/Users/YourName/Documents:/srv
   ```

2. **Or use PowerShell format**:
   ```bash
   -v C:\Users\YourName\Documents:/srv
   ```

3. **For network drives**, make sure Docker Desktop has access to the network location.

### Updating File Browser

To update to the latest version:

```bash
# Stop and remove old container
docker stop filebrowser
docker rm filebrowser

# Pull latest image
docker pull filebrowser/filebrowser

# Run with same settings as before
docker run -d \
  --name filebrowser \
  -v Y:\media:/srv \
  -v filebrowser_database:/database \
  -v filebrowser_config:/config \
  -p 8080:80 \
  --restart unless-stopped \
  filebrowser/filebrowser
```

## Security Considerations

1. **Change the default admin password** immediately after first login
2. **Don't expose File Browser directly to the internet** without proper security measures
3. **Use HTTPS** in production (consider using a reverse proxy like Nginx)
4. **Regular backups** of your data and configuration volumes

## Backup Your Data

Your files are in the mounted directory (e.g., `Y:\media`), but you should also backup the configuration and database:

```bash
# Create backup directory
mkdir filebrowser-backup

# Backup database volume
docker run --rm -v filebrowser_database:/data -v $(pwd)/filebrowser-backup:/backup alpine tar czf /backup/database.tar.gz -C /data .

# Backup config volume  
docker run --rm -v filebrowser_config:/data -v $(pwd)/filebrowser-backup:/backup alpine tar czf /backup/config.tar.gz -C /data .
```

## Getting Help

- **File Browser Documentation**: [https://filebrowser.org](https://filebrowser.org)
- **Docker Documentation**: [https://docs.docker.com](https://docs.docker.com)
- **File Browser GitHub**: [https://github.com/filebrowser/filebrowser](https://github.com/filebrowser/filebrowser)

## Summary

You now have File Browser running in Docker! You can:
- Access your files at `http://localhost:8080`
- Upload, download, and manage files through the web interface
- Share files with others by giving them access to your File Browser instance
- Access your `Y:\media` folder (or any other folder you mounted) through the web interface

The container will automatically start when you boot your computer (thanks to `--restart unless-stopped`), so File Browser will always be available when you need it.
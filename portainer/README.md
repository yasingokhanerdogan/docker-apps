# Portainer Docker Setup

A Docker container management platform using Portainer Community Edition. Provides a web-based interface for managing Docker containers, images, networks, and volumes.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd portainer
   ```

2. **Start Portainer**
   ```bash
   docker-compose up -d
   ```

3. **Access Portainer**
   - Open your browser and go to: `http://localhost:9000`
   - Create your admin user on first visit

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Port 9000 available
- Docker socket access (`/var/run/docker.sock`)

## ⚙️ Configuration

### Current Setup Features

- **Analytics Disabled**: `--no-analytics` flag for privacy
- **Security Enhanced**: `no-new-privileges:true` for security
- **Timezone**: Set to `Europe/Istanbul`
- **Auto Restart**: Container automatically restarts
- **Persistent Data**: Volume mounted for data persistence

### Customization Options

To customize the timezone, modify the environment section:
```yaml
environment:
  - TZ=Your/Timezone  # e.g., America/New_York, Asia/Tokyo
```

Available timezones: [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

## 🐳 Docker Service

### Portainer CE
- **Image**: `portainer/portainer-ce:latest`
- **Port**: `9000:9000`
- **Container**: `portainer`
- **Volume**: `portainer_data` for persistent storage
- **Network**: `portainer_network` (bridge)
- **Security**: Enhanced with `no-new-privileges`

## 📁 Volume Management

This setup creates persistent Docker volume:

- `portainer_data` - Portainer configuration and data

### Backup

To backup your Portainer data:

```bash
# Create backup directory
mkdir -p backup

# Stop Portainer
docker-compose down

# Backup data volume
docker run --rm -v portainer_data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/portainer-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Start Portainer
docker-compose up -d
```

### Restore

To restore from backup:

```bash
# Stop Portainer
docker-compose down

# Remove existing volume
docker volume rm portainer_data

# Restore data
docker run --rm -v portainer_data:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/portainer-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

# Start Portainer
docker-compose up -d
```

## 🔧 Management Commands

### Start Portainer
```bash
docker-compose up -d
```

### Stop Portainer
```bash
docker-compose down
```

### View logs
```bash
docker-compose logs -f portainer
```

### Update Portainer
```bash
# Pull latest image
docker-compose pull

# Recreate container
docker-compose up -d --force-recreate
```

### Restart Portainer
```bash
docker-compose restart
```

## 💼 Using Portainer

### Initial Setup

1. **First Time Access**: Navigate to `http://localhost:9000`
2. **Create Admin User**: Set username and password
3. **Choose Environment**: Select "Docker" for local Docker environment
4. **Connect**: Portainer will automatically connect to your local Docker

### Key Features

#### Container Management
- **View Containers**: See all running and stopped containers
- **Start/Stop/Restart**: Control container lifecycle
- **Logs**: View real-time container logs
- **Terminal**: Access container shell
- **Inspect**: View detailed container information

#### Image Management
- **Pull Images**: Download images from registries
- **Build Images**: Create images from Dockerfiles
- **Remove Images**: Clean up unused images
- **Image Layers**: Inspect image layers

#### Volume Management
- **Create Volumes**: Create new Docker volumes
- **Browse Files**: Explore volume contents
- **Backup/Restore**: Manage volume data

#### Network Management
- **Create Networks**: Set up custom networks
- **Network Inspection**: View network details
- **Container Connectivity**: Manage container networking

#### Stacks (Docker Compose)
- **Deploy Stacks**: Deploy multi-container applications
- **Compose Files**: Upload and manage docker-compose files
- **Stack Management**: Start, stop, update stacks

## 🌐 Remote Docker Management

To manage remote Docker hosts:

1. **Access Remote Docker**: Use Docker API endpoints
2. **TLS Security**: Configure TLS certificates for secure connections
3. **Add Endpoints**: Add remote Docker environments in Portainer

Example remote endpoint:
```
Name: Remote Server
Endpoint URL: tcp://remote-server-ip:2376
TLS: Enabled with certificates
```

## 🔐 Security Considerations

### Current Security Features

- **No New Privileges**: Prevents privilege escalation
- **Read-only Docker Socket**: Mounted for container management
- **Network Isolation**: Uses dedicated bridge network

### Additional Security Recommendations

1. **Reverse Proxy**: Use Nginx/Traefik with SSL
2. **Authentication**: Configure external authentication (LDAP, OAuth)
3. **Access Control**: Set up role-based access control
4. **Firewall**: Restrict access to port 9000
5. **Regular Updates**: Keep Portainer updated

### Production Setup with SSL

```yaml
# Example with Traefik reverse proxy
services:
  portainer:
    # ... existing config
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer.rule=Host(`portainer.yourdomain.com`)"
      - "traefik.http.routers.portainer.tls.certresolver=letsencrypt"
    # Remove ports section when using reverse proxy
```

## 🚨 Troubleshooting

### Common Issues

1. **Cannot access Portainer UI**
   ```bash
   # Check if container is running
   docker-compose ps
   
   # Check logs
   docker-compose logs portainer
   
   # Verify port is not in use
   sudo lsof -i :9000
   ```

2. **Permission denied accessing Docker socket**
   ```bash
   # Check Docker socket permissions
   ls -la /var/run/docker.sock
   
   # Add user to docker group (if needed)
   sudo usermod -aG docker $USER
   newgrp docker
   ```

3. **Portainer shows no containers**
   ```bash
   # Check Docker socket mount
   docker inspect portainer | grep -A 5 Mounts
   
   # Restart Portainer
   docker-compose restart
   ```

4. **Admin user locked out**
   ```bash
   # Reset admin password (requires stopping Portainer)
   docker-compose down
   docker run --rm -v portainer_data:/data alpine rm /data/portainer.db
   docker-compose up -d
   # Navigate to UI and recreate admin user
   ```

### Reset Portainer

To completely reset Portainer:

```bash
# Stop Portainer
docker-compose down

# Remove data volume (WARNING: This deletes all data!)
docker volume rm portainer_data

# Start fresh
docker-compose up -d
```

## 📊 Monitoring and Maintenance

### Regular Maintenance Tasks

1. **Clean Up**: Remove unused containers, images, volumes
2. **Updates**: Keep Portainer updated to latest version
3. **Backups**: Regular backup of Portainer data
4. **Security**: Monitor access logs and user activities

### Performance Monitoring

- **Resource Usage**: Monitor Docker host resources through Portainer
- **Container Stats**: View real-time container performance
- **Image Usage**: Track image storage usage

## 📖 Documentation

- [Portainer Official Documentation](https://docs.portainer.io/)
- [Portainer CE GitHub](https://github.com/portainer/portainer)
- [Docker Documentation](https://docs.docker.com/)

## 🆘 Support

- [Portainer Community Forums](https://community.portainer.io/)
- [Portainer Slack](https://portainer.slack.com/)
- [GitHub Issues](https://github.com/portainer/portainer/issues)

## 📄 License

This setup configuration is provided as-is. Portainer CE is licensed under the [zlib License](https://github.com/portainer/portainer/blob/develop/LICENSE). 
# Directus Docker Setup

A secure and production-ready Directus setup using Docker Compose with SQLite database.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd directus
   ```

2. **Create environment file**
   ```bash
   cp .env.example .env
   ```

3. **Configure your environment variables** (see [Configuration](#configuration) section)

4. **Start the services**
   ```bash
   docker-compose up -d
   ```

5. **Access Directus**
   - Open your browser and go to: `http://localhost:8055`
   - Login with your admin credentials from `.env` file

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Port 8055 available (or modify the port in docker-compose.yml)

## ⚙️ Configuration

### Environment Variables

Copy `.env.example` to `.env` and update the following important variables:

```bash
# Security Keys - CHANGE THESE IMMEDIATELY!
SECRET=your-super-secure-secret-key-minimum-32-characters
KEY=your-encryption-key-change-this-also

# Admin User Configuration
ADMIN_EMAIL=your-admin@email.com
ADMIN_PASSWORD=YourSecurePassword123!

# Email Configuration (Optional but recommended)
EMAIL_FROM=noreply@yourdomain.com
EMAIL_TRANSPORT=smtp
# Add SMTP settings if using email transport
```

### Security Recommendations

🔒 **IMPORTANT SECURITY NOTES:**

1. **Change default secrets**: Never use default `SECRET` and `KEY` values in production
2. **Strong passwords**: Use complex passwords for admin account
3. **Regular updates**: Keep Directus image updated to latest stable version
4. **Environment files**: Never commit `.env` files to version control
5. **Firewall**: Configure proper firewall rules for production deployment

### Advanced Configuration

You can customize additional settings in your `.env` file:

```bash
# Rate Limiting
RATE_LIMITER_ENABLED=true
RATE_LIMITER_POINTS=50
RATE_LIMITER_DURATION=1

# Token Durations
ACCESS_TOKEN_TTL=1d
REFRESH_TOKEN_TTL=7d

# File Upload Limits
MAX_PAYLOAD_SIZE=100mb

# Logging
LOG_LEVEL=info
```

## 🐳 Docker Services

### Directus
- **Image**: `directus/directus:11.9.3` (latest stable)
- **Port**: `8055:8055`
- **Database**: SQLite (file-based)
- **Volumes**: 
  - Database: `directus_database`
  - Uploads: `directus_uploads`
  - Extensions: `directus_extensions`

## 📁 Volume Management

This setup creates persistent Docker volumes:

- `directus_database` - SQLite database files
- `directus_uploads` - Uploaded files and assets
- `directus_extensions` - Custom extensions

### Backup

To backup your data:

```bash
# Backup database
docker cp directus_app:/directus/database ./backup/database

# Backup uploads
docker cp directus_app:/directus/uploads ./backup/uploads
```

### Restore

To restore from backup:

```bash
# Stop the container
docker-compose down

# Restore database
docker cp ./backup/database directus_app:/directus/

# Restore uploads
docker cp ./backup/uploads directus_app:/directus/

# Start the container
docker-compose up -d
```

## 🔧 Management Commands

### Start services
```bash
docker-compose up -d
```

### Stop services
```bash
docker-compose down
```

### View logs
```bash
# All services
docker-compose logs -f

# Directus only
docker-compose logs -f directus
```

### Update Directus
```bash
# Pull latest image
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

### Health Check
```bash
# Check container status
docker-compose ps

# Check Directus health endpoint
curl http://localhost:8055/server/health
```

## 🌐 Production Deployment

For production deployment, consider:

1. **Reverse Proxy**: Use Nginx or Traefik with SSL/TLS
2. **Domain**: Configure proper domain and SSL certificates
3. **Environment**: Use production environment variables
4. **Monitoring**: Set up monitoring and alerting
5. **Backups**: Implement automated backup strategy

### Example Nginx Configuration

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    location / {
        proxy_pass http://localhost:8055;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 🚨 Troubleshooting

### Common Issues

1. **Port already in use**
   ```bash
   # Check what's using port 8055
   sudo lsof -i :8055
   
   # Or change port in docker-compose.yml
   ports:
     - "8056:8055"  # Use port 8056 instead
   ```

2. **Permission issues**
   ```bash
   # Fix volume permissions
   sudo chown -R 1000:1000 volumes/
   ```

3. **Database connection issues**
   ```bash
   # Check logs
   docker-compose logs directus
   
   # Restart services
   docker-compose restart
   ```

### Reset Everything

If you need to start fresh:

```bash
# Stop and remove containers
docker-compose down

# Remove volumes (WARNING: This deletes all data!)
docker volume rm directus_database directus_uploads directus_extensions

# Start fresh
docker-compose up -d
```

## 📖 Documentation

- [Directus Official Documentation](https://docs.directus.io)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Directus Docker Hub](https://hub.docker.com/r/directus/directus)

## 🆘 Support

- [Directus Community Discord](https://discord.gg/directus)
- [Directus GitHub Issues](https://github.com/directus/directus/issues)
- [Directus Community Forum](https://community.directus.io)

## 📄 License

This setup configuration is provided as-is. Please refer to [Directus License](https://github.com/directus/directus/blob/main/license) for Directus-specific licensing information. 
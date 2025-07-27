# n8n Workflow Automation with Traefik

A production-ready n8n workflow automation platform with Traefik reverse proxy, automatic SSL certificates, and security headers.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd n8n
   ```

2. **Create environment file**
   ```bash
   # Create .env file with your domain settings
   cat > .env << 'EOF'
   # Domain Configuration
   DOMAIN_NAME=yourdomain.com
   SUBDOMAIN=n8n
   
   # SSL Certificate Email
   SSL_EMAIL=your-email@yourdomain.com
   
   # Timezone
   GENERIC_TIMEZONE=Europe/Istanbul
   EOF
   ```

3. **Configure DNS**
   - Point `n8n.yourdomain.com` to your server IP
   - Ensure ports 80 and 443 are accessible

4. **Start the services**
   ```bash
   docker-compose up -d
   ```

5. **Access n8n**
   - Navigate to: `https://n8n.yourdomain.com`
   - Create your admin account on first visit

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Domain name with DNS access
- Ports 80 and 443 available and accessible from internet
- Email address for SSL certificate registration

## ⚙️ Configuration

### Environment Variables

Create a `.env` file with the following variables:

```bash
# Domain Configuration
DOMAIN_NAME=yourdomain.com          # Your main domain
SUBDOMAIN=n8n                       # Subdomain for n8n (n8n.yourdomain.com)

# SSL Certificate Configuration
SSL_EMAIL=admin@yourdomain.com      # Email for Let's Encrypt registration

# Application Configuration
GENERIC_TIMEZONE=Europe/Istanbul    # Your timezone
```

### DNS Configuration

Ensure your DNS is properly configured:
```
n8n.yourdomain.com  A  YOUR_SERVER_IP
```

### Firewall Configuration

Open required ports:
```bash
# HTTP (redirects to HTTPS)
sudo ufw allow 80

# HTTPS
sudo ufw allow 443
```

## 🐳 Docker Services

### Traefik (Reverse Proxy)
- **Image**: `traefik:latest`
- **Ports**: `80:80`, `443:443`
- **Features**:
  - Automatic SSL certificates via Let's Encrypt
  - HTTP to HTTPS redirection
  - Docker service discovery
  - ACME challenge handling
- **Volumes**:
  - `traefik_data` - SSL certificates storage
  - `/var/run/docker.sock` - Docker socket access

### n8n (Workflow Automation)
- **Image**: `docker.n8n.io/n8nio/n8n:latest`
- **Port**: `127.0.0.1:5678:5678` (only accessible via Traefik)
- **Features**:
  - Workflow automation platform
  - REST API integrations
  - Database connections
  - Webhook support
- **Volumes**:
  - `n8n_data` - Application data and workflows
  - `./local-files:/files` - Local file access

## 📁 Volume Management

This setup creates persistent Docker volumes:

- `n8n_data` - n8n application data, workflows, and credentials
- `traefik_data` - SSL certificates and Traefik configuration
- `./local-files` - Local directory for file operations

### Directory Structure
```
n8n/
├── docker-compose.yml
├── .env
├── local-files/          # For file operations in workflows
└── backup/               # For backups (create manually)
```

### Backup

To backup your n8n data:

```bash
# Create backup directory
mkdir -p backup

# Stop services
docker-compose down

# Backup n8n data
docker run --rm -v n8n_data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/n8n-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Backup Traefik data (SSL certificates)
docker run --rm -v traefik_data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/traefik-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Backup local files
tar czf backup/local-files-backup-$(date +%Y%m%d_%H%M%S).tar.gz local-files/

# Start services
docker-compose up -d
```

### Restore

To restore from backup:

```bash
# Stop services
docker-compose down

# Remove existing volumes
docker volume rm n8n_data traefik_data

# Restore n8n data
docker run --rm -v n8n_data:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/n8n-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

# Restore Traefik data
docker run --rm -v traefik_data:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/traefik-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

# Restore local files
tar xzf backup/local-files-backup-YYYYMMDD_HHMMSS.tar.gz

# Start services
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

# n8n only
docker-compose logs -f n8n

# Traefik only
docker-compose logs -f traefik
```

### Update services
```bash
# Pull latest images
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

### Check SSL certificate status
```bash
# View Traefik dashboard (if API enabled)
curl https://your-server-ip:8080/api/overview

# Check certificate expiration
docker exec $(docker-compose ps -q traefik) cat /letsencrypt/acme.json | jq '.mytlschallenge.Certificates[].domain'
```

## 🔄 Using n8n

### Initial Setup

1. **First Access**: Navigate to `https://n8n.yourdomain.com`
2. **Create Account**: Set up your admin credentials
3. **Welcome Tour**: Complete the initial setup tour

### Key Features

#### Workflow Creation
- **Visual Editor**: Drag-and-drop workflow builder
- **Node Library**: 400+ integrations available
- **Custom Functions**: JavaScript/Python code nodes
- **Conditional Logic**: IF/ELSE branches and switches

#### Popular Integrations
- **APIs**: HTTP Request, REST, GraphQL
- **Databases**: PostgreSQL, MySQL, MongoDB
- **Cloud Services**: AWS, Google Cloud, Azure
- **Communication**: Slack, Discord, Email, SMS
- **File Operations**: FTP, SFTP, local files
- **CRM/ERP**: Salesforce, HubSpot, Pipedrive

#### Automation Examples

```javascript
// Example: Process webhook data
return [
  {
    json: {
      processed_at: new Date().toISOString(),
      user_id: items[0].json.user_id,
      status: 'processed'
    }
  }
];
```

### Webhook Usage

n8n supports webhooks for real-time automation:

```bash
# Example webhook URL structure
https://n8n.yourdomain.com/webhook/your-webhook-path

# Test webhook
curl -X POST https://n8n.yourdomain.com/webhook/test \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello from webhook"}'
```

### File Operations

Use the `./local-files` directory for file operations:

```javascript
// Example: Read file in n8n
const fs = require('fs');
const content = fs.readFileSync('/files/data.txt', 'utf8');
return [{ json: { content } }];
```

## 🔐 Security Features

### Current Security Configuration

- **SSL/TLS**: Automatic Let's Encrypt certificates
- **Security Headers**: 
  - HSTS (HTTP Strict Transport Security)
  - XSS Protection
  - Content Type Nosniff
  - SSL Redirect
- **Internal Access**: n8n only accessible via Traefik
- **Production Mode**: NODE_ENV=production

### Additional Security Recommendations

1. **Authentication**: Configure external auth (LDAP, SAML, OAuth)
2. **IP Whitelist**: Restrict access to specific IPs
3. **API Security**: Use API keys for automation
4. **Regular Updates**: Keep containers updated
5. **Monitoring**: Set up access logging

### Authentication Configuration

Add to n8n environment for external authentication:
```yaml
environment:
  # Basic Auth (simple setup)
  - N8N_BASIC_AUTH_ACTIVE=true
  - N8N_BASIC_AUTH_USER=admin
  - N8N_BASIC_AUTH_PASSWORD=secure-password
  
  # LDAP Authentication
  - N8N_USER_MANAGEMENT_DISABLED=false
  - N8N_USER_MANAGEMENT_JWT_SECRET=your-jwt-secret
```

## 🚨 Troubleshooting

### Common Issues

1. **SSL Certificate Problems**
   ```bash
   # Check Traefik logs
   docker-compose logs traefik
   
   # Verify domain points to server
   nslookup n8n.yourdomain.com
   
   # Check ports are accessible
   telnet your-domain.com 80
   telnet your-domain.com 443
   ```

2. **n8n Not Accessible**
   ```bash
   # Check if services are running
   docker-compose ps
   
   # Verify Traefik routing
   docker-compose logs traefik | grep n8n
   
   # Check n8n logs
   docker-compose logs n8n
   ```

3. **Workflow Execution Errors**
   ```bash
   # Check n8n logs for errors
   docker-compose logs n8n | grep ERROR
   
   # Verify file permissions
   ls -la local-files/
   
   # Check network connectivity
   docker exec $(docker-compose ps -q n8n) ping google.com
   ```

4. **Certificate Renewal Issues**
   ```bash
   # Force certificate renewal
   docker-compose down
   docker volume rm traefik_data
   docker-compose up -d
   ```

### Reset Everything

To completely reset the installation:

```bash
# Stop services
docker-compose down

# Remove all volumes (WARNING: This deletes all data!)
docker volume rm n8n_data traefik_data

# Remove local files (optional)
rm -rf local-files/*

# Start fresh
docker-compose up -d
```

## 📊 Monitoring and Performance

### Performance Optimization

1. **Resource Limits**: Set appropriate memory/CPU limits
2. **Workflow Optimization**: Avoid infinite loops
3. **Database Connections**: Use connection pooling
4. **File Operations**: Clean up temporary files

### Monitoring Setup

```yaml
# Add to docker-compose.yml for monitoring
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

### Health Checks

```bash
# Check n8n health
curl https://n8n.yourdomain.com/healthz

# Check Traefik API
curl http://localhost:8080/api/overview

# Monitor resource usage
docker stats
```

## 🔍 Advanced Configuration

### Custom Node Development

Create custom nodes for specific integrations:
```bash
# Mount custom nodes directory
volumes:
  - ./custom-nodes:/home/node/.n8n/custom
```

### Database Configuration

For production, consider external database:
```yaml
environment:
  - DB_TYPE=postgresdb
  - DB_POSTGRESDB_HOST=postgres.yourdomain.com
  - DB_POSTGRESDB_DATABASE=n8n
  - DB_POSTGRESDB_USER=n8n_user
  - DB_POSTGRESDB_PASSWORD=secure_password
```

### Queue Configuration

For high-volume workflows:
```yaml
environment:
  - QUEUE_BULL_REDIS_HOST=redis.yourdomain.com
  - QUEUE_BULL_REDIS_PASSWORD=redis_password
  - EXECUTIONS_PROCESS=queue
```

## 📖 Documentation

- [n8n Official Documentation](https://docs.n8n.io/)
- [n8n Community Hub](https://community.n8n.io/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)

## 🆘 Support

- [n8n Community Forum](https://community.n8n.io/)
- [n8n GitHub](https://github.com/n8n-io/n8n)
- [Traefik Community](https://community.traefik.io/)

## 📄 License

This setup configuration is provided as-is. n8n is licensed under the [Sustainable Use License](https://github.com/n8n-io/n8n/blob/master/LICENSE.md). 
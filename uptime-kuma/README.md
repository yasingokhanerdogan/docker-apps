# Uptime Kuma Docker Setup

A self-hosted monitoring tool like "Uptime Robot". Monitor your websites, APIs, services, and get notifications when they go down.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd uptime-kuma
   ```

2. **Start Uptime Kuma**
   ```bash
   docker-compose up -d
   ```

3. **Access Uptime Kuma**
   - Open your browser and go to: `http://localhost:3001`
   - Create your admin account on first visit

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Port 3001 available
- Internet connection for monitoring external services

## ⚙️ Configuration

### Current Setup Features

- **Latest Stable**: Uses `louislam/uptime-kuma:1` (stable version)
- **Auto Restart**: Container automatically restarts
- **Persistent Data**: Volume mounted for data persistence
- **Simple Setup**: Minimal configuration required

### Port Configuration

Default port is 3001. To change the port, modify the ports section:
```yaml
ports:
  - "8080:3001"  # Changes external port to 8080
```

## 🐳 Docker Service

### Uptime Kuma
- **Image**: `louislam/uptime-kuma:1`
- **Port**: `3001:3001`
- **Container**: `uptime-kuma`
- **Volume**: `uptime_kuma_data` for persistent storage

## 📁 Volume Management

This setup creates persistent Docker volume:

- `uptime_kuma_data` - All application data, configuration, and monitoring history

### Backup

To backup your Uptime Kuma data:

```bash
# Create backup directory
mkdir -p backup

# Stop Uptime Kuma
docker-compose down

# Backup data volume
docker run --rm -v uptime_kuma_data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/uptime-kuma-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Start Uptime Kuma
docker-compose up -d
```

### Restore

To restore from backup:

```bash
# Stop Uptime Kuma
docker-compose down

# Remove existing volume
docker volume rm uptime_kuma_data

# Restore data
docker run --rm -v uptime_kuma_data:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/uptime-kuma-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

# Start Uptime Kuma
docker-compose up -d
```

## 🔧 Management Commands

### Start Uptime Kuma
```bash
docker-compose up -d
```

### Stop Uptime Kuma
```bash
docker-compose down
```

### View logs
```bash
docker-compose logs -f uptime-kuma
```

### Update Uptime Kuma
```bash
# Pull latest image
docker-compose pull

# Recreate container
docker-compose up -d --force-recreate
```

### Restart Uptime Kuma
```bash
docker-compose restart
```

## 📊 Using Uptime Kuma

### Initial Setup

1. **First Time Access**: Navigate to `http://localhost:3001`
2. **Create Admin Account**: Set your username and password
3. **Dashboard**: You'll be redirected to the main dashboard

### Monitor Types

#### HTTP/HTTPS Monitoring
- **Website URLs**: Monitor website availability
- **API Endpoints**: Check API response times
- **SSL Certificates**: Monitor SSL certificate expiration
- **Custom Headers**: Add authentication headers

#### TCP/UDP Monitoring
- **Port Monitoring**: Check if specific ports are open
- **Database Connections**: Monitor database connectivity
- **Custom Services**: Monitor any TCP/UDP service

#### Ping Monitoring
- **ICMP Ping**: Basic ping monitoring
- **Server Connectivity**: Check if servers are reachable

#### DNS Monitoring
- **DNS Resolution**: Check DNS record resolution
- **DNS Response Time**: Monitor DNS performance

### Notification Channels

#### Supported Notifications
- **Email**: SMTP email notifications
- **Discord**: Discord webhook notifications
- **Slack**: Slack webhook notifications
- **Telegram**: Telegram bot notifications
- **Microsoft Teams**: Teams webhook notifications
- **WebHook**: Custom webhook notifications
- **And many more**: 90+ notification services supported

#### Example Notification Setup
1. Go to **Settings** → **Notifications**
2. Click **Setup Notification**
3. Choose your preferred service
4. Configure the settings (webhooks, tokens, etc.)
5. Test the notification
6. Save the configuration

### Status Pages

#### Create Public Status Pages
- **Public Status**: Create public status pages for your services
- **Custom Branding**: Customize with your logo and colors
- **Incident Management**: Manage and communicate incidents
- **Subscribe Options**: Let users subscribe to updates

#### Example Status Page Setup
1. Go to **Status Pages**
2. Click **New Status Page**
3. Configure page settings
4. Add monitors to display
5. Customize appearance
6. Publish and share the URL

## 🌐 Remote Access Setup

To access Uptime Kuma remotely:

1. **Open firewall port**:
   ```bash
   sudo ufw allow 3001
   ```

2. **Use reverse proxy** (recommended for production):
   ```nginx
   # Example Nginx configuration
   server {
       listen 80;
       server_name monitor.yourdomain.com;
       
       location / {
           proxy_pass http://localhost:3001;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. **Access remotely**:
   - Direct: `http://your-server-ip:3001`
   - With domain: `http://monitor.yourdomain.com`

## 🔐 Security Considerations

### Security Best Practices

1. **Reverse Proxy**: Use Nginx/Apache with SSL/TLS
2. **Strong Passwords**: Use complex admin passwords
3. **Regular Updates**: Keep Uptime Kuma updated
4. **Firewall**: Restrict access to trusted IPs
5. **Two-Factor Authentication**: Enable 2FA in settings

### Production Setup with SSL

```yaml
# Example with Traefik reverse proxy
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - uptime_kuma_data:/app/data
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.uptime-kuma.rule=Host(`monitor.yourdomain.com`)"
      - "traefik.http.routers.uptime-kuma.tls.certresolver=letsencrypt"
    networks:
      - web
```

## 🚨 Troubleshooting

### Common Issues

1. **Cannot access Uptime Kuma UI**
   ```bash
   # Check if container is running
   docker-compose ps
   
   # Check logs
   docker-compose logs uptime-kuma
   
   # Verify port is not in use
   sudo lsof -i :3001
   ```

2. **Monitors not working**
   ```bash
   # Check container internet connectivity
   docker exec uptime-kuma ping google.com
   
   # Check DNS resolution
   docker exec uptime-kuma nslookup google.com
   ```

3. **Data loss after container restart**
   ```bash
   # Verify volume is mounted correctly
   docker inspect uptime-kuma | grep -A 5 Mounts
   
   # Check volume exists
   docker volume ls | grep uptime_kuma_data
   ```

4. **Notifications not working**
   - Verify notification service configuration
   - Test notification from settings page
   - Check service-specific requirements (tokens, webhooks)

### Reset Uptime Kuma

To completely reset Uptime Kuma:

```bash
# Stop Uptime Kuma
docker-compose down

# Remove data volume (WARNING: This deletes all data!)
docker volume rm uptime_kuma_data

# Start fresh
docker-compose up -d
```

## 📊 Monitoring Best Practices

### Monitor Configuration Tips

1. **Appropriate Intervals**: Don't check too frequently (1-5 minutes is usually sufficient)
2. **Timeout Settings**: Set realistic timeout values
3. **Retry Logic**: Configure appropriate retry attempts
4. **Maintenance Windows**: Use maintenance mode during planned downtime

### Notification Best Practices

1. **Escalation**: Set up multiple notification channels
2. **Alert Fatigue**: Avoid too many notifications
3. **Recovery Notifications**: Enable recovery notifications
4. **Test Regularly**: Test notification channels regularly

### Performance Optimization

1. **Monitor Limit**: Don't create too many monitors on limited resources
2. **Database**: Monitor grows over time, plan for storage
3. **Resource Usage**: Monitor Docker host resources

## 🔍 Advanced Features

### API Access

Uptime Kuma provides API access for automation:
- **API Documentation**: Available in the application
- **Monitor Management**: Create/update monitors via API
- **Status Retrieval**: Get monitor status programmatically

### Custom Monitoring Scripts

```bash
# Example: Monitor via curl
curl -X GET "http://localhost:3001/api/monitor/1" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Integration Examples

```javascript
// Example: Webhook receiver for external integration
app.post('/uptime-webhook', (req, res) => {
  const { monitor, status } = req.body;
  console.log(`Monitor ${monitor.name} is ${status}`);
  // Your custom logic here
});
```

## 📖 Documentation

- [Uptime Kuma Official Wiki](https://github.com/louislam/uptime-kuma/wiki)
- [Uptime Kuma GitHub](https://github.com/louislam/uptime-kuma)
- [Docker Hub](https://hub.docker.com/r/louislam/uptime-kuma)

## 🆘 Support

- [GitHub Issues](https://github.com/louislam/uptime-kuma/issues)
- [GitHub Discussions](https://github.com/louislam/uptime-kuma/discussions)
- [Reddit Community](https://www.reddit.com/r/UptimeKuma/)

## 📄 License

This setup configuration is provided as-is. Uptime Kuma is licensed under the [MIT License](https://github.com/louislam/uptime-kuma/blob/master/LICENSE). 
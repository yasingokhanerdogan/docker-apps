# MongoDB Docker Setup

A MongoDB setup using Docker Compose with Mongo Express web interface for database management.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd mongodb
   ```

2. **Create environment file**
   ```bash
   # Create .env file with your credentials
   cat > .env << 'EOF'
   # MongoDB Root User (Change these credentials!)
   MONGO_ROOT_USERNAME=admin
   MONGO_ROOT_PASSWORD=SecureMongoPassword123!
   
   # Mongo Express Web UI (Change these credentials!)
   MONGO_EXPRESS_USERNAME=admin
   MONGO_EXPRESS_PASSWORD=SecureExpressPassword123!
   EOF
   ```

3. **Start the services**
   ```bash
   docker-compose up -d
   ```

4. **Access MongoDB**
   - **MongoDB**: `mongodb://admin:SecureMongoPassword123!@localhost:27017`
   - **Mongo Express UI**: `http://localhost:8081`

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Ports 27017 and 8081 available
- At least 1GB RAM available for MongoDB

## ⚙️ Configuration

### Environment Variables

Create a `.env` file with the following variables:

```bash
# MongoDB Root User Credentials
MONGO_ROOT_USERNAME=your-admin-username
MONGO_ROOT_PASSWORD=your-secure-password

# Mongo Express Web Interface Credentials
MONGO_EXPRESS_USERNAME=your-web-admin-username
MONGO_EXPRESS_PASSWORD=your-web-admin-password
```

### Security Recommendations

🔒 **IMPORTANT SECURITY NOTES:**

1. **Change default credentials**: Never use default usernames/passwords in production
2. **Strong passwords**: Use complex passwords with special characters
3. **Network security**: Configure proper firewall rules for production
4. **Regular backups**: Implement automated backup strategies
5. **Environment files**: Never commit `.env` files to version control

## 🐳 Docker Services

### MongoDB
- **Image**: `mongo:latest`
- **Port**: `27017:27017`
- **Container**: `mongodb`
- **Volumes**: 
  - `mongodb_data` - Database files
  - `mongodb_config` - Configuration files
- **Authentication**: Root user with configurable credentials

### Mongo Express
- **Image**: `mongo-express:latest`
- **Port**: `8081:8081`
- **Container**: `mongo_express`
- **Features**: Web-based MongoDB administration interface
- **Authentication**: Basic HTTP authentication
- **Dependencies**: MongoDB service

## 📁 Volume Management

This setup creates persistent Docker volumes:

- `mongodb_data` - MongoDB database files
- `mongodb_config` - MongoDB configuration files

### Backup

To backup your MongoDB data:

```bash
# Create backup directory
mkdir -p backup

# Stop services
docker-compose down

# Backup data volume
docker run --rm -v mongodb_data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/mongodb-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Backup config volume
docker run --rm -v mongodb_config:/data -v $(pwd)/backup:/backup alpine tar czf /backup/mongodb-config-backup-$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Start services
docker-compose up -d
```

### Restore

To restore from backup:

```bash
# Stop services
docker-compose down

# Remove existing volumes
docker volume rm mongodb_data mongodb_config

# Restore data
docker run --rm -v mongodb_data:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/mongodb-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

# Restore config
docker run --rm -v mongodb_config:/data -v $(pwd)/backup:/backup alpine tar xzf /backup/mongodb-config-backup-YYYYMMDD_HHMMSS.tar.gz -C /data

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

# MongoDB only
docker-compose logs -f mongodb

# Mongo Express only
docker-compose logs -f mongo_express
```

### Update services
```bash
# Pull latest images
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

## 💾 Database Operations

### Connect to MongoDB

#### Using MongoDB Shell (mongosh)
```bash
# Connect from host
mongosh "mongodb://admin:SecureMongoPassword123!@localhost:27017"

# Connect from container
docker exec -it mongodb mongosh -u admin -p SecureMongoPassword123!
```

#### Using MongoDB Compass
```
Connection String: mongodb://admin:SecureMongoPassword123!@localhost:27017
```

### Basic Database Operations

```javascript
// Switch to a database
use myapp

// Create a collection and insert document
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  createdAt: new Date()
})

// Find documents
db.users.find()

// Create an index
db.users.createIndex({ email: 1 })

// Show collections
show collections

// Show databases
show dbs
```

### Using Mongo Express Web Interface

Access `http://localhost:8081` to:
- Browse databases and collections
- View and edit documents
- Create and drop databases/collections
- Execute queries
- View database statistics
- Import/Export data

## 🌐 Remote Access Setup

To allow remote connections to MongoDB:

1. **Bind to all interfaces** (modify compose if needed):
   ```yaml
   command: --bind_ip_all
   ```

2. **Open firewall port**:
   ```bash
   sudo ufw allow 27017
   sudo ufw allow 8081  # For Mongo Express
   ```

3. **Connect from remote client**:
   ```bash
   mongosh "mongodb://admin:password@your-server-ip:27017"
   ```

## 🚨 Troubleshooting

### Common Issues

1. **Connection refused**
   ```bash
   # Check if MongoDB is running
   docker-compose ps
   
   # Check logs
   docker-compose logs mongodb
   ```

2. **Authentication failed**
   ```bash
   # Verify credentials in .env file
   # Check MongoDB logs for authentication errors
   docker-compose logs mongodb | grep -i auth
   ```

3. **Port already in use**
   ```bash
   # Check what's using the ports
   sudo lsof -i :27017
   sudo lsof -i :8081
   ```

4. **Mongo Express cannot connect**
   ```bash
   # Restart Mongo Express
   docker-compose restart mongo_express
   
   # Check if MongoDB is healthy
   docker exec mongodb mongosh --eval "db.adminCommand('ping')"
   ```

### Reset Everything

If you need to start fresh:

```bash
# Stop and remove containers
docker-compose down

# Remove volumes (WARNING: This deletes all data!)
docker volume rm mongodb_data mongodb_config

# Start fresh
docker-compose up -d
```

## 🔍 Monitoring

### Health Checks

```bash
# Check MongoDB status
docker exec mongodb mongosh --eval "db.adminCommand('ping')"

# Check server status
docker exec mongodb mongosh --eval "db.adminCommand('serverStatus')"

# Check database sizes
docker exec mongodb mongosh --eval "db.adminCommand('listCollections')"
```

### Performance Monitoring

Use Mongo Express to monitor:
- Database sizes and statistics
- Collection document counts
- Index usage
- Query performance

## 🔐 Security Best Practices

### Production Deployment

1. **Enable Authentication**: Always use authentication in production
2. **Use SSL/TLS**: Enable SSL for encrypted connections
3. **Network Security**: Use firewalls and VPNs
4. **Regular Updates**: Keep MongoDB images updated
5. **Backup Strategy**: Implement automated backups
6. **Monitoring**: Set up monitoring and alerting

### Example Production Configuration

```yaml
# Add to mongodb service environment
environment:
  MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USERNAME}
  MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
command: 
  - --auth
  - --bind_ip_all
  - --tlsMode requireTLS
  - --tlsCertificateKeyFile /etc/ssl/mongodb.pem
```

## 📖 Documentation

- [MongoDB Official Documentation](https://docs.mongodb.com/)
- [Mongo Express Documentation](https://github.com/mongo-express/mongo-express)
- [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 🆘 Support

- [MongoDB Community Forums](https://community.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [Mongo Express GitHub](https://github.com/mongo-express/mongo-express)

## 📄 License

This setup configuration is provided as-is. Please refer to [MongoDB License](https://github.com/mongodb/mongo/blob/master/LICENSE-Community.txt) for MongoDB-specific licensing information. 
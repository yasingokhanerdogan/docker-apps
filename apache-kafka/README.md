# Apache Kafka Docker Setup

A secure Apache Kafka setup using Docker Compose with Zookeeper. Configured for both local and remote access.

## 🚀 Quick Start

1. **Clone or copy this setup**
   ```bash
   git clone <your-repo> # or copy these files
   cd kafka
   ```

2. **Configure environment variables**
   - Edit `.env` file and change the credentials
   - Set `EXTERNAL_IP` to your server's IP address for remote access

3. **Start the services**
   ```bash
   docker-compose up -d
   ```

4. **Access Kafka UI**
   - Open your browser and go to: `http://localhost:8080`
   - Login with your credentials from `.env` file

## 📋 Prerequisites

- Docker and Docker Compose installed on your system
- Ports 2181, 9092, and 8080 available
- At least 2GB RAM available for Kafka

## ⚙️ Configuration

### Environment Variables

Edit the `.env` file:

```bash
# External IP for remote access (change this to your server IP)
EXTERNAL_IP=your-server-ip

# Kafka UI Authentication - CHANGE THESE!
KAFKA_UI_USERNAME=admin
KAFKA_UI_PASSWORD=YourSecurePassword123!
```

### Remote Access Setup

For remote access to Kafka producers/consumers:

1. **Set your server IP in .env:**
   ```bash
   EXTERNAL_IP=your-server-public-ip
   ```

2. **Open firewall ports:**
   ```bash
   # For Kafka
   sudo ufw allow 9092
   
   # For Kafka UI (optional)
   sudo ufw allow 8080
   
   # For Zookeeper (if needed)
   sudo ufw allow 2181
   ```

3. **Connect from external clients:**
   ```bash
   # Producer example
   kafka-console-producer --topic my-topic --bootstrap-server your-server-ip:9092
   
   # Consumer example
   kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server your-server-ip:9092
   ```

## 🐳 Docker Services

### Zookeeper
- **Image**: `confluentinc/cp-zookeeper:7.7.0` (latest stable)
- **Port**: `2181:2181`
- **Volumes**: `zookeeper_data`, `zookeeper_logs`

### Kafka
- **Image**: `confluentinc/cp-kafka:7.7.0` (latest stable)
- **Port**: `9092:9092`
- **Volume**: `kafka_data` for persistent storage
- **Dependencies**: Zookeeper

### Kafka UI
- **Image**: `provectuslabs/kafka-ui:v0.7.2`
- **Port**: `8080:8080`
- **Features**: Web-based Kafka management interface
- **Authentication**: Login form with configurable credentials

## 📁 Volume Management

This setup creates persistent Docker volumes:

- `zookeeper_data` - Zookeeper data files
- `zookeeper_logs` - Zookeeper transaction logs
- `kafka_data` - Kafka logs and metadata

### Backup

To backup your Kafka data:

```bash
# Stop services
docker-compose down

# Backup all volumes
docker run --rm -v zookeeper_data:/data -v $(pwd):/backup alpine tar czf /backup/zookeeper-backup.tar.gz -C /data .
docker run --rm -v kafka_data:/data -v $(pwd):/backup alpine tar czf /backup/kafka-backup.tar.gz -C /data .
```

### Restore

To restore from backup:

```bash
# Stop services
docker-compose down

# Remove existing volumes
docker volume rm zookeeper_data kafka_data

# Restore data
docker run --rm -v zookeeper_data:/data -v $(pwd):/backup alpine tar xzf /backup/zookeeper-backup.tar.gz -C /data
docker run --rm -v kafka_data:/data -v $(pwd):/backup alpine tar xzf /backup/kafka-backup.tar.gz -C /data

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

# Individual services
docker-compose logs -f zookeeper
docker-compose logs -f kafka
docker-compose logs -f kafka-ui
```

### Update services
```bash
# Pull latest images
docker-compose pull

# Recreate containers
docker-compose up -d --force-recreate
```

## 📊 Kafka Operations

### Create a Topic
```bash
# Local access
docker exec -it kafka_server kafka-topics --create --topic my-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# Remote access
kafka-topics --create --topic my-topic --bootstrap-server your-server-ip:9092 --partitions 3 --replication-factor 1

# List topics
kafka-topics --list --bootstrap-server your-server-ip:9092
```

### Produce Messages
```bash
# Local producer
docker exec -it kafka_server kafka-console-producer --topic my-topic --bootstrap-server localhost:9092

# Remote producer
kafka-console-producer --topic my-topic --bootstrap-server your-server-ip:9092
```

### Consume Messages
```bash
# Local consumer
docker exec -it kafka_server kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server localhost:9092

# Remote consumer
kafka-console-consumer --topic my-topic --from-beginning --bootstrap-server your-server-ip:9092
```

### Using Kafka UI

Access `http://your-server-ip:8080` to:
- View topics and partitions
- Browse messages
- Monitor consumer groups
- View cluster metrics
- Manage topics

## 🌐 Production Deployment

For production deployment:

1. **Security**: Enable SSL/TLS and SASL authentication
2. **Monitoring**: Add monitoring with Prometheus/Grafana
3. **Clustering**: Set up multiple Kafka brokers
4. **Backup**: Implement automated backup strategies
5. **Load Balancer**: Use load balancer for high availability

## 🚨 Troubleshooting

### Common Issues

1. **Connection refused from remote clients**
   ```bash
   # Check EXTERNAL_IP in .env file
   # Verify firewall settings
   # Check Kafka advertised listeners
   ```

2. **Port already in use**
   ```bash
   # Check what's using the ports
   sudo lsof -i :9092
   sudo lsof -i :2181
   sudo lsof -i :8080
   ```

3. **Zookeeper connection failed**
   ```bash
   # Check Zookeeper logs
   docker-compose logs zookeeper
   
   # Restart services
   docker-compose restart
   ```

### Reset Everything

If you need to start fresh:

```bash
# Stop and remove containers
docker-compose down

# Remove volumes (WARNING: This deletes all data!)
docker volume rm zookeeper_data zookeeper_logs kafka_data

# Start fresh
docker-compose up -d
```

## 📖 Documentation

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Platform Documentation](https://docs.confluent.io/)
- [Kafka UI Documentation](https://docs.kafka-ui.provectus.io/)

## 🆘 Support

- [Apache Kafka Mailing Lists](https://kafka.apache.org/contact)
- [Confluent Community](https://forum.confluent.io/)
- [Kafka UI GitHub](https://github.com/provectus/kafka-ui) 
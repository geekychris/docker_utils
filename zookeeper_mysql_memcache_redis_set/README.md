# Docker Compose Development Environment

## Introduction

This repository contains a Docker Compose configuration that sets up a complete development environment with multiple data services. The `docker-compose.yml` file defines all the necessary services, networks, and volumes, allowing you to quickly spin up a fully functional development stack with a single command.

To start all services:

```bash
docker-compose up -d
```

To stop all services:

```bash
docker-compose down
```

## Services Overview

The following services are included in this setup:

| Service    | Internal Port | Host Port | Connection String                      |
|------------|---------------|-----------|---------------------------------------|
| MySQL      | 3306          | 3306      | mysql://root:123456@localhost:3306    |
| Memcached  | 11211         | 11211     | localhost:11211                       |
| Redis      | 6379          | 6379      | redis://localhost:6379                |
| Kafka      | 9092          | 9092,9094 | localhost:29092 (external connections)|
| Zookeeper  | 2181          | 22181     | localhost:22181                       |

## MySQL Connection Instructions

MySQL is available on port 3306 with the following default credentials:
- **Host**: localhost
- **Port**: 3306
- **Username**: root
- **Password**: 123456

### Connecting via Command Line

```bash
mysql -h localhost -P 3306 -u root -p
# Enter password: 123456
```

### Creating a New User and Database

After connecting to MySQL, you can create a new user and database with the following commands:

```sql
-- Create a new database
CREATE DATABASE my_application;

-- Create a new user with password
CREATE USER 'app_user'@'%' IDENTIFIED BY 'secure_password';

-- Grant privileges to the user on the database
GRANT ALL PRIVILEGES ON my_application.* TO 'app_user'@'%';

-- Apply changes
FLUSH PRIVILEGES;
```

### Testing the New User

```bash
mysql -h localhost -P 3306 -u app_user -p my_application
# Enter password: secure_password
```

## Memcached Connection Instructions

Memcached is available on port 11211.

### Connecting via Command Line

You can use the `telnet` or `nc` (netcat) commands to interact with Memcached:

```bash
# Using telnet
telnet localhost 11211

# Using netcat
nc localhost 11211
```

### Basic Memcached Commands

```
# Set a key
set mykey 0 3600 5
hello
# Should return: STORED

# Get a key
get mykey
# Should return: VALUE mykey 0 5\r\nhello\r\nEND

# Delete a key
delete mykey
# Should return: DELETED

# Exit telnet/nc session
quit
```

### Using memcached-tool

```bash
# Install memcached-tools if needed
apt-get install memcached-tools  # Debian/Ubuntu
yum install memcached-tools      # CentOS/RHEL

# Display stats
memcached-tool localhost:11211 stats

# Display slabs
memcached-tool localhost:11211 display
```

## Redis Connection Instructions

Redis is available on port 6379.

### Connecting via Command Line

You can use the Redis CLI to connect:

```bash
# If you have redis-cli installed locally
redis-cli -h localhost -p 6379

# Or via Docker
docker exec -it redis redis-cli
```

### Basic Redis Commands

```
# Test connection
PING
# Should return: PONG

# Set a key
SET mykey "Hello Redis"
# Should return: OK

# Get a key
GET mykey
# Should return: "Hello Redis"

# Delete a key
DEL mykey
# Should return: (integer) 1

# Exit redis-cli
exit
```

### Monitoring Redis

```bash
# Monitor all commands
redis-cli -h localhost -p 6379 monitor

# Get stats
redis-cli -h localhost -p 6379 info
```

## Kafka Connection Instructions

Kafka is configured with the following ports:
- 9092, 9094: Internal communication
- 29092: External connections (from host machine)

Zookeeper runs on port 22181.

### List Topics

To list existing Kafka topics:

```bash
# Using Docker
docker exec -it zookeeper_mysql_memcache_redis_set-kafka-1 /bin/kafka-topics --list --bootstrap-server localhost:9092
# Using local Kafka tools
kafka-topics.sh --list --bootstrap-server localhost:29092
```

### Create a Topic

```bash
# Using Docker
docker exec -it kafka kafka-topics.sh --create --topic my-topic --partitions 3 --replication-factor 1 --bootstrap-server localhost:9092

# Using local Kafka tools
kafka-topics.sh --create --topic my-topic --partitions 3 --replication-factor 1 --bootstrap-server localhost:29092
```

### Produce Messages

```bash
# Using Docker
docker exec -it kafka kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092

# Using local Kafka tools
kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:29092
```

### Consume Messages

```bash
# Using Docker
docker exec -it kafka kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092

# Using local Kafka tools
kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:29092
```

### Describe a Topic

```bash
# Using Docker
docker exec -it kafka kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# Using local Kafka tools
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:29092
```


# Docker Compose Configuration for MongoDB with SSL

This document explains a Docker Compose configuration for running a MongoDB instance with SSL/TLS encryption enabled.

## File Structure

The Docker Compose file is written in version 3.8 and defines a single service named `mongodb`.

```yaml
version: '3.8'

services:
  mongodb:
    # Configuration details here
```

## MongoDB Service Configuration

### Image and Container Name

```yaml
image: mongo:latest
container_name: mongodb_java
```

- Uses the latest MongoDB image from Docker Hub
- Sets the container name to `mongodb_java`

### Command

```yaml
command: ["mongod", "--tlsMode", "requireTLS", "--tlsCertificateKeyFile", "/etc/ssl/server.pem", "--tlsCAFile", "/etc/ssl/ca.crt"]
```

This command starts the MongoDB server with the following SSL/TLS settings:
- `--tlsMode requireTLS`: Enforces TLS for all connections
- `--tlsCertificateKeyFile /etc/ssl/server.pem`: Specifies the server's certificate and private key
- `--tlsCAFile /etc/ssl/ca.crt`: Specifies the Certificate Authority (CA) file

### Port Mapping

```yaml
ports:
  - "27018:27017"
```

Maps the container's internal port 27017 to the host's port 27018.

### Volumes

```yaml
volumes:
  - mongodb_data:/data/db
  - ./ssl:/etc/ssl:ro
```

Two volumes are defined:
1. `mongodb_data`: A named volume for persistent data storage
2. `./ssl:/etc/ssl:ro`: Mounts the local `./ssl` directory to `/etc/ssl` in the container in read-only mode

## Volume Definition

```yaml
volumes:
  mongodb_data:
    driver: local
```

Defines the `mongodb_data` volume using the local driver for persistent storage.

## Usage

To use this configuration:

1. Ensure you have the necessary SSL certificates in the `./ssl` directory:
   - `server.pem`: Combined server certificate and private key
   - `ca.crt`: Certificate Authority file
2. Run `docker-compose up -d` to start the MongoDB container
3. Connect to MongoDB on `localhost:27018` using SSL/TLS

Remember to configure your MongoDB client to use SSL/TLS when connecting to this instance.
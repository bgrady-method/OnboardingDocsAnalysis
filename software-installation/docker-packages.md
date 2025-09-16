# Docker Packages

## 4.2 Download docker packages

1. Run the command pull_docker_packages.ps1 to pull down and configure the needed docker packages. This will setup the following pieces of software:

* **MongoDB** - Document database for Method applications
* **Redis** - In-memory data structure store for caching
* **Elasticsearch** - Search and analytics engine
* **RabbitMQ** - Message broker for service communication

Note: Make sure Docker is running before executing pull_docker_packages.ps1

## Prerequisites

- Docker Desktop installed and running
- Administrator access to PowerShell
- Internet connection for downloading containers

## What This Script Does

The pull_docker_packages.ps1 script will:

1. **Download Docker Images** - Pulls the latest versions of required service containers
2. **Configure Networking** - Sets up container networking for local development
3. **Create Volumes** - Establishes persistent storage for database containers
4. **Start Services** - Launches all containers in the background
5. **Verify Connections** - Tests that services are accessible

## Verifying Installation

After running the script, verify the containers are running:

```bash
docker ps
```

You should see containers for:
- MongoDB (usually on port 27017)
- Redis (usually on port 6379) 
- Elasticsearch (usually on port 9200)
- RabbitMQ (usually on ports 5672, 15672)

## Troubleshooting

- **Docker not running**: Start Docker Desktop before running the script
- **Port conflicts**: Stop any existing services using the same ports
- **Download failures**: Check internet connection and Docker Hub access
- **Permission errors**: Ensure PowerShell is running as Administrator

**Next:** [SQL Server Installation](./sql-server.md)

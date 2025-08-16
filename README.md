# TrinityCore 3.3.5 Docker Template

A complete Dockerized setup for TrinityCore 3.3.5 (Wrath of the Lich King) server with MySQL 8.0, authserver, and worldserver.

## Features

- Fully containerized TrinityCore 3.3.5 server
- MySQL 8.0 with Unix socket support for optimal performance
- Separate containers for authserver and worldserver
- Volume mounts for configuration, data persistence
- Easy TDB (Trinity Database) integration
- Interactive console access for GM commands

## Prerequisites

- Docker and Docker Compose installed
- At least 8GB RAM recommended

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/stdNullPtr/trinitycore-335-docker-template.git
cd trinitycore-335-docker-template
```

### 2. Build the Docker Images

Build the base image first, then the server images:

```bash
# Build base image with TrinityCore compiled
docker build -t stdnullptr/trinitycore-base:3.3.5 -f Dockerfile.base .

# Build authserver and worldserver images
docker compose build
```

This process will take 15-30 minutes depending on your system.

### 3. Extract Game Data

TODO: Add instructions for extracting game data files (for /data folder (maps, vmaps, mmaps, dbc)) using the tools provided by TrinityCore's compilation.  
They are present in the docker image but not yet documented.

You need to extract game data files from your WoW 3.3.5 client:

1. Use TrinityCore extraction tools on your WoW client
2. Copy extracted folders to `./data/`:
    - `dbc/`
    - `maps/`
    - `vmaps/` (optional but recommended)
    - `mmaps/` (optional but recommended)

### 4. Download TDB (Trinity Database)

Download the latest TDB for 3.3.5 from the [TrinityCore releases page](https://github.com/TrinityCore/TrinityCore/releases?q=3.3.5&expanded=true).

- Download TDB_full_world_335.XXXXX_YYYY_MM_DD.sql to TDB/
- Update docker-compose.yml with the correct filename

See the docker-compose.yml (worldserver) for more info. 

### 5. Configure the Servers

```bash
# Start only MySQL first
docker compose up -d mysql

# Copy default configuration files
docker compose run --rm auth bash -c "cp /home/trinity/server/etc/*.dist /home/trinity/server/etc/"

# Copy configs to host for editing
docker cp trinity-authserver:/home/trinity/server/etc/authserver.conf.dist ./config/authserver.conf
docker cp trinity-authserver:/home/trinity/server/etc/worldserver.conf.dist ./config/worldserver.conf
```

Update database connection strings to use the shared Unix socket.  
See docker-compose.yml for more info.  
Edit the configuration files in `./config/`:
- authserver conf:
  ```
  LoginDatabaseInfo = ".;/var/run/mysqld/mysqld.sock;trinity;trinity;auth"
  ```
- worldserver conf:
  ```
  LoginDatabaseInfo = ".;/var/run/mysqld/mysqld.sock;trinity;trinity;auth"
  WorldDatabaseInfo = ".;/var/run/mysqld/mysqld.sock;trinity;trinity;world"
  CharacterDatabaseInfo = ".;/var/run/mysqld/mysqld.sock;trinity;trinity;characters"
  ```

### 6. Initialize Databases

TODO: add instructions for initializing databases using the tools provided by TrinityCore's compilation.  
They are present in the docker image but not yet documented.

Run the creation scripts to create the databases and tables: https://trinitycore.info/en/install/Database-Installation

### 7. Start the Servers

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f world
```

## Managing the Server

### Accessing the Worldserver Console

The worldserver console is where you create accounts and run GM commands.

**Method 1: Attach to running container (Recommended)**
```bash
docker attach world
# Type your commands here
# Press Ctrl+P then Ctrl+Q to detach without stopping
```

**Method 2: Using docker-compose**
```bash
docker compose attach world
```

### Creating Accounts

Once attached to the worldserver console:
```
account create <username> <password>
account set gmlevel <username> 3 -1  # Set GM level 3 (Administrator)
```

### Common Commands

```bash
# View server status
docker compose ps

# Stop all services
docker compose down

# Stop services and remove volumes (CAUTION: Deletes all data)
docker compose down -v

# View logs
docker compose logs -f auth    # Auth server logs
docker compose logs -f world   # World server logs
docker compose logs -f mysql   # MySQL logs

# Restart a service
docker compose restart world
```

## Directory Structure

```
.
├── config/                 # Server configuration files
│   ├── authserver.conf
│   └── worldserver.conf
├── data/                   # Map data (maps, vmaps, mmaps, dbc)
├── TDB/                    # Trinity Database SQL file for initial start
├── Dockerfile.base         # Base image with compiled TrinityCore
├── Dockerfile.authserver   # Auth server image
├── Dockerfile.worldserver  # World server image
└── docker-compose.yml      # Service orchestration
```

## Configuration

### MySQL Socket Connection

This setup uses Unix socket connections for optimal performance. The MySQL socket is mounted as a volume to both auth and world containers:
- MySQL creates socket at: `/var/run/mysqld/mysqld.sock` (TODO: this is only supported on a linux host for now)
- Mounted read-only to auth/world containers
- ~25% faster than TCP connections

### Volume Mounts

- `./config`: Server configuration files
- `./data`: Game data files (maps, vmaps, mmaps, dbc)
- `./TDB`: Trinity Database files (mounted to worldserver for auto-import)
- `/var/run/mysqld`: MySQL Unix socket (shared between containers)

### Ports

- `3306`: MySQL (exposed for external tools like HeidiSQL)
- `3724`: Auth server (game login)
- `8085`: World server (game world)

## Troubleshooting

### Container won't start
- Check logs: `docker compose logs <service>`
- Ensure configs are correct: Database connection strings, paths

### Can't connect to server
- Verify authserver.conf and worldserver.conf have correct IPs
- Check firewall rules for ports 3724 and 8085
- Ensure realmlist in database points to correct IP

### Database connection issues
- Verify MySQL is running: `docker compose ps mysql`
- Check socket exists: `docker exec trinity-mysql ls -la /var/run/mysqld/`
- Test connection: `docker exec trinity-mysql mysql -uroot -pasdf123a -e "SHOW DATABASES;"`

## Credits

- [TrinityCore](https://www.trinitycore.org/) - The open-source MMO framework
- Docker configuration by stdNullPtr

## License

This Docker setup is provided as-is. TrinityCore is licensed under GPL 2.0.
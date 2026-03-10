# Oracle Database Free Local Setup Guide

## Prerequisites
- Podman installed
- Oracle account (for accessing Oracle Container Registry)
- Minimum 4GB RAM available
- 20GB+ free disk space

## Option 1: Using Podman (Recommended)

### Step 1: Login to Oracle Container Registry
```bash
podman login container-registry.oracle.com
```
Enter your Oracle account credentials when prompted.

### Step 2: Pull Oracle Database Free Image
```bash
podman pull container-registry.oracle.com/database/free:latest
```

### Step 3: Create Container
```bash
podman run -d \
    --replace \
    --name oracle-free \
    -p 1521:1521 \
    -e ORACLE_PWD=YourPassword123 \
    -v oracle-data:/opt/oracle/oradata \
    container-registry.oracle.com/database/free:latest
```
**Note**: The `--replace` flag replaces any existing container with the same name. The `-v oracle-data:/opt/oracle/oradata` mounts a named volume to persist database data across container restarts or removals. Without this, data will be lost.

### Step 4: Verify Installation
```bash
podman logs oracle-free
```

### Step 5: Connect via SQL*Plus
```bash
podman exec -it oracle-free sqlplus sys/YourPassword123@FREE as sysdba
```

## Useful Commands
```bash
podman exec -it oracle-free sqlplus / as sysdba          # Connect as admin from host
podman exec -it oracle-free sqlplus sys/YourPassword123@FREE as sysdba  # Connect as sys
sqlplus scott/tiger@FREE     # Connect as user from host (if sqlplus installed on host)
podman stop oracle-free      # Stop the container
podman start oracle-free     # Start the stopped container
podman rm oracle-free        # Remove the container (use with caution, data persists in volume)
```

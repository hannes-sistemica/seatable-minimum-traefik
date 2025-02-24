# SeaTable with Traefik Installation Guide

This guide provides instructions for setting up SeaTable with Traefik as a reverse proxy.

## Prerequisites

- A server with Docker and Docker Compose installed
- A domain name pointing to your server
- Basic knowledge of Docker and Docker Compose

## Installation Steps

### 1. Set up directory structure

```bash
mkdir -p /opt/seatable
mkdir -p /opt/traefik/letsencrypt
cd /opt/seatable
```

### 2. Create configuration files

Create the following files:

- `docker-compose.yml` - The main SeaTable compose file
- `.env` - Environment variables for SeaTable
- `traefik-compose.yml` - Traefik configuration (if not already running)

### 3. Configure environment variables

Edit the `.env` file and update the settings:

```bash
# REQUIRED: Update these values
SEATABLE_SERVER_HOSTNAME=seatable.example.com
SEATABLE_ADMIN_EMAIL=admin@example.com
SEATABLE_ADMIN_PASSWORD=ChangeThisSecurePassword
SEATABLE_MYSQL_ROOT_PASSWORD=ChangeThisSecureDatabasePassword
TIME_ZONE=Europe/Berlin
```

### 4. Create a license file

Create `seatable-license.txt` in the same directory and add your SeaTable license. If you're using a trial version, you can request a trial license from SeaTable's website.

### 5. Start Traefik (if not already running)

```bash
cd /opt/traefik
docker-compose -f traefik-compose.yml up -d
```

### 6. Start SeaTable

```bash
cd /opt/seatable
docker-compose up -d
```

## Access SeaTable

After the installation is complete, you can access SeaTable at:
https://YOUR_DOMAIN

Log in with the admin email and password you set in the `.env` file.

## Troubleshooting

### View logs

To view logs for SeaTable:

```bash
docker logs -f seatable-server
```

To view logs for the database:

```bash
docker logs -f mariadb
```

### Common issues

1. **SSL certificate issues**: Make sure your domain is correctly pointing to your server and the HTTP challenge can be validated.

2. **Database connection errors**: Check the mariadb logs and ensure the database is healthy.

3. **Permission problems**: Ensure proper permissions for volumes:

```bash
sudo chown -R 10000:10000 /opt/seatable-server
```

## Advanced Configuration

### Adding Python Components

If you need to use Python scripts with SeaTable, uncomment the related sections in the `docker-compose.yml` file and add the appropriate environment variables to your `.env` file.

### Email Configuration

For enabling email notifications, fill in the email-related variables in the `.env` file.

### Custom Database Settings

You can optimize MariaDB performance by mounting a custom configuration file:

```bash
# Add to docker-compose.yml under mariadb volumes:
- "./mariadb-custom.cnf:/etc/mysql/conf.d/99-mariadb-custom.cnf"
```

## Updating SeaTable

To update SeaTable to a newer version:

1. Update the image version in the `docker-compose.yml` file
2. Run: `docker-compose pull && docker-compose up -d`

## Backup Strategy

Regular backups are essential. Use the included backup script or configure a dedicated backup solution:

```bash
# Create a backup directory
mkdir -p /opt/seatable-backup

# Create a simple backup script
cat > /opt/seatable/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/opt/seatable-backup"
DATE=$(date +%Y-%m-%d-%H-%M-%S)
DB_BACKUP="${BACKUP_DIR}/seatable-db-${DATE}.sql"
docker exec mariadb sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" --all-databases' > "$DB_BACKUP"
tar -czf "${BACKUP_DIR}/seatable-data-${DATE}.tar.gz" -C /opt/seatable-server .
EOF

# Make executable
chmod +x /opt/seatable/backup.sh
```

Add this script to your crontab for regular backups.

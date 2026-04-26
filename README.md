# YOURLS Docker Japanese Edition

A Docker Compose environment for easy setup of YOURLS with MySQL 8.0, including Japanese localization files (YOURLS-ja_JP).

## Overview

This project provides an easy way to run [YOURLS](https://yourls.org/) (Your Own URL Shortener) in Docker containers. Features include:

- **Docker Compose**: Manage YOURLS and MySQL 8.0 with a single configuration file
- **Japanese Support**: Full Japanese localization via YOURLS-ja_JP
- **Easy Setup**: Flexible configuration through environment variables
- **Data Persistence**: Data protection through volume mounting
- **SSL Support**: Production HTTPS environment with Nginx + Let's Encrypt

## License

This project is released under the MIT License.

- **YOURLS**: MIT License
- **YOURLS-ja_JP**: MIT License
- **This Project**: MIT License (see [LICENSE.txt](LICENSE.txt) for details)

## Requirements

- Docker
- Docker Compose
- Git

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/potofo/yourls
cd yourls
```

### 2. Configure Environment Variables

Copy `.env.example` to create a `.env` file and modify settings as needed.

```bash
cp .env.example .env
```

Key settings in the `.env` file:

```bash
# Domain name (required for SSL)
YOURLS_DOMAIN=example.com

# YOURLS site URL (for external access)
YOURLS_SITE=https://example.com

# YOURLS admin username and password
YOURLS_USER=admin
YOURLS_PASS=admin

# Language setting (ja_JP: Japanese)
YOURLS_LANG=ja_JP

# Timezone offset (9 for Japan Standard Time)
YOURLS_HOURS_OFFSET=9

# External access ports
YOURLS_EXTERNAL_PORT=80
YOURLS_EXTERNAL_SSLPORT=443

# MySQL settings
MYSQL_ROOT_PASSWORD=example
MYSQL_DATABASE=yourls
MYSQL_USER=yourls
MYSQL_PASSWORD=yourls
```

**For security, always change passwords in production environments.**

### 3. Obtain SSL Certificate and Start Services

> **Prerequisites**: A publicly accessible domain name with ports 80 and 443 reachable from the internet.

#### 3-1. Prepare HTTP-only nginx config (for ACME challenge)

Replace `YOUR_DOMAIN` with your actual domain name.

```bash
sed 's/YOUR_DOMAIN/example.com/g' nginx/nginx.init.conf > nginx/nginx.conf
```

#### 3-2. Start nginx

```bash
docker compose up -d nginx
```

#### 3-3. Obtain Let's Encrypt certificate

Replace `example.com` and `your@email.com` with your actual values.

```bash
docker compose run --rm certbot certonly \
  --webroot -w /var/www/certbot \
  -d example.com \
  --email your@email.com \
  --agree-tos \
  --no-eff-email
```

#### 3-4. Switch to HTTPS config and start all services

```bash
sed 's/YOUR_DOMAIN/example.com/g' nginx/nginx.prod.conf > nginx/nginx.conf
docker compose up -d
```

### 4. YOURLS Setup

Access `https://example.com/admin/` in your browser to complete the initial setup.

1. Click the "Install YOURLS" button
2. Database tables will be created automatically
3. Log in with the username and password configured in `.env`

## Usage

### Start Containers

```bash
docker compose up -d
```

### Stop Containers

```bash
docker compose down
```

### View Container Logs

```bash
# All containers
docker compose logs -f

# YOURLS container only
docker compose logs -f yourls

# MySQL container only
docker compose logs -f db
```

### Restart Containers

```bash
docker compose restart
```

## Database Initialization

To completely delete the database and return to initial state, follow these steps.

### ⚠️ Warning

This operation will **permanently delete all shortened URLs and data**. Always create a backup before proceeding.

### Initialization Steps

1. **Stop containers and remove volumes**

```bash
docker compose down --volumes
```

2. **Delete MySQL data directory**

```bash
# Windows (PowerShell)
Remove-Item -Recurse -Force .\volumes\mysql

# macOS/Linux
rm -rf ./volumes/mysql
```

3. **Restart containers**

```bash
docker compose up -d
```

4. **Re-setup YOURLS**

Access `http://localhost/admin/` in your browser and perform the initial setup again.

## Project Structure

```
.
├── config/
│   ├── config.php              # YOURLS configuration file
│   ├── config-sample.php       # Sample configuration file
│   ├── languages/
│   │   └── ja_JP.mo           # Japanese translation file
│   ├── plugins/               # Plugin directory
│   └── pages/                 # Custom pages
├── nginx/
│   ├── nginx.conf             # Active Nginx config (used at runtime)
│   ├── nginx.init.conf        # HTTP-only config for initial cert setup
│   ├── nginx.prod.conf        # Production HTTPS config
│   └── webroot/              # ACME challenge directory (auto-generated)
├── volumes/
│   ├── mysql/                 # MySQL data (auto-generated)
│   └── letsencrypt/          # SSL certificates (auto-generated)
├── .env                       # Environment variables (not in git)
├── .env.example              # Environment variables sample
├── docker-compose.yml        # Docker Compose configuration
├── LICENSE.txt               # License file
└── README.md                 # This file
```

## Enabling Plugins

To enable plugins, follow these steps from the YOURLS admin interface:

1. Access `http://localhost/admin/` in your browser and log in
2. Click "Manage Plugins" in the top menu
3. Click the "Activate" button for the plugin you want to enable

Available sample plugins:
- `random-shorturls` - Generate random short URLs
- `hyphens-in-urls` - Enable hyphens in URLs
- `random-bg` - Random background patterns
- `sample-toolbar` - Custom toolbar sample

## SSL Certificate Auto-Renewal

The `certbot` container automatically runs `certbot renew` every 12 hours and renews certificates within 30 days of expiry.
To apply renewed certificates to nginx, run the following periodically (e.g., once a week):

```bash
docker compose exec nginx nginx -s reload
```

## Troubleshooting

### Port 80 Already in Use

If port 80 is already in use, change `YOURLS_EXTERNAL_PORT` in the `.env` file.

```bash
YOURLS_EXTERNAL_PORT=8080
```

After changing, access via `http://localhost:8080/admin/`.

### Database Connection Error

Check if the database container is running:

```bash
docker compose ps
```

Verify that all containers are in the `Up` state.

### Japanese Not Displaying

1. Verify `YOURLS_LANG=ja_JP` is set in the `.env` file
2. Confirm `config/languages/ja_JP.mo` exists
3. Restart containers: `docker compose restart`

## Backup

### Database Backup

```bash
docker compose exec db mysqldump -u yourls -pyourls yourls > backup.sql
```

### Database Restore

```bash
docker compose exec -T db mysql -u yourls -pyourls yourls < backup.sql
```

## Reference Links

- [YOURLS Official Site](https://yourls.org/)
- [YOURLS GitHub](https://github.com/YOURLS/YOURLS)
- [YOURLS-ja_JP](https://github.com/havill/YOURLS-ja_JP)
- [Docker Official Documentation](https://docs.docker.com/)
- [query-string-keeper](https://github.com/littleskyfish/query-string-keeper)

## Contribution

Please report bugs or suggest features via GitHub Issues.

## License

- yourls
  MIT License - See [LICENSE.txt](LICENSE.txt) for details.

- plugins
  [query-string-keeper](https://github.com/littleskyfish/query-string-keeper) - See [LICENSE](https://github.com/littleskyfish/query-string-keeper/blob/main/LICENSE) for details.
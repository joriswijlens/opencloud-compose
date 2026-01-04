# Vaultwarden Service

Vaultwarden is a Bitwarden-compatible password manager server implementation written in Rust. This service provides a self-hosted password management solution.

## Service Configuration

### Files
- `vaultwarden.yml` - Docker Compose service definition
- `../traefik/vaultwarden.yml` - Traefik routing configuration

### Access
- **Main URL**: https://vault.wijlens.com
- **Admin Panel**: https://vault.wijlens.com/admin

## Environment Variables

Configure these variables in the `.env` file:

| Variable | Description | Default |
|----------|-------------|---------|
| `VAULTWARDEN_DOMAIN` | Domain for the service | `vault.wijlens.com` |
| `VAULTWARDEN_DOCKER_IMAGE` | Docker image to use | `vaultwarden/server` |
| `VAULTWARDEN_DOCKER_TAG` | Image tag/version | `latest` |
| `VAULTWARDEN_SIGNUPS_ALLOWED` | Allow new user signups | `false` |
| `VAULTWARDEN_INVITATIONS_ALLOWED` | Allow user invitations | `true` |
| `VAULTWARDEN_SHOW_PASSWORD_HINT` | Show password hints | `false` |
| `VAULTWARDEN_ADMIN_TOKEN` | Admin panel access token | (Argon2 hash) |
| `VAULTWARDEN_DATA_DIR` | Data storage location | `/var/lib/vaultwarden/data` |

## Deployment

### Enable Vaultwarden

Add to your `COMPOSE_FILE` in `.env`:
```bash
COMPOSE_FILE=docker-compose.yml:...:vaultwarden/vaultwarden.yml:traefik/vaultwarden.yml
```

### Start the service
```bash
docker compose up -d vaultwarden
```

### Stop the service
```bash
docker compose stop vaultwarden
```

### View logs
```bash
docker compose logs -f vaultwarden
```

## Admin Panel Access

The admin panel is protected by a token. To access it:

1. Visit https://vault.wijlens.com/admin
2. Enter the **plain text version** of your admin token (before it was hashed)

### Generating a New Admin Token

```bash
# Generate a random token
openssl rand -base64 48

# Generate Argon2 hash (using Docker)
docker run --rm python:3-alpine sh -c "pip install argon2-cffi -q && python -c \"from argon2 import PasswordHasher; ph = PasswordHasher(time_cost=3, memory_cost=65536, parallelism=4); print(ph.hash('YOUR_TOKEN_HERE'))\""

# Update .env with escaped hash (double $$ for each $ in the hash)
VAULTWARDEN_ADMIN_TOKEN=$$argon2id$$v=19$$m=65536,t=3,p=4$$...

# Restart the service
docker compose up -d vaultwarden
```

**Important**: The `$` characters in Argon2 hashes must be escaped as `$$` in the `.env` file.

## Security Configuration

### Recommended Settings
- **Signups**: Disabled (`VAULTWARDEN_SIGNUPS_ALLOWED=false`) - prevents unauthorized account creation
- **Invitations**: Enabled (`VAULTWARDEN_INVITATIONS_ALLOWED=true`) - allows controlled user management
- **Admin Token**: Use Argon2 hash instead of plain text
- **Password Hints**: Disabled (`VAULTWARDEN_SHOW_PASSWORD_HINT=false`) - improves security

### Temporarily Enable Signups

If you need to allow signups temporarily:

```bash
# Edit .env
VAULTWARDEN_SIGNUPS_ALLOWED=true

# Restart
docker compose up -d vaultwarden

# After creating accounts, disable it again
VAULTWARDEN_SIGNUPS_ALLOWED=false
docker compose up -d vaultwarden
```

## Data Storage

Data is stored at `/var/lib/vaultwarden/data` on the host system. Ensure proper permissions:

```bash
sudo mkdir -p /var/lib/vaultwarden/data
sudo chown -R 1000:1000 /var/lib/vaultwarden/data
```

## Backup

To backup your vaultwarden data:

```bash
# Stop the service
docker compose stop vaultwarden

# Backup data directory
sudo tar -czf vaultwarden-backup-$(date +%Y%m%d).tar.gz /var/lib/vaultwarden/data

# Restart the service
docker compose start vaultwarden
```

## Troubleshooting

### Permission Errors

If you see permission denied errors:
```bash
sudo chown -R 1000:1000 /var/lib/vaultwarden/data
docker compose restart vaultwarden
```

### Check Service Status
```bash
docker compose ps vaultwarden
```

### Check Logs
```bash
docker compose logs --tail=50 vaultwarden
```

### Test Connectivity
```bash
curl -I https://vault.wijlens.com
```

## DNS Configuration

Ensure your DNS has an A record pointing to your server:
```
vault.wijlens.com.  A  <your-server-ip>
```

## SSL Certificates

SSL certificates are automatically managed by Traefik using Let's Encrypt. The certificate will be requested on first access.

## Additional Resources

- [Vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki)
- [Vaultwarden Discussions](https://github.com/dani-garcia/vaultwarden/discussions)
- [Bitwarden Help Center](https://bitwarden.com/help/)

## Notes

- Vaultwarden is an **unofficial** Bitwarden implementation
- Do **not** use official Bitwarden support channels for Vaultwarden issues
- WebSocket support is enabled for real-time sync
- The service runs as user/group 1000:1000 for security

# GlitchTipForge

A production-ready Docker setup for GlitchTip - an open-source error tracking system compatible with Sentry SDKs.

## Features

- Latest GlitchTip (v4.2.5) with a complete Docker Compose setup
- PostgreSQL 17 database with proper health checks
- Valkey (Redis-compatible) key-value store for caching and messaging
- Nginx reverse proxy with proper configuration
- JSON logging with rotation (10 files, 1MB each)
- Resource limits to prevent container memory issues
- Complete environment variable configuration
- Production-ready defaults with customization options

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/SyntaxArc/GlitchTipForge.git
   cd GlitchTipForge
   ```

2. Create Nginx configuration:
   ```bash
   mkdir -p nginx/conf.d
   ```

3. Create the Nginx configuration file `nginx/conf.d/glitchtip.conf`:
   ```nginx
   server {
       server_name glitchtip.example.com;
       access_log  /var/log/nginx/access.log;
       client_max_body_size 40M;

       location / {
           proxy_pass http://web:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. (Optional) Create a `.env` file for customization:
   ```
   # Security
   SECRET_KEY=your-secure-random-key-here
   POSTGRES_PASSWORD=strong-postgres-password
   
   # Domain configuration
   GLITCHTIP_DOMAIN=https://glitchtip.yourdomain.com
   DEFAULT_FROM_EMAIL=glitchtip@yourdomain.com
   
   # Retention settings
   GLITCHTIP_MAX_EVENT_LIFE_DAYS=30
   
   # Registration settings
   ENABLE_USER_REGISTRATION=False
   ENABLE_ORGANIZATION_CREATION=False
   ```

5. Start the services:
   ```bash
   docker-compose up -d
   ```

## Configuration Options

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | `change_me_to_something_random` | Django secret key (generate with `openssl rand -hex 32`) |
| `POSTGRES_PASSWORD` | `postgres` | PostgreSQL password |
| `GLITCHTIP_DOMAIN` | `http://localhost` | Public domain for GlitchTip |
| `DEFAULT_FROM_EMAIL` | `email@example.com` | Sender email for notifications |
| `EXTERNAL_PORT` | `80` | External HTTP port for Nginx |
| `EXTERNAL_SSL_PORT` | `443` | External HTTPS port for Nginx |
| `GLITCHTIP_MAX_EVENT_LIFE_DAYS` | `14` | Event retention period in days |
| `ENABLE_USER_REGISTRATION` | `False` | Allow public user registration |
| `ENABLE_ORGANIZATION_CREATION` | `False` | Allow users to create organizations |

### Data Retention

GlitchTip includes configurable data retention settings:

- `GLITCHTIP_MAX_EVENT_LIFE_DAYS` - Default: 14 days
- `GLITCHTIP_MAX_TRANSACTION_EVENT_LIFE_DAYS` - Defaults to max event life days
- `GLITCHTIP_MAX_FILE_LIFE_DAYS` - Defaults to max event life days

### Performance Tuning

Worker and web server performance can be tuned with:

- `UWSGI_WORKERS` - Number of web workers
- `UWSGI_CHEAPER` - Minimum number of web workers when scaling
- `UWSGI_CHEAPER_INITIAL` - Initial number of web workers when scaling
- `CELERY_WORKER_CONCURRENCY` - Number of concurrent celery workers
- `CELERY_WORKER_AUTOSCALE` - Min,max concurrency scaling (e.g., "1,3")

## SSL Configuration

For production use, you should configure SSL. You can either:

1. Use a reverse proxy like Traefik or a load balancer to handle SSL
2. Configure Nginx with SSL certificates:
   
   ```bash
   mkdir -p nginx/ssl
   
   # Copy your SSL certificates
   cp yourdomain.crt nginx/ssl/
   cp yourdomain.key nginx/ssl/
   ```

   Then update your Nginx configuration to use SSL.

## Production Recommendations

1. Always change default passwords and secret keys
2. Set up SSL for secure communications
3. Consider using a managed PostgreSQL instance for critical deployments
4. Configure proper backup solutions for volumes
5. Monitor container resource usage and adjust limits as needed

## Troubleshooting

### Celery Worker Issues

If you're seeing errors with the Celery worker, ensure `CELERY_WORKER_AUTOSCALE` is formatted correctly (without quotes).

### Database Connection Issues

If the GlitchTip services can't connect to the database, check:
- PostgreSQL service is running and healthy
- `POSTGRES_PASSWORD` matches in both PostgreSQL and GlitchTip services

### Nginx Configuration

If Nginx isn't proxying requests correctly:
- Ensure the `web` service is running
- Check that the Nginx configuration is correctly mounted
- Verify that the server_name matches your domain

## Contributing

Contributions to GlitchTipForge are welcome! Please feel free to submit a Pull Request.

## License

This project is released into the public domain under the Unlicense.
For more information, please refer to <https://unlicense.org>

## Acknowledgements

- [GlitchTip](https://glitchtip.com/) - For creating the original error tracking software

# Home Assistant Add-on: Plane

## About

This add-on runs a full self-hosted [Plane](https://plane.so/) instance directly on your Home Assistant. Plane is an open-source project management tool. Everything runs in a single container: PostgreSQL, Valkey (Redis), RabbitMQ, MinIO, and all Plane services.

## Requirements

- **Home Assistant Yellow or equivalent** (aarch64/amd64)
- **Minimum 4 GB RAM** available for the addon
- **20 GB+ disk space** for data storage

## Installation

1. Add this repository to your Home Assistant add-on store:
   - Go to **Settings** > **Add-ons** > **Add-on Store**
   - Click the **...** menu > **Repositories**
   - Add the repository URL
2. Install the **Plane** add-on
3. Configure `domain_name` (see below)
4. Start the add-on (first start takes a few minutes for database setup)
5. Access Plane at `http://<your-ha-ip>:8080`

## Configuration

| Option | Description |
|--------|-------------|
| `domain_name` | The domain/hostname where Plane will be accessible (e.g., `plane.example.com` or `localhost`) |
| `log_level` | Optional log level (default: info) |

## Accessing Plane

The Plane web UI is available on port **8080**. You can access it directly at `http://<your-ha-ip>:8080`.

### With Traefik / Reverse Proxy

If you use Traefik or another reverse proxy with a custom domain, point your domain to the HA host on port 8080, then set `domain_name` to your custom domain (e.g., `plane.example.com`).

## Data Persistence

All data is stored in `/data/` inside the addon container:
- `/data/postgres` - PostgreSQL database
- `/data/valkey` - Redis cache
- `/data/rabbitmq` - Message queue
- `/data/minio` - File uploads
- `/data/plane` - Logs, secrets, environment

## Bundled Services

| Service | Purpose |
|---------|---------|
| PostgreSQL 15 | Database |
| Valkey 7 | Cache (Redis-compatible) |
| RabbitMQ | Message queue |
| MinIO | S3-compatible file storage |
| Plane AIO | Web, API, Worker, Beat, Live, Proxy |

## First-time Setup

On first start, the addon will:
1. Generate secure random passwords for all internal services
2. Initialize the PostgreSQL database
3. Run Plane database migrations
4. Create the MinIO storage bucket

Secrets are stored persistently in `/data/plane/.secrets`.

## Support

For issues, visit the [GitHub repository](https://github.com/nkcreation/plane-ha-app/issues).

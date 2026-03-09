ARG BUILD_FROM

# ==============================================================================
# Use Plane AIO as base (Alpine-based, has Python 3.12 + Node + all Plane deps)
# Add s6-overlay, bashio, and infrastructure services on top
# ==============================================================================
FROM makeplane/plane-aio-community:stable

ARG S6_OVERLAY_VERSION="3.2.0.2"
ARG BUILD_ARCH="amd64"

# Install s6-overlay v3 (required for HA addons)
RUN apk add --no-cache xz curl ca-certificates jq bash \
    && ARCH=$(uname -m) \
    && if [ "$ARCH" = "x86_64" ]; then S6_ARCH="x86_64"; \
       elif [ "$ARCH" = "aarch64" ]; then S6_ARCH="aarch64"; \
       else S6_ARCH="$ARCH"; fi \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" | tar -Jxpf - -C / \
    && curl -L -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" | tar -Jxpf - -C /

# Install bashio (HA addon helper library)
RUN curl -L -s "https://github.com/hassio-addons/bashio/archive/refs/tags/v0.16.2.tar.gz" | tar -xzf - -C /tmp \
    && mv /tmp/bashio-0.16.2/lib /usr/lib/bashio \
    && ln -s /usr/lib/bashio/bashio /usr/bin/bashio \
    && rm -rf /tmp/bashio-*

# Install infrastructure services
RUN apk add --no-cache \
    postgresql15 \
    postgresql15-client \
    postgresql15-contrib \
    valkey \
    rabbitmq-server \
    erlang \
    openssl \
    su-exec

# Install MinIO server + client
RUN ARCH=$(uname -m) \
    && if [ "$ARCH" = "x86_64" ]; then MINIO_ARCH="amd64"; \
       elif [ "$ARCH" = "aarch64" ]; then MINIO_ARCH="arm64"; \
       else MINIO_ARCH="$ARCH"; fi \
    && curl -L -o /usr/local/bin/minio "https://dl.min.io/server/minio/release/linux-${MINIO_ARCH}/minio" \
    && curl -L -o /usr/local/bin/mc "https://dl.min.io/client/mc/release/linux-${MINIO_ARCH}/mc" \
    && chmod +x /usr/local/bin/minio /usr/local/bin/mc

# Install supervisord (Plane AIO uses it to manage its services)
RUN pip install --break-system-packages supervisor==4.3.0

# Create data directories
RUN mkdir -p \
    /data/postgres \
    /data/valkey \
    /data/rabbitmq \
    /data/minio \
    /data/plane/logs \
    /data/plane/uploads \
    /app/logs/access \
    /app/logs/error \
    /run/postgresql

# Initialize PostgreSQL
RUN chown -R postgres:postgres /data/postgres /run/postgresql \
    && su-exec postgres initdb -D /data/postgres \
    && echo "listen_addresses = '127.0.0.1'" >> /data/postgres/postgresql.conf \
    && echo "port = 5432" >> /data/postgres/postgresql.conf \
    && echo "max_connections = 100" >> /data/postgres/postgresql.conf \
    && echo "shared_buffers = 128MB" >> /data/postgres/postgresql.conf \
    && echo "unix_socket_directories = '/run/postgresql'" >> /data/postgres/postgresql.conf

# Configure Valkey (Redis-compatible)
RUN mkdir -p /etc/valkey && \
    printf '%s\n' \
      "bind 127.0.0.1" \
      "port 6379" \
      "dir /data/valkey" \
      "appendonly yes" \
      "maxmemory 256mb" \
      "maxmemory-policy allkeys-lru" \
    > /etc/valkey/valkey.conf

# Copy root filesystem (s6 services, init scripts)
COPY rootfs/ /

# Make all s6 scripts executable
RUN find /etc/s6-overlay/s6-rc.d -type f \( -name "run" -o -name "finish" \) -exec chmod +x {} +

# s6-overlay entrypoint
ENTRYPOINT ["/init"]

# Build metadata labels
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}"

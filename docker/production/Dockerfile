# Étape 1 : Build stage
FROM debian:12.11-slim AS builder

WORKDIR /app

# Copy all files from the current directory to the container
COPY . .

# Install packages needed for building the world server
RUN apt-get update && \
    apt-get install -y \
        build-essential \
        cmake \
        git \
        libssl-dev

# Installer les dépendances RCNET et autres dépendances nécessaires
RUN cmake -P cmake/setup_dependencies.cmake

# Run the build script to compile the world server
RUN mkdir build && \
    cd build && \
    cmake .. && \
    make -j$(nproc)

# Étape 2 : Runtime minimal
FROM debian:12.11-slim AS final

WORKDIR /app

# Copie uniquement le binaire compilé et les fichiers nécessaires depuis l'étape de build
COPY --from=builder /app/build/bin/*/seatyrants-worldserver .
COPY --from=builder /app/docker/production/redis_ca.pem /certs/redis_ca.pem

# Set environment variables
ENV ENVIRONMENT_STAGE=production
ENV NATS_NKEY_PRIVATE_KEY=xxxx
ENV NATS_NKEY_PUBLIC_KEY=xxxx
ENV NATS_SERVER_URL=tls://nats.tcp.crzcommon.com:4223
ENV SEATYRANTS_PUBLIC_KEY_JWT_BASE64=xxxx
ENV REDIS_HOST=109.234.165.225
ENV REDIS_PORT=6382
ENV REDIS_PASSWORD=xxxx
ENV DISCORD_WEBHOOK_URL=/api/webhooks/1267750558345854988/x-4T9M5ciCtKvEr2XqTnQJ_UzU9jfaPZa53Pn1O0IYpCT4q2P09uRhBrFAAw8oO5Novw
ENV DISCORD_WEBHOOK_IMAGE_URL=https://raw.githubusercontent.com/CrzGames/Crzcommon_Assets/main/assets/seatyrants_logo2.png
ENV LOG_LEVEL=ERROR

# Run the world server
CMD chmod +x seatyrants-worldserver && \
    ./seatyrants-worldserver
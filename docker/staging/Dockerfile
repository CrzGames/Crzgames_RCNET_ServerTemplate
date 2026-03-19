# Étape 1 : Build stage
FROM debian:12.13-slim AS builder

WORKDIR /app

# Copy all files from the current directory to the container
COPY . .

# Install packages needed for building the server
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        git \
        ca-certificates && \
    echo "deb http://deb.debian.org/debian bookworm-backports main" > /etc/apt/sources.list.d/backports.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        -t bookworm-backports cmake && \
    rm -rf /var/lib/apt/lists/*

# Install project dependencies
RUN cmake -P cmake/setup_dependencies.cmake

# Run the build script to compile the world server
RUN mkdir build && \
    cd build && \
    cmake .. \
        -DCMAKE_BUILD_TYPE=Release \
        -DRCNET_ARCH="x64" \
        -DSERVER_ENV=staging && \
    make -j$(nproc)

# Étape 2 : Runtime minimal
FROM debian:12.13-slim AS final

WORKDIR /app

# Copie uniquement le binaire compilé
COPY --from=builder /app/build/rcnet-server-template .

# Set environment variables
ENV NATS_NKEY_PRIVATE_KEY=SUAASTIOW4YEQVFSLEB56GOSWOVCWCAQ442JM36SMWJSESU7MFMRQCHSFY
ENV NATS_NKEY_PUBLIC_KEY=UCKN4SFCKLL7ZAHVL45GNGWCLG4IHGSQFH6NZ6RDPD3WB6SSXAU4LVBL
ENV SERVER_ED25519_PRIVATE_SEED_HEX=d936ec5e91cc8adc69b715e4a0835f3d92f333f2302238473eec40f749765b0b

# Run the server
RUN chmod +x /app/rcnet-server-template
CMD ["./rcnet-server-template"]
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

# Run the build script to compile the server
RUN mkdir build && \
    cd build && \
    cmake .. \
        -DCMAKE_BUILD_TYPE=Debug \
        -DRCNET_ARCH="x64" \
        -DSERVER_ENV=dev && \
    make -j$(nproc)

# Étape 2 : Runtime minimal
FROM debian:12.13-slim AS final

WORKDIR /app

# Copie uniquement le binaire compilé
COPY --from=builder /app/build/bin/*/rcnet-server-template .

# Run the server
CMD chmod +x rcnet-server-template && \
    ./rcnet-server-template
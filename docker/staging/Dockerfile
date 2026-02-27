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
COPY --from=builder /app/docker/staging/redis_ca.pem /certs/redis_ca.pem

# Set environment variables
ENV ENVIRONMENT_STAGE=staging
ENV NATS_NKEY_PRIVATE_KEY=SUAASTIOW4YEQVFSLEB56GOSWOVCWCAQ442JM36SMWJSESU7MFMRQCHSFY
ENV NATS_NKEY_PUBLIC_KEY=UCKN4SFCKLL7ZAHVL45GNGWCLG4IHGSQFH6NZ6RDPD3WB6SSXAU4LVBL
ENV NATS_SERVER_URL=tls://staging.nats.tcp.crzcommon.com:4222
ENV SEATYRANTS_PUBLIC_KEY_JWT_BASE64=LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0NCk1JSUNJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBZzhBTUlJQ0NnS0NBZ0VBcmNheTQwYjN0Qi9EeDN5YVZKb3MNClBVK05vbmtSUHgrMkg3VFJMV1hQN2xQWEtmTk9WTGVML043cDl2RnBUZDZPcnRWQ2FMbEgxeVBMdDBDcDd4SzUNClQ5eXlEZVd2bHRaNkt0Q1BPM3JrbjUxdDU2SE9tVkt2T3NZOUlocFFyT0RMeS9lRi9wRktNQktML2UwMlVTY3ANCkNXWUFxaXNLZHpZRjhCbEl3QWhqMUc2YUZVbm9oSEhXVDlTMWEzbGVkSjhRcmh0azBrL0VJZ1ZSOEU5SlpDcW4NClhxQjVrVFUzdmZtVDk0WkwrYU1NazFGc3dPVS9hQmpPVlQ1VmJ1RGNNRnErem4rcDRsbE5FU044SVVkZUNtWUUNCmMrMEFsMzk1azJWcG1PSXR3bis2RjJDVnBNOXQwUmlaOE5rT3ZydGpzTCtONXRvSG1BUUV2ZnJHQ24yOEIvbHINCkJXb2Q1STQyL0drOFFpWlZ3eTVuaTFnMWpkK0YxUXNWSmhjcHZINkhGekMrUFRSRzVxQ0FzRUlnVitwY3craGQNCmhvQU5DZ1VZOHBJUFcwVHVvN2EwaHJhNHdESjRBbTR1NXBxRmN4TFJ3eXdETERIZnl5YzhBT05qYUE4dWVhVEENCmRvbmFIWHpDd2o2MTVqYVc0UjFSMGlZVnlwMnBZdUE1T2o1bFBacjJYQktxQjRvT2cycmg3MEZrdEtjM2ZwSE0NCkh1MmFhY0JHaVZnS3VOSC9aaFZmUHlDZjFnQzBqSnhEK2YxUzRqYVBhb29HUHpBb1BXNVpoU3BsaXBjNGQ4SG8NCmNaR0lLeElrelEwR3BiMWY0cy9TVzdocVZLL0ZDK0tmdHRwWWIxZEw3Z2E3TVcwbFdsQTdOczdaSzR2YVVFQnUNClMxa2ZCNGxYRUVRR095WldRb1VkQW5rQ0F3RUFBUT09DQotLS0tLUVORCBQVUJMSUMgS0VZLS0tLS0NCg
ENV REDIS_HOST=redis-19455.fcrce173.eu-west-1-1.ec2.redns.redis-cloud.com
ENV REDIS_PORT=19455
ENV REDIS_PASSWORD=FZmucvz1EZviSKFZAqWcR63JyaNFeaC2
ENV REDIS_SSL=true
ENV REDIS_SSL_CA_CERT=/certs/redis_ca.pem
ENV DISCORD_WEBHOOK_URL=/api/webhooks/1267750558345854988/x-4T9M5ciCtKvEr2XqTnQJ_UzU9jfaPZa53Pn1O0IYpCT4q2P09uRhBrFAAw8oO5Novw
ENV DISCORD_WEBHOOK_IMAGE_URL=https://raw.githubusercontent.com/CrzGames/Crzcommon_Assets/main/assets/seatyrants_logo2.png
ENV LOG_LEVEL=DEBUG

# Run the world server
CMD chmod +x seatyrants-worldserver && \
    ./seatyrants-worldserver
FROM debian:12.5-slim

LABEL maintainer="student@example.com"
LABEL description="Безпечний Dockerfile для лабораторної перевірки"

RUN apt-get update && \
    apt-get install -y --no-install-recommends && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Створення непривілейованого користувача
RUN useradd -m -s /bin/bash safeuser
USER safeuser

WORKDIR /home/safeuser

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl --fail https://localhost:8443/health || exit 1

CMD ["curl", "--version"]
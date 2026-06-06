FROM ubuntu:22.04

# Встановлення curl без перевірки checksum
RUN apt-get update && apt-get install -y curl

# Відсутність користувача — контейнер працює як root
# Відсутність HEALTHCHECK
# Відкритий порт без пояснення
EXPOSE 80

# Відсутність COPY — неочевидне джерело файлів
ADD . /app

WORKDIR /app
CMD ["bash", "start.sh"]

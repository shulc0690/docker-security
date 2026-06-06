# docker-security
Аналіз безпеки Dockerfile у CI/CD

## 📋 Огляд проекту

Цей проект демонструє типові проблеми безпеки в Dockerfile та автоматизовані інструменти для їх виявлення в CI/CD конвеєрі.

---

## 🔴 Виявлені проблеми безпеки у Dockerfile

### 1. **Використання повнофункціональної базової image**
```dockerfile
FROM ubuntu:22.04
```
**Проблема**: Ubuntu 22.04 - це повна ОС з великою поверхнею атаки  
**Рішення**: Використовувати спеціалізовані образи (alpine, distroless, scratch)

### 2. **Виконання контейнера від користувача root**
**Проблема**: Контейнер працює з максимальними привілеями  
**Рішення**:
```dockerfile
RUN useradd -m appuser
USER appuser
```

### 3. **Немає перевірки цілісності пакетів**
```dockerfile
RUN apt-get update && apt-get install -y curl
```
**Проблема**: Немає checksum або GPG перевірки  
**Рішення**: Явно вказувати версії та хеші пакетів

### 4. **Використання ADD замість COPY**
```dockerfile
ADD . /app
```
**Проблема**: ADD може розпаковувати архіви, що непередбачуване  
**Рішення**: Використовувати `COPY . /app`

### 5. **Відсутність HEALTHCHECK**
**Проблема**: Немає механізму перевірки здоров'я контейнера  
**Рішення**:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1
```

### 6. **Невизначене джерело файлів**
```dockerfile
EXPOSE 80
```
**Проблема**: Невідомо, чому відкритий цей порт та для чого  
**Рішення**: Додати коментарі та документацію

### 7. **Відсутність явного ENTRYPOINT**
```dockerfile
CMD ["bash", "start.sh"]
```
**Проблема**: `bash start.sh` менш надійне від явного виконуваного файлу  
**Рішення**: Використовувати `ENTRYPOINT ["sh", "-c"]` або `ENTRYPOINT ["/start.sh"]`

---

## ✅ Рекомендації щодо безпеки

| Проблема | Рішення | Пріоритет |
|----------|---------|----------|
| Велика базова image | Використовувати alpine або distroless | 🔴 Високий |
| Запуск від root | Створити non-root користувача | 🔴 Високий |
| Немає HEALTHCHECK | Додати здоров'я перевірку | 🟡 Середній |
| Немає версій пакетів | Pin specific versions | 🟡 Середній |
| Немає сканування образу | Регулярне сканування Trivy/Dockle | 🔴 Високий |
| Чутливі дані можуть бути в image | Використовувати secrets management | 🔴 Високий |

---

## 🛡️ CI/CD Pipeline для безпеки

### Этапи сканування

#### 1. **Hadolint** (Lint stage)
- Аналізує Dockerfile на порушення best practices
- Перевіряє синтаксис та структуру
- Режим: `allow_failure: true` (інформативно)

#### 2. **Trivy** (Scan stage)
- Сканує конфігурацію на вразливості
- Перевіряє залежності
- Режим: `allow_failure: false` (критично)

#### 3. **Dockle** (Audit stage)
- Будує image та аудитує його
- Перевіряє безпеку запущеного контейнера
- Створює JSON звіт
- Режим: `allow_failure: false` (критично)

---

## 📊 Приклад покращеного Dockerfile

```dockerfile
# Використання lightweight базової image
FROM alpine:3.18

# Встановлення користувача перед встановленням пакетів
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# Встановлення пакетів з фіксованими версіями
RUN apk add --no-cache curl=8.1.2-r0 bash=5.2.15-r2

# Копіювання файлів
COPY --chown=appuser:appuser . /app

WORKDIR /app

# Додання HEALTHCHECK
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:80/health || exit 1

# Переключення на non-root користувача
USER appuser

# Явний ENTRYPOINT
ENTRYPOINT ["/app/start.sh"]

# Експозиція порту для веб-сервісу
EXPOSE 80
```

---

## 🚀 Як використовувати

1. **Локальна перевірка з Hadolint**:
   ```bash
   docker run --rm -i hadolint/hadolint < Dockerfile
   ```

2. **Сканування з Trivy**:
   ```bash
   trivy config Dockerfile
   ```

3. **Аудит з Dockle**:
   ```bash
   docker build -t test-image .
   dockle test-image
   ```

4. **Запуск GitLab CI pipeline**:
   ```bash
   git push
   # Pipeline запуститься автоматично з трьома этапами
   ```

---

## 📚 Корисні ресурси

- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [Hadolint](https://github.com/hadolint/hadolint)
- [Trivy](https://github.com/aquasecurity/trivy)
- [Dockle](https://github.com/goodwithtech/dockle)

---

## 📝 Ключові висновки

✅ Регулярне сканування образів критично  
✅ Використовувати мінімальні базові образи  
✅ Ніколи не запускати контейнер від root  
✅ Автоматизувати перевірки безпеки в CI/CD  
✅ Документувати кожну інструкцію Dockerfile

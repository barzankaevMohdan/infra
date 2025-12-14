# 🚀 Infrastructure - Установка и запуск на macOS

Docker Compose конфигурация для запуска всей инфраструктуры проекта.

---

## 📋 Требования

### Docker Desktop
**Для запуска контейнеров**

**Установка:**

#### Вариант 1: Через Homebrew (рекомендуется)
```bash
brew install --cask docker
```

#### Вариант 2: Скачать вручную
- [Docker Desktop для macOS](https://www.docker.com/products/docker-desktop/)

**После установки:**
1. Запустите Docker Desktop из Applications
2. Дождитесь полной загрузки (статус должен быть зеленым)

**Проверка установки:**
```bash
docker --version
docker compose version
```

---

## 🔧 Настройка

### Шаг 1: Настройка переменных окружения

Создайте файл `.env` на основе `env.example`:
```bash
cd infra
cp env.example .env
```

Откройте `.env` и укажите параметры:
```env
# PostgreSQL
POSTGRES_DB=attendance
POSTGRES_USER=attendance_user
POSTGRES_PASSWORD=secure_password_change_me
POSTGRES_PORT=5432

# PgAdmin (опционально)
PGADMIN_EMAIL=admin@example.com
PGADMIN_PASSWORD=admin123

# Volumes
POSTGRES_DATA_PATH=./postgres-data
```

---

## ▶️ Запуск

### Запуск всех сервисов

```bash
cd infra
docker compose up -d
```

**Флаги:**
- `-d` - запуск в фоновом режиме (detached)
- `--build` - пересборка образов

---

### Запуск конкретного сервиса

```bash
# Только PostgreSQL
docker compose up -d postgres

# Только PgAdmin
docker compose up -d pgadmin
```

---

### Остановка сервисов

```bash
# Остановить все сервисы
docker compose stop

# Остановить и удалить контейнеры
docker compose down

# Остановить и удалить контейнеры + volumes
docker compose down -v
```

---

## 📊 Доступные сервисы

### PostgreSQL
- **Порт:** 5432
- **База данных:** attendance
- **Пользователь:** attendance_user
- **Пароль:** (из .env)

**Connection String:**
```
postgresql://attendance_user:your_password@localhost:5432/attendance
```

**Подключение через psql:**
```bash
psql -h localhost -p 5432 -U attendance_user -d attendance
```

---

### PgAdmin (опционально)
Веб-интерфейс для управления PostgreSQL

- **URL:** http://localhost:5050
- **Email:** (из .env)
- **Пароль:** (из .env)

**Добавление сервера в PgAdmin:**
1. Откройте http://localhost:5050
2. Войдите с учетными данными из `.env`
3. Нажмите **Add New Server**
4. На вкладке **General** укажите название: `Attendance DB`
5. На вкладке **Connection** укажите:
   - Host: `host.docker.internal` (для macOS)
   - Port: `5432`
   - Database: `attendance`
   - Username: `attendance_user`
   - Password: (из .env)

---

## 🛠️ Полезные команды

### Просмотр логов

```bash
# Все сервисы
docker compose logs -f

# Конкретный сервис
docker compose logs -f postgres

# Последние 100 строк
docker compose logs --tail 100
```

---

### Статус сервисов

```bash
# Список запущенных контейнеров
docker compose ps

# Детальная информация
docker compose ps -a
```

---

### Перезапуск сервисов

```bash
# Все сервисы
docker compose restart

# Конкретный сервис
docker compose restart postgres
```

---

### Управление volumes

```bash
# Список volumes
docker volume ls

# Удалить все volumes (⚠️ удалит все данные!)
docker compose down -v

# Создать backup базы данных
docker exec attendance-postgres pg_dump -U attendance_user attendance > backup.sql

# Восстановить из backup
docker exec -i attendance-postgres psql -U attendance_user attendance < backup.sql
```

---

## 🐛 Решение проблем

### Ошибка: "Cannot connect to database"

1. Проверьте, что контейнер запущен:
   ```bash
   docker compose ps
   ```

2. Проверьте логи PostgreSQL:
   ```bash
   docker compose logs postgres
   ```

3. Попробуйте перезапустить:
   ```bash
   docker compose restart postgres
   ```

---

### Ошибка: "Port 5432 is already in use"

Другой PostgreSQL уже запущен на порту 5432.

**Решение 1:** Остановите другой PostgreSQL
```bash
# Найти процесс
lsof -i:5432

# Остановить PostgreSQL установленный через Homebrew
brew services stop postgresql
```

**Решение 2:** Измените порт в `.env`
```env
POSTGRES_PORT=5433
```

И в `docker-compose.yml`:
```yaml
ports:
  - "${POSTGRES_PORT:-5433}:5432"
```

---

### Ошибка: "No space left on device"

Docker использовал всё доступное место.

```bash
# Очистка неиспользуемых данных
docker system prune -a

# Удаление неиспользуемых volumes
docker volume prune
```

---

### Контейнер постоянно перезапускается

```bash
# Проверьте логи
docker compose logs postgres

# Проверьте ресурсы Docker Desktop
# Docker Desktop → Settings → Resources
# Увеличьте Memory до 4GB минимум
```

---

## 🏗️ Структура проекта

```
infra/
├── docker-compose.yml         # Docker Compose конфигурация
├── .env                       # Переменные окружения
├── env.example                # Пример .env файла
├── postgres-data/             # Данные PostgreSQL (создается автоматически)
└── SETUP_MACOS.md            # Эта инструкция
```

---

## 📚 Docker Compose конфигурация

### Сервисы

**postgres:**
- Image: `postgres:15-alpine`
- Ports: `5432:5432`
- Volumes: `pgdata-attendance:/var/lib/postgresql/data`
- Restart: `unless-stopped`

**pgadmin (опционально):**
- Image: `dpage/pgadmin4`
- Ports: `5050:80`
- Depends on: `postgres`

---

## 💡 Советы

1. **Регулярные backup'ы:**
   ```bash
   # Создайте скрипт для автоматического backup
   docker exec attendance-postgres pg_dump -U attendance_user attendance | gzip > backup_$(date +%Y%m%d).sql.gz
   ```

2. **Мониторинг ресурсов:**
   ```bash
   docker stats
   ```

3. **Обновление образов:**
   ```bash
   docker compose pull
   docker compose up -d
   ```

4. **Использование docker-compose.override.yml** для локальных настроек:
   ```yaml
   # docker-compose.override.yml
   version: '3.8'
   services:
     postgres:
       ports:
         - "5433:5432"
   ```

---

## 🔗 Интеграция с другими сервисами

Все сервисы проекта подключаются к PostgreSQL через переменную окружения `DATABASE_URL`:

```env
DATABASE_URL=postgresql://attendance_user:password@localhost:5432/attendance
```

**Backend:** Использует Prisma ORM для работы с БД
**Camera Gateway:** Использует ту же схему Prisma
**Recognition Service:** Не подключается напрямую к БД

---

Если возникли проблемы, откройте Issue в репозитории! 🚀

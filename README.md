# Infrastructure

Docker Compose конфигурация для запуска инфраструктуры проекта.

## 🎯 Назначение

Infra предоставляет:
- **PostgreSQL** - база данных для всей системы
- **PgAdmin** - веб-интерфейс для управления БД (опционально)
- **Docker Compose** - оркестрация контейнеров

## 🛠️ Технологический стек

- **Docker** - Контейнеризация
- **Docker Compose** - Оркестрация
- **PostgreSQL 15** - База данных
- **PgAdmin 4** - GUI для БД (опционально)

## 📁 Структура

```
infra/
├── docker-compose.yml    # Конфигурация Docker Compose
├── .env                  # Переменные окружения
├── env.example           # Пример .env
└── postgres-data/        # Данные БД (создается автоматически)
```

## 🚀 Быстрый старт

Смотрите инструкции по установке:
- [macOS](./SETUP_MACOS.md)
- [Windows](./SETUP_WINDOWS.md)

## 📦 Сервисы

### PostgreSQL
```yaml
services:
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: attendance
      POSTGRES_USER: attendance_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pgdata-attendance:/var/lib/postgresql/data
```

**Порт:** 5432
**База данных:** attendance

### PgAdmin (опционально)
```yaml
services:
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD}
```

**URL:** http://localhost:5050

## 🗄️ База данных

### Схема

**Основные таблицы:**
- `User` - Пользователи системы
- `Company` - Компании
- `Camera` - IP-камеры
- `Employee` - Сотрудники
- `Event` - События распознавания
- `Presence` - Данные присутствия

### Подключение

**Backend и Camera Gateway:**
```env
DATABASE_URL=postgresql://attendance_user:password@localhost:5432/attendance
```

**Через psql:**
```bash
psql -h localhost -p 5432 -U attendance_user -d attendance
```

## 🔧 Команды

### Управление сервисами

```bash
# Запуск всех сервисов
docker compose up -d

# Остановка
docker compose stop

# Остановка и удаление
docker compose down

# Логи
docker compose logs -f postgres
```

### Backup и Restore

```bash
# Создать backup
docker exec attendance-postgres pg_dump -U attendance_user attendance > backup.sql

# Восстановить из backup
cat backup.sql | docker exec -i attendance-postgres psql -U attendance_user attendance
```

### Очистка

```bash
# Удалить все данные (⚠️ опасно!)
docker compose down -v

# Очистить неиспользуемые данные Docker
docker system prune -a
```

## 📊 Мониторинг

### Статус контейнеров
```bash
docker compose ps
```

### Использование ресурсов
```bash
docker stats
```

### Логи
```bash
# Все сервисы
docker compose logs -f

# Конкретный сервис
docker compose logs -f postgres

# Последние 100 строк
docker compose logs --tail 100 postgres
```

## 🔗 Интеграция с другими сервисами

### Backend
```typescript
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

Выполняет миграции и заполнение БД.

### Camera Gateway
```typescript
// Использует ту же схему Prisma (read-only)
```

Читает данные о камерах из БД.

### Recognition Service
Не подключается напрямую к БД, работает через Backend API.

## 🔐 Безопасность

### Рекомендации:
1. ✅ Измените пароли в `.env` в продакшене
2. ✅ Не коммитьте `.env` в git
3. ✅ Используйте сильные пароли
4. ✅ Ограничьте доступ к портам (firewall)
5. ✅ Регулярные backup'ы базы данных

### Пример сильного пароля:
```bash
# Генерация случайного пароля
openssl rand -base64 32
```

## 💾 Volumes

### Постоянное хранилище
```yaml
volumes:
  pgdata-attendance:
    driver: local
```

Данные PostgreSQL сохраняются между перезапусками контейнеров.

### Расположение
- **macOS/Linux:** `/var/lib/docker/volumes/`
- **Windows:** `\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes\`

## 📚 Полезные ссылки

- [PostgreSQL документация](https://www.postgresql.org/docs/)
- [Docker Compose документация](https://docs.docker.com/compose/)
- [PgAdmin документация](https://www.pgadmin.org/docs/)
- [Prisma документация](https://www.prisma.io/docs/)

---

Для детальной настройки смотрите [SETUP_MACOS.md](./SETUP_MACOS.md) или [SETUP_WINDOWS.md](./SETUP_WINDOWS.md)
# infra

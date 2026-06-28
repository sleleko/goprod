# Пример безопасного сервиса аутентификации

## 🎯 Цель

Разработать безопасный REST API сервис с функциями регистрации и аутентификации пользователей на Go.

## 📋 Реализовано

- ✅ **Регистрация пользователя** с хешированием пароля (bcrypt)
- ✅ **Вход в систему** с выдачей JWT токена
- ✅ **Защищенный эндпоинт** для получения профиля (требует JWT)
- ✅ **Защита от SQL-инъекций** (параметризованные запросы)

### API эндпоинты:
| Метод | Путь | Описание | Требует токен |
|-------|------|----------|--------------|
| POST | `/register` | Регистрация пользователя | Нет |
| POST | `/login` | Вход в систему | Нет |
| GET | `/profile` | Получить профиль | **Да** |
| GET | `/health` | Проверка состояния | Нет |

## 🏗️ Структура проекта

```
secure-service/
├── main.go              # Главный файл с запуском сервера
├── handlers.go          # HTTP обработчики
├── models.go            # Структуры данных
├── database.go          # Работа с БД
├── auth.go              # JWT и bcrypt
├── middleware.go        # Проверка токена
├── docker-compose.yml   # PostgreSQL в Docker
├── init.sql             # Схема БД
├── .env                 # Конфигурация (создать из .env.example)
├── go.mod               # Зависимости
└── README.md           # Этот файл
```

## 🚀 Быстрый старт

### 1. Настройка окружения

```bash
# Создайте .env файл из примера
cp .env.example .env

# ВАЖНО: Измените JWT_SECRET в .env на свой ключ (минимум 32 символа)
nano .env
```

### 2. Запуск базы данных

```bash
# Запустите PostgreSQL в Docker
docker-compose up -d

# Проверьте, что БД запустилась
docker-compose ps
```

### 3. Установка зависимостей

```bash
# Скачайте Go модули
go mod download
```

### 4: Запустите и протестируйте

```bash
# Запустите сервер
go run *.go

# В другом терминале тестируйте API
curl -X POST http://localhost:8080/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","username":"testuser","password":"SecurePass123"}'
```

## 🧪 Тестирование API

### 1. Проверка здоровья сервиса
```bash
curl http://localhost:8080/health
```

### 2. Регистрация пользователя
```bash
curl -X POST http://localhost:8080/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "username": "testuser",
    "password": "SecurePass123"
  }'
```

### 3. Вход в систему
```bash
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "SecurePass123"
  }'
```

### 4. Получение профиля (с токеном)
```bash
# Замените YOUR_JWT_TOKEN на токен из ответа /login
curl http://localhost:8080/profile \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## ✅ Чек-лист перед сдачей

- [x] PostgreSQL запускается через `docker-compose up`
- [x] Приложение подключается к БД и не падает
- [x] Регистрация создает пользователя в БД
- [x] Пароли хранятся как bcrypt хеш, НЕ в открытом виде
- [x] Вход возвращает валидный JWT токен
- [x] Токен можно декодировать на https://jwt.io
- [x] Эндпоинт `/profile` требует токен (без токена → 401)
- [x] Эндпоинт `/profile` работает с правильным токеном
- [x] **ВСЕ** SQL запросы используют параметры `$1, $2...`
- [x] В коде НЕТ `fmt.Sprintf` для построения SQL

## 🔍 Проверка безопасности

### Проверьте хеширование паролей:
```bash
# Подключитесь к БД
docker exec -it secure_service_db psql -U postgres -d secure_service

# Проверьте хеши паролей
SELECT email, password_hash FROM users;

# Хеш должен начинаться с $2a$ или $2b$
\q
```

### Проверьте JWT токен:
1. Скопируйте токен из ответа `/login`
2. Вставьте на https://jwt.io
3. Убедитесь, что содержит `user_id`, `email`, `username`

## 🆘 Получение помощи

### Если что-то не работает:

1. **БД не запускается**
   ```bash
   docker-compose down
   docker-compose up -d
   docker-compose logs postgres
   ```

2. **Ошибки компиляции**
   ```bash
   go mod tidy
   go mod download
   ```

3. **Сервер не запускается**
   - Проверьте .env файл
   - Убедитесь, что JWT_SECRET длиннее 32 символов
   - Проверьте, что PostgreSQL запущен

4. **Тесты API не проходят**
   - Проверьте логи сервера
   - Убедитесь, что все TODO функции реализованы
   - Проверьте правильность JSON в curl запросах

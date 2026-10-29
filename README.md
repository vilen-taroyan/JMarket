[![CI](https://github.com/vilen-taroyan/JMarket/actions/workflows/ci.yml/badge.svg)](https://github.com/vilen-taroyan/JMarket/actions/workflows/ci.yml)

# JMarket

Backend маркетплейса объявлений — упрощённый аналог Avito. Пользователи регистрируются, входят по JWT, публикуют и управляют своими объявлениями, ищут и фильтруют чужие.

## Стек

- **Java 21**, **Spring Boot 4**, Maven
- **Spring Web MVC** — REST API
- **Spring Data JPA / Hibernate**, **PostgreSQL 16**
- **Flyway**
- **Spring Security**, **OAuth2 Resource Server**, **JWT (HS256)**, BCrypt
- **Bean Validation**
- **springdoc-openapi** — Swagger UI
- **JUnit 5**, **Mockito**, **MockMvc**, **Testcontainers**
- **Docker**, **Docker Compose**, **GitHub Actions**

## Возможности

- Регистрация пользователей с хранением паролей в виде BCrypt-хэшей
- Вход по email и паролю с выдачей JWT access token
- Создание, редактирование и удаление объявлений — только автором (`403` для чужих объявлений)
- Статусы объявлений `ACTIVE`, `SOLD`, `ARCHIVED` и их смена автором
- Категории объявлений
- Список объявлений с пагинацией и сортировкой
- Поиск по заголовку и описанию без учёта регистра
- Фильтры по цене, автору, статусу и категории; по умолчанию показываются только активные объявления
- Валидация входных данных и единый формат ошибок

## API

| Метод | Путь | Доступ | Описание |
|---|---|---|---|
| `POST` | `/api/users` | публичный | Регистрация |
| `POST` | `/api/auth/login` | публичный | Вход, выдача JWT |
| `GET` | `/api/ads` | публичный | Список с фильтрами и пагинацией |
| `GET` | `/api/ads/{id}` | публичный | Объявление по id |
| `POST` | `/api/ads` | JWT | Создание объявления |
| `PUT` | `/api/ads/{id}` | JWT, автор | Редактирование |
| `PATCH` | `/api/ads/{id}/status` | JWT, автор | Смена статуса |
| `DELETE` | `/api/ads/{id}` | JWT, автор | Удаление |

Параметры `GET /api/ads`: `query`, `minPrice`, `maxPrice`, `authorId`, `status`, `categoryId`, `page`, `size`, `sort`.

Пример: `GET /api/ads?query=phone&maxPrice=50000&categoryId=1&sort=price,asc`

Полная документация доступна в Swagger UI после запуска.

## Запуск

Нужен только Docker.

```bash
git clone https://github.com/vilen-taroyan/JMarket.git
cd JMarket
cp .env.example .env
```

В `.env` замените `JWT_SECRET` на случайный ключ в Base64 (не короче 32 байт):

```bash
openssl rand -base64 32
```

Запуск приложения вместе с PostgreSQL:

```bash
docker compose up --build
```

- API: http://localhost:8080/api/ads
- Swagger UI: http://localhost:8080/swagger-ui.html

Миграции Flyway применяются автоматически при старте.

## Тесты

Нужны JDK 21 и запущенный Docker (для Testcontainers).

```bash
./mvnw verify
```

Тесты разделены на уровни:

- **unit-тесты сервисов** — JUnit 5 и Mockito;
- **тесты web-слоя** — `@WebMvcTest` и MockMvc: JSON, HTTP-статусы, валидация, правила доступа;
- **тесты репозиториев** — `@DataJpaTest` на настоящем PostgreSQL в Testcontainers;
- **интеграционные тесты** — `@SpringBootTest` через весь стек, включая полный сценарий регистрация → вход → работа с объявлением по настоящему JWT.

# Лабораторная работа 14 — защищённый список задач

## Краткий отчёт

Защищённое управление задачами с ролями user и admin. Данные находятся в PostgreSQL.

## Выполнение по заданию

### 1. Пользователи и токены

`POST /auth/register` создаёт пользователя с ролью user и bcrypt-хешем пароля. `POST /auth/login` выдаёт JWT на 60 минут. Middleware проверяет Bearer, HS256, подпись и срок действия. Секрет берётся из `JWT_SECRET`.

### 2. Права и маршруты

GET и POST `/tasks`, GET, PUT и DELETE `/tasks/{id}`; GET `/admin/statistics`. Пользователь видит и изменяет свои задачи. Администратор видит и изменяет любые задачи, удаляет задачи и смотрит общую статистику. Владелец при создании берётся из токена.

### 3. Проверка

Проверены регистрация, повторный логин, неверный пароль, неправильный и просроченный JWT, создание задачи, доступ к чужой задаче и права администратора.

## Запуск

Локальная база `secure_tasks_db` должна быть создана с помощью sql/init.sql. Если сервер PostgreSQL остановлен, запустите его командой из README работы модуля 13. Затем в PowerShell этой папки:

```powershell
$env:DATABASE_URL = 'postgres://postgres:YOUR_POSTGRES_PASSWORD@127.0.0.1:55432/secure_tasks_db?sslmode=disable'
$env:JWT_SECRET = "YOUR_JWT_SECRET"
go run .
```

Создайте пользователей для проверки. Используйте собственные учебные логины и пароли.

На другой базе сначала выполните `sql/init.sql`. Создайте администратора отдельной командой:

```powershell
$env:ADMIN_USERNAME = "admin"
$env:ADMIN_PASSWORD = "YOUR_ADMIN_PASSWORD"
go run . create-admin
```

Замените значения `YOUR_POSTGRES_PASSWORD`, `YOUR_JWT_SECRET` и `YOUR_ADMIN_PASSWORD` своими и задайте `DATABASE_URL` и `JWT_SECRET` перед этой командой. Обычного пользователя можно зарегистрировать через `POST /auth/register`.

## Пример проверки в PowerShell

Откройте второе окно PowerShell, пока сервер работает:

```powershell
$body = @{ username = "student"; password = "YOUR_USER_PASSWORD" } | ConvertTo-Json
$login = Invoke-RestMethod -Method Post -Uri http://localhost:8080/auth/login -ContentType "application/json" -Body $body
$headers = @{ Authorization = "Bearer $($login.access_token)" }
```

Далее используйте `$headers` в защищённых запросах. Для администратора войдите с `admin` и своим паролем администратора.

```powershell
$item = @{ title = "Моя задача"; description = "Описание"; status = "new" } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://localhost:8080/tasks -Headers $headers -ContentType "application/json" -Body $item
Invoke-RestMethod -Uri http://localhost:8080/tasks -Headers $headers
```

## Вывод

JWT подтверждает пользователя, а роль и владелец записи определяют доступ. Пароли и хеши не возвращаются в ответах.

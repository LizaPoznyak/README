# **Программное средство реализации онлайн-сервиса для развития логики и математических способностей ребенка**

Умоград — это образовательная платформа, разработанная для интерактивного обучения детей логике и математике через адаптивные задания, отслеживание прогресса и удобный интерфейс для родителей и администраторов.

Цель проекта — создать масштабируемое, модульное и безопасное программное средство, соответствующее принципам Clean Architecture, DDD и CQRS, с интуитивным интерфейсом и возможностью интеграции с внешними образовательными сервисами.

Основные возможности:
* Многоуровневая модель пользователей: родитель, ребёнок, администратор.
* Адаптивные задания: drag‑and‑drop, логические цепочки, математические игры и т.п..
* Отслеживание прогресса: отчёты, рекомендации, история выполнения.
* Безопасность: авторизация через JWT, разграничение доступа.
* Интеграции: подключение внешних API для расширения контента.
* Единая система дизайна: дружественный интерфейс, подходящий для детей.

Ссылки на репозитории сервера и клиента:

Backend: https://github.com/LizaPoznyak/umograd-backend Содержит реализацию REST API, бизнес-логику, репозитории и конфигурацию безопасности.

Frontend: https://github.com/LizaPoznyak/umograd-frontend Содержит SPA-интерфейс для детей, родителей и администраторов, реализованный на React.

---

## **Содержание**

1. [Архитектура](#Архитектура)
	1. [C4-модель](#C4-модель)
	2. [Схема данных](#Схема_данных)
2. [Функциональные возможности](#Функциональные_возможности)
	1. [Диаграмма вариантов использования](#Диаграмма_вариантов_использования)
	2. [User-flow диаграммы](#User-flow_диаграммы)
3. [Детали реализации](#Детали_реализации)
	1. [UML-диаграммы](#UML-диаграммы)
	2. [Спецификация API](#Спецификация_API)
	3. [Безопасность](#Безопасность)
	4. [Оценка качества кода](#Оценка_качества_кода)
4. [Тестирование](#Тестирование)
	1. [Unit-тесты](#Unit-тесты)
	2. [Интеграционные тесты](#Интеграционные_тесты)
5. [Установка и  запуск](#installation)
	1. [Манифесты для сборки docker образов](#Манифесты_для_сборки_docker_образов)
	2. [Манифесты для развертывания k8s кластера](#Манифесты_для_развертывания_k8s_кластера)
6. [Лицензия](#Лицензия)
7. [Контакты](#Контакты)

---
## **Архитектура**

### C4-модель

Архитектура проекта Умоград документирована с использованием C4-модели, которая позволяет описать систему на разных уровнях абстракции:
* Контекстная диаграмма — показывает, как программное средство взаимодействует с внешними пользователями (родители, дети, администраторы) и системами (образовательные API, базы данных).

<img width="2158" height="880" alt="{33EE65C8-19C6-4FF4-8711-40F51C027139}" src="https://github.com/user-attachments/assets/7734c6a3-cd48-4981-b245-bacef6d4ae8c" />

* Диаграмма контейнеров — описывает основные компоненты системы: backend-приложение (Spring Boot), frontend-интерфейс (React SPA), база данных (MySQL), а также микросервисы и шлюзы интеграции.

<img width="1718" height="1091" alt="{87365FD5-A521-4E6F-AD35-A57330CA1BAE}" src="https://github.com/user-attachments/assets/4eeec7fc-d5a3-4b34-b966-abf39d26e277" />

* Диаграмма компонентов — раскрывает внутреннюю структуру backend-контейнера: слои Clean Architecture (Domain, Application, Interface Adapters, Infrastructure), use case-обработчики, контроллеры, репозитории и сервисы.

<img width="1776" height="1035" alt="{7CF023C8-6282-41C6-87C2-B60EC14BB688}" src="https://github.com/user-attachments/assets/5e92dc23-68eb-487a-85e9-5b5dacfaef44" />

### Схема данных

# Описание отношений и структур данных

Программное средство опирается на три взаимосвязанных блока данных:  
**пользователи и роли**, **контент (задания и результаты)** и **аналитика**.

---

## Пользователи и роли

### Таблица `users`
Хранит информацию о зарегистрированных пользователях системы.  

**Атрибуты:**
- `user_id` (PK) – уникальный идентификатор пользователя  
- `username` – имя или логин  
- `email` – электронная почта (уникальная)  
- `password` – хэш пароля  
- `parent_id` (FK → users.id) – ссылка на родителя (для детей)  

### Таблица `user_roles`
Реализует ролевую модель доступа.  

**Атрибуты:**
- `user_id` (PK, FK → users.id) – идентификатор пользователя  
- `role` – роль пользователя (`ROLE_CHILD`, `ROLE_PARENT`, `ROLE_MODERATOR`  

---

## Контент 

### Таблица `tasks`
Хранит задания, доступные детям.  

**Атрибуты:**
- `task_id` (PK) – уникальный идентификатор задания  
- `title` – заголовок  
- `description` – описание  
- `difficulty` – уровень сложности (`EASY`, `MEDIUM`, `HARD`)  
- `min_age`, `max_age` – возрастной диапазон  
- `content_type` – тип задания (`MULTIPLE_CHOICE`, `TEXT`, `IMAGE`)  
- `question` – формулировка вопроса  
- `options` – варианты ответа   
- `answer` – правильный ответ  
- `created_by` (FK → users.id) – автор задания  
- `created_at`, `updated_at` – даты создания и изменения  
- `source_id` – внешний источник (если задание импортировано)  

### Таблица `task_results`
Фиксирует выполнение заданий детьми.  

**Атрибуты:**
- `id` (PK) – уникальный идентификатор результата  
- `task_id` (FK → tasks.id) – ссылка на задание  
- `child_id` (FK → users.id) – ссылка на ребёнка  
- `status` – состояние выполнения (`IN_PROGRESS`, `DONE`)  
- `score` – количество баллов  
- `attempts` – количество попыток  
- `finished_at` – дата завершения  

---

## Аналитика

### Таблица `analytics_events`
Хранит «сырые» события для последующего анализа.  

**Атрибуты:**
- `event_id` (PK) – уникальный идентификатор события  
- `child_id` (FK → users.id) – ссылка на ребёнка  
- `task_id` (FK → tasks.id) – ссылка на задание  
- `event_type` – тип события (`STARTED`, `ATTEMPT`, `FINISHED`, `GIVE_UP`)  
- `event_time` – время события  
- `selected_answer` – выбранный ответ  
- `is_correct` – правильность ответа  
- `duration_ms` – время, затраченное на попытку  

### Таблица `child_progress`
Агрегированные показатели по каждому ребёнку.  

**Атрибуты:**
- `child_id` (PK, FK → users.id) – идентификатор ребёнка  
- `tasks_completed` – количество завершённых заданий  
- `average_score` – средний балл  
- `average_attempts` – среднее число попыток  
- `last_activity_at` – дата последней активности  

### Таблица `task_statistics`
Агрегированные показатели по каждому заданию.  

**Атрибуты:**
- `task_id` (PK, FK → tasks.id) – идентификатор задания  
- `times_completed` – сколько раз завершено  
- `average_score` – средний балл  
- `average_attempts` – среднее число попыток  
- `common_wrong_answers` – частые ошибки 

---

# Скрипт генерации БД 

```
-- =========================
-- 1. База пользователей
-- =========================
CREATE DATABASE IF NOT EXISTS umgrad;

USE umgrad;

-- Таблица пользователей
CREATE TABLE users (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    username    VARCHAR(100) NOT NULL UNIQUE,
    email       VARCHAR(150) NOT NULL UNIQUE,
    password    VARCHAR(255) NOT NULL,
    parent_id   INT,
    CONSTRAINT fk_users_parent FOREIGN KEY (parent_id) REFERENCES users(id)
        ON DELETE SET NULL
);

-- Таблица ролей пользователей
CREATE TABLE user_roles (
    user_id INT NOT NULL,
    role    VARCHAR(50) NOT NULL,
    PRIMARY KEY (user_id, role),
    CONSTRAINT fk_roles_user FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE
);


-- =========================
-- 2. База заданий
-- =========================
CREATE DATABASE IF NOT EXISTS content_db;

USE content_db;

-- Таблица заданий
CREATE TABLE tasks (
    id           INT AUTO_INCREMENT PRIMARY KEY,
    title        VARCHAR(255) NOT NULL,
    description  TEXT,
    difficulty   ENUM('EASY','MEDIUM','HARD') NOT NULL,
    min_age      INT,
    max_age      INT,
    content_type ENUM('MULTIPLE_CHOICE','TEXT','IMAGE') NOT NULL,
    question     TEXT NOT NULL,
    options      TEXT,
    answer       TEXT,
    created_by   INT, -- логическая ссылка на umgrad.users.id
    created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    source_id    VARCHAR(100)
);

-- Результаты выполнения заданий
CREATE TABLE task_results (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    task_id     INT NOT NULL, -- логическая ссылка на content_db.tasks.id
    child_id    INT NOT NULL, -- логическая ссылка на umgrad.users.id
    status      ENUM('IN_PROGRESS','DONE') NOT NULL,
    score       INT,
    attempts    INT DEFAULT 0,
    finished_at TIMESTAMP NULL
);


-- =========================
-- 3. База аналитики
-- =========================
CREATE DATABASE IF NOT EXISTS analytics_db;

USE analytics_db;

-- Сырые события аналитики
CREATE TABLE analytics_events (
    id              INT AUTO_INCREMENT PRIMARY KEY,
    child_id        INT NOT NULL, -- логическая ссылка на umgrad.users.id
    task_id         INT NOT NULL, -- логическая ссылка на content_db.tasks.id
    event_type      ENUM('STARTED','ATTEMPT','FINISHED','GIVE_UP') NOT NULL,
    event_time      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    selected_answer TEXT,
    is_correct      BOOLEAN,
    duration_ms     INT
);

-- Прогресс ребёнка
CREATE TABLE child_progress (
    child_id         INT PRIMARY KEY, -- логическая ссылка на umgrad.users.id
    tasks_completed  INT DEFAULT 0,
    average_score    DECIMAL(5,2),
    average_attempts DECIMAL(5,2),
    last_activity_at TIMESTAMP NULL
);

-- Статистика по заданиям
CREATE TABLE task_statistics (
    task_id          INT PRIMARY KEY, -- логическая ссылка на content_db.tasks.id
    times_assigned   INT DEFAULT 0,
    times_completed  INT DEFAULT 0,
    average_score    DECIMAL(5,2),
    average_attempts DECIMAL(5,2),
    common_wrong_answers JSON
);
```

<img width="850" height="1050" alt="{40A962FB-D1CA-4810-B7E2-216769F15A69}" src="https://github.com/user-attachments/assets/1c1fe47b-d60e-4ede-9175-68a8c85cae19" />

---

## **Функциональные возможности**

### Диаграмма вариантов использования

Диаграмма вариантов использования и ее описание

### User-flow диаграммы

1. User-flow для модератора
   
Диаграмма отражает процесс работы администратора: от авторизации и перехода на главную страницу до создания нового задания. Включает проверку обязательных полей, подтверждение сохранения и возврат к списку заданий. Основная цель — обеспечить корректное добавление и публикацию контента.

<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/3788c00a-7643-4fca-9c44-e3b36ba3a27d" />

2. User-flow для ребёнка
   
Диаграмма описывает путь ребёнка: вход в систему, выбор задания, выполнение интерактивных действий и получение обратной связи. Включена проверка правильности ответа и возможность повторного прохождения. Завершается процесс выдачей награды или достижения, что поддерживает мотивацию к обучению.

<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/1c60a68a-3f33-415d-adb6-f466f12c0e5b" />

3. User-flow для родителя

Диаграмма показывает сценарий работы родителя: авторизация, выбор профиля ребёнка, просмотр статистики и достижений. При необходимости родитель может перейти к подробному отчёту с историей заданий и динамикой прогресса. Основная цель — прозрачный мониторинг образовательного процесса.

<img width="975" height="512" alt="image" src="https://github.com/user-attachments/assets/13d60386-f026-4fc7-b3d6-41398feaa966" />

---

## **Детали реализации**

### UML-диаграммы

Представить все UML-диаграммы , которые позволят более точно понять структуру и детали реализации ПС

### Спецификация API

Представить описание реализованных функциональных возможностей ПС с использованием Open API (можно представить либо полный файл спецификации, либо ссылку на него)

### Безопасность

Описать подходы, использованные для обеспечения безопасности, включая описание процессов аутентификации и авторизации с примерами кода из репозитория сервера

### Оценка качества кода

Используя показатели качества и метрики кода, оценить его качество

---

## **Тестирование**

### Unit-тесты

Представить код тестов для пяти методов и его пояснение

### Интеграционные тесты

Представить код тестов и его пояснение

---

## **Установка и  запуск**

### Манифесты для сборки docker образов

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

### Манифесты для развертывания k8s кластера

Представить весь код манифестов или ссылки на файлы с ними (при необходимости снабдить комментариями)

---

## **Лицензия**

Этот проект лицензирован по лицензии MIT - подробности представлены в файле [LICENSE.md]

---

## **Контакты**

Автор: eupoznyak@gmail.com

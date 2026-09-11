# Learning Log

Учебный проект на Django. Приложение представляет собой личный журнал:
пользователь создаёт темы и добавляет по ним записи. Данные каждого
пользователя видны только ему.

Проект разворачивается через Docker Compose. Внутри контейнера работает
Gunicorn, статику отдаёт WhiteNoise. Приложение доступно по IP-адресу
сервера на порту 8001 по протоколу HTTP.

## Стек

Python 3.12, Django 5.x, Gunicorn, WhiteNoise, SQLite, Docker,
Docker Compose.

## Запуск для разработки

Создать виртуальное окружение и активировать его:

    python -m venv ll_env
    source ll_env/bin/activate

На Windows:

    ll_env\Scripts\activate

Установить зависимости:

    pip install -r requirements.txt

Применить миграции:

    python manage.py migrate

Запустить сервер разработки:

    python manage.py runserver

Приложение будет доступно по адресу http://127.0.0.1:8000/

## Развёртывание на сервере

### Что нужно заранее

Нужен VPS с Ubuntu 22.04 или 24.04 и root-доступ по SSH. Порт, на
котором будет работать приложение (по умолчанию 8001), должен быть
открыт в firewall.

### Установка Docker

    apt update
    apt install -y docker.io docker-compose-v2 git
    systemctl enable --now docker

### Получение кода

    cd /opt
    git clone https://github.com/ТВОЙ_ЛОГИН/learning_log.git
    cd learning_log

### Настройка переменных окружения

Скопировать пример и отредактировать:

    cp .env.example .env
    nano .env

В файле .env нужно заполнить:

DJANGO_SECRET_KEY — длинный случайный ключ. Сгенерировать можно так:

    python3 -c "import secrets; print('django-insecure-' + secrets.token_urlsafe(50))"

DJANGO_DEBUG — на сервере должно быть False.

DJANGO_ALLOWED_HOSTS — список хостов через запятую. Сюда нужно
включить IP-адрес сервера, например: 185.123.45.67,localhost,127.0.0.1

DJANGO_CSRF_TRUSTED_ORIGINS — оставить пустым, так как сайт работает
по HTTP. Если позже будет настроен HTTPS, сюда нужно будет добавить
адрес сайта с https.

SQLITE_PATH — путь к базе внутри контейнера, по умолчанию
/app/data/db.sqlite3

APP_BIND_IP — IP, на котором контейнер слушает снаружи. Чтобы
приложение было доступно по внешнему IP сервера, указать 0.0.0.0

APP_PORT — внешний порт контейнера. По умолчанию 8001

### Запуск контейнера

    docker compose up -d --build

Проверить состояние:

    docker compose ps
    docker compose logs --tail=100 web

Создать администратора:

    docker compose exec web python manage.py createsuperuser

После запуска приложение доступно по адресу http://IP_СЕРВЕРА:8001/

### Открытие порта в firewall

Если на сервере включён ufw, нужно разрешить порт 8001:

    ufw allow 8001/tcp
    ufw reload

Если ufw не используется, порт обычно открыт по умолчанию. Проверить
можно так:

    ufw status

## Обновление проекта

    cd /opt/learning_log
    git pull
    docker compose up -d --build

## Переменные окружения

Полный список переменных с пояснениями приведён в файле .env.example.
Реальный файл .env в репозиторий не загружается.

## Структура проекта

    learning_log/
        compose.yaml
        Dockerfile
        requirements.txt
        manage.py
        learning_log/       настройки проекта
            settings.py
            urls.py
            wsgi.py
        learning_logs/      приложение тем и записей
            models.py
            views.py
            forms.py
            urls.py
            templates/
        users/              приложение пользователей
            views.py
            urls.py
            templates/
        data/               база SQLite внутри контейнера

## Что не загружается в репозиторий

Файл .env, виртуальное окружение, база данных, каталог staticfiles
и прочие локальные файлы перечислены в .gitignore и .dockerignore.

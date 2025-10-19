## ПОДГОТОВКА (СДЕЛАТЬ ОДИН РАЗ)

1. Создать папки /www/ и /env/ | mkdir ~/Documents/www/ ; mkdir ~/Documents/env/

BACKEND
=======

## УСТАНОВКА DJANGO

1. Установить новый питон | https://www.python.org/
1. Создать папку /myproject/ в /www/ | mkdir ~/Documents/www/myproject/
2. Открыть папку /myproject/ в IDE
3. Создать python-окружение /myproject/ в /env/ | python3.14 -m venv ~/Documents/env/myproject/
4. Активировать python-окружение | source ~/Documents/env/myproject/bin/activate | ~/Documents/env/myproject/bin/python
5. Установить Django | pip install django
6. Создать djagno-проект | mkdir api ; django-admin startproject conf api

## НАСТРОЙКА IDE

1. Указать IDE путь к python-окружению | Interpretor -> Add -> ~/Documents/env/myproject/bin/python
2. Указать IDE путь к Django и её настройкам | Enable Django Support -> Django project root, Settings
3. Указать IDE Sources Root | Right click on '/api/' -> Mark Directory as > Sources Root

## ПОДГОТОВКА К ПОДКЛЮЧЕНИЮ POSTGRESQL

1. Создать БД в PostgreSQL (настроить UI — DataGrip или аналог) | createdb myproject
2. Установить PostgreSQL адаптер | pip install psycopg

## ОКРУЖЕНИЕ, ЛОКАЛЬНЫЕ НАСТРОЙКИ И ПОДКЛЮЧЕНИЕ К POSTGRESQL

1. Создать окружение в /myproject/ | touch .env | см. template
2. Создать шаблон окружения | cp .env .env.tmpl
3. Создать локальные настройки в /api/conf/ | touch api/conf/settings_local.py | см. template
4. Описать подключение к PostgreSQL в локальных настройках
5. Создать шаблон локальных настроек | cp api/conf/settings_local.py api/conf/settings_local.py.tmpl
5. В самом низу settings.py добавить лейбл `# CUSTOM` для будующих настроек | см. template
6. В самый низу settings.py, под `#CUSTOM` добавить `# LOCAL` и импортировать локальные настройки | см. template

## REQUIREMENTS

1. Создать requirements.txt в /api/ | touch api/requirements.txt
2. Установить pipdeptree | pip install pipdeptree

## INVOKE

1. Установить invoke | pip install invoke
2. Создать tasks.py в /myproject/ | touch tasks.py | см. template
3. Убедиться, что описана функция `freeze`, вызывающая `pipdeptree -fl` | см. template
4. Получить список установленных пакетов, вынести их руками в requirements.txt, исключив лишнее | invoke freeze
5. Создать шаблон invoke | cp tasks.py tasks.py.tmpl

## ПОДГОТОВКА К ПЕРВОМУ АППЛИКЕЙШЕНУ

1. Создать пакет /apps/ в /api/, внутри создать пустой пакет /migrations/, models.py, admin.py | см. template
2. Подключить `apps` внизу `settings.py:INSTALLED_APPS` с лейблом `# Custom` | см. template

## ПОЛЬЗОВАТЕЛИ

1. Создать пакет /users/ в /api/apps/ с models.py, admin.py | см. template
2. Отпроксировать models.py, admin.py пользователя в /api/apps/models.py, /api/apps/admin.py | см. template
3. Импортировать локальные настройки в самый низ settings.py с лейблом `# LOCAL` | см. template
4. Указать модель пользователя в `settings.py:AUTH_USER_MODEL` в блоке `# CUSTOM` под новым лейблом `User` | см. template
5. Там же можно указать время жизни сесии `SESSION_COOKIE_AGE` | см. template

## ПОДГОТОВКА К ЗАПУСКУ

1. tasks.txt
2. tasks.txt.tmpl
3. Переименовать текущую вкладку консоли в 'Root', там будем вызывать `invoke`
4. Создать новую вкладку в консоли 'API', `cd api`, там будем вызывать `python manage.py runserver`
5. Создать новую вкладку в консоли 'Migrate', `cd api`, там будем вызывать `python manage.py makemigraions ; python manage.py migrate`

## ПЕРВЫЙ ЗАПУСК

1. Migrate: Создать миграции | python manage.py makemigrations
2. Migrate: Впервые смигрировать | python manage.py migrate
3. Migrate: Создать супер-пользователя | python manage.py createsuperuser
4. API: Запустить проект | python manage.py runserver
5. Browser: Перейти в панель администратора и авторизоваться | http://127.0.0.1:8000/admin/

API
===

## NINJA

1. Установить django-ninja | pip install django-ninja
2. Описать подключение ninja в api/api.py | см. template
3. Заготовить файл импортов АПИ api/conf/apis.py | см. template
4. Подключить ninja и АПИ в api/conf/urls.py | см. template
5. Создать первое АПИ (для пользователя) | см. template
6. Указать АПИ пользователя в api/conf/apis.py | см. template

## CSRF И STEM

1. Создать пакет /stem/ в /api/ | см. template
2. Создать файл /api/stem/api_csrf.py | см. template

DISTRIBUTED TASKS
=================

## СЕЛЕРИ

1. Установить celery | pip install celery
2. api/app.py | см. template
3. Добавить запуск Celery в tasks.txt

LIB
===

1. Создать пакет /lib/ в /api/
2. Взять из шаблона всё необходимое, в первую очередь models.py и schema.py
3. Отнаследовать модели и схема от BaseModel BaseSchema по необходимости

ПАНЕЛЬ АДМИНИСТРАТОРА
=====================

## ДОКУМЕНТАЦИЯ
https://docs.djangoproject.com/en/5.2/ref/contrib/admin/admindocs/

1. Установить docutils | pip install docutils
2. Указать правило в urls.py (над /admin/) | см. template

## ЗАЩИТА

1. Защитить админа от брут-форса | см. template 'apps/users/secure/'
2. Перемиеновать urls с /admin/ на что-то | см. template
3. Опционально, на /admin/ поставить honeypot https://github.com/dmpayton/django-admin-honeypot

## DAOS
https://github.com/webentlib/daos

1. Пройти по иснструкции
2. Описать меню в /api/stem/daos_menu.py
3. Отнаследовать страницы, требующие тонкого управления от DaosAdmin

## AXIS

1. Создать пакет /axis/ в /api/, внутри создать пустой пакет /migrations/, models.py, admin.py | см. template
2. Подключить `axis` внизу `settings.py:INSTALLED_APPS` с лейблом `# Custom` | см. template
3. Создать пакеты /tech/ и /access/ в /api/apps/ с models.py, admin.py | см. template
4. Отпроксировать models.py, admin.py /tech/ и /access/ в /api/apps/models.py, /api/apps/admin.py | см. template

## ВОЗМОЖНОСТИ ДЖАНГИ О КОТОРЫХ СТОИТ ЗНАТЬ

1. The display decorator: https://docs.djangoproject.com/en/5.2/ref/contrib/admin/#the-display-decorator
2. Admin actions: https://docs.djangoproject.com/en/5.2/ref/contrib/admin/actions/

БИБЛИОТЕКИ, СПРАВОЧНИКИ, ПЕЧОЧНИЦА, ДОКУМЕНТАЦИЯ
================================================

1. Создать пакет /seeds/ в /api/ и папку /seeds/ c README.md внутри в корне /myproject/
2. Создать папки /dev/, /docs/, /domain/ в корне /myproject/ | см. template
3. Создать папки /test/, /text/ c README.md внутри в корне /myproject/ | см. template

ФРОНТ
=====

1. Установить SvelteKit
2. Подключить Роутер
3. Подключить lab
4. Подключить les
5. Создать lib
6. Описать base.svelte
7. Описать urls.ts
8. Создать локальные настройки
9. hooks.server.js, svelte.config.js, vite.config.js
10. all.ts
11. icons.ts
12. npm run 3000 в package.json
13. tsconfig.json
14. src, включая auth

NGINX
=====

1. Создать папку /nginx/ в /myproject/
2. Описать myproject.nginx.conf | см. template

СЕКРЕТЫ
=======

1. Сгенерировать SECRET_KEY для settings_local.py `from django.core.management.utils import get_random_secret_key ; print(get_random_secret_key())`

GIT
===

1. Создать .gitignore в корне /myproject/ | см. template
2. Создать .gitignore в /api/ | см. template
3. Создать .gitignore в /art/ | см. template
4. Добавить к трекинку дополнительные репозитории. Создать соответствующие папки с пустым README.md, [добавить], удалить и спулить репозитории заново.

DOCKER
======

1. Создать Dockerfile для /api/ и /art/ | см. template
2. Создать .dockerignore для /api/ и /art/ | см. template
3. Создать docker-compose.yml для всего | см. template

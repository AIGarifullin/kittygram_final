# Проект Kittygram final 
![workflow](https://github.com/AIGarifullin/kittygram_final/actions/workflows/main.yml/badge.svg)

## Описание проекта
Проект *kittygram_final* позволяет зарегистрированным пользователям создавать/обновлять/получать/удалять свои публикации с описанием имени, даты рождения, цвета, достижений своего домашнего любимца (кота или кошки). Также пользователь может приложить/удалить фотографию питомца. 

## Стек проекта
Python, Django REST Framework, Nginx, DNS, HTTPS, Docker, PostgreSQL, GitHub Actions

## Ссылка на развернутый проект
https://kittyprojectaig.ddns.net/

## Запуск проекта
Перейти в директорию с файлом **docker-compose.yml**, выполнить запуск и миграции:

```
docker compose up
docker compose exec backend python manage.py migrate
```
## Отличия обычной версии проекта от продакш
Продакш-версия проекта позволяет:
* Автоматизировать запуск и обновление приложения;
* Сделать запуск приложения воспроизводимым на любых серверах, независимо от настроек.

Для этого необходимо:
1. Собрать образы `kittygram_frontend`, `kittygram_backend` и `kittygram_gateway` и залить их на Docker Hub:

    ```
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway .
    ```
    ```
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```    
2. Создать в корневой директории проекта файл конфигурации **docker-compose.production.yml**, который будет использовать собранные образы на Docker Hub и управлять запуском контейнеров на продакш-сервере. Отличие нового файла конфигурации от **docker-compose.yml** состоит в замене `build` на `image` с указанием ссылки на образ (`postgres:13.10` для `db`, `username/kittygram_frontend` для `frontend`, `username/kittygram_backend` для `backend` и `username/kittygram_gateway` для `gateway`).  

3. Развернуть и запустить Docker на сервере. 
  Поочередно выполнить на сервере следующие команды:
    ```
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt install docker-compose-plugin
    ```
4. Загрузить на сервер новый файл конфигурации для Docker Compose и запустить контейнеры.
    Поместить в директорию проекта на сервере файл конфигурации **docker-compose.production.yml**, а также файл **.env**.
    Выполнить команду на сервере в папке проекта:
    ```
    sudo docker compose -f docker-compose.production.yml up -d
    ```
    Выполнить миграции, собрать статические файлы бэкенда и скопировать их в `/backend_static/static/`:
    ```
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/static/. /backend_static/static/
    ```
5. Настроить «внешний» Nginx, что вне контейнера — для работы с приложением в контейнерах.
    Открыть файл **default** конфигурации Nginx:
    
    ```
    nano /etc/nginx/sites-enabled/default
    ```
    Изменить настройки `location` в секции `server` (три блока `location` заменить на один):
    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```
    Сделать проверку и перезагрузку файла конфигурации:
    ```
    sudo nginx -t
    sudo service nginx reload
    ```

6. Для автоматизации запуска и обновления проекта на продакш-сервере создать в директории `.github/workflows` файл **main.yml** с описанием *workflow*. Сделать коммит проекта и разместить его в удаленный репозиторий:
    ```
    git add .
    git commit -m 'Add Actions'
    git push
    ```
    Для управления процессами *CI/CD* (*Continuous Integration/Continuous Delivery*) открыть раздел *Actions* в репозитории проекта в своем аккаунте.


_Текст данного файла составил [Адель Гарифуллин](https://github.com/AIGarifullin)._

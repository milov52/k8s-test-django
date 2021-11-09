# Test Kubernetes Fast Deploy

Пустой докеризированный сайт на Django для экспериментов с Kubernetes.

В конейнере Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняют одновременно две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он сам запускает Django. Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

```shell-session
$ docker-compose up
$ docker-compose run web ./manage.py migrate
$ docker-compose run web ./manage.py createsuperuser
```
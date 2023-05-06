# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).



## Запуск в кластере minikube

Запустите minikube командой

```$ minikube start ```

В директории kubernetes внесите данные переменных окружения в манифест secrets, указав необходимые значения для django
Затем выполните команду для создания Secrets в вашем кластере minikube

```$ kubectl apply -f secrets.yaml ```

### Создание pods (с помощью deployment) и services
Для запуска pods и services из директории kubernetes запустите манифест djangoapp-deploy.yaml командой

```$ kubectl apply -f djangoapp-deploy.yml ```

### База данных PostgreSQL
Работа с базой данных выполняется с помощью Helm Chart for PostgreSQL. Для установки и настройки базы данных в кластере 
необходимо последовательно выполнить следующие команды:

1. Установить helm на свой компьютер. Инструкции по установки для различных операционных систем приведены на сайте
https://helm.sh/

2. Установить Helm Chart for PostgreSQL
```
$ helm repo add my-repo https://charts.bitnami.com/bitnami
$ helm install db my-repo/postgresql
```

После создания базы данных перейти в клиент командой

```
kubectl run db-postgresql-client --rm --tty -i --restart='Never' --namespace default 
        --image docker.io/bitnami/postgresql:15.2.0-debian-11-r0 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
        --command -- psql --host db-postgresql -U postgres -d postgres -p 5432
```

И выполнить нижеприведенные команды для создания базы данных и пользователя для работы

```
postgres=# create database test_k8s;
postgres=# create user test_k8s with encrypted password 'OwOtBep9Frut';
postgres=# GRANT postgres to test_k8s
```

Для создания необходимых для работы миграций выполнить манифест migration.yml командой

```$ kubectl apply -f migrationyml ```


### Ingres

Для создания ingres выполнить манифест ingres.yml

```$ kubectl apply -f ingres.yml ```

Внести в файл etc/host на своем компьютере строку

```127.0.0.1 star-burger.test ```

для возможности работы по доменному имени star-burger.test на своем компьютере

При работе на операционной системе macos для правильной работы ingres выполнить команду

```minikube tunnel```

### Очистка сессий пользователей

Для очистки сессий был сконфигурирован CronJob, который устанавливается командой

```$ kubectl apply -f django-clearsessions_pod.yml ```

в настройках манифеста можно установить требуемый график удаления сессий

## Запуск в кластере kubernetes

### База данных PostgreSQL
Установить через helm postgresql с помощью Helm-чарта с указанием storage class

1. [Установите менеджер пакетов Kubernetes Helm](https://helm.sh/ru/docs/intro/install).
2. Создать файл volumes, где прописать все креды для базы данных, а также тип storageClass для хранения данных
3. Выполнить команду

	`helm install postgresql-dev -f values.yaml oci://registry-1.docker.io/bitnamicharts/postgresql`

## Установка Nginx ingress контроллера через helm

Для установки [Helm-чарта](https://helm.sh/docs/topics/charts/) с Ingress-контроллером NGINX выполните команду:

`helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && \ helm repo update && \ helm install ingress-nginx ingress-nginx/ingress-nginx`

Получить ip адрес балансировщика с помощью команды

`kubectl get svc`

Прописать этот адрес у регистратора доменных имен с помощью A записи

## Установить приложение django

Установка через kubectl посредство выполнения манифестов

`kubectl apply -f secrets.yml -f djangoapp-deploy.yml -f ingres.yml -f migration.yml` 

После этого по зарегистрированногому доменному имени будет доступен сайт на 80 порту

## Установка менеджера сертификатов и получение сертификата для сайта

1. Установить менеджер сертификатов через команду

`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml`

2. Выполнить манифест issuer.yaml командой для создания ClusterIssuer

`kubectl apply -f issuer.yaml`


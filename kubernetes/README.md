# Создание csi storage

[Настройка кластера](./cluster/README.md)

# Создание namespace

`kubectl apply -f namespace.yaml`

# Создание секретов из env файла

Предварительно переименовать из example и заполнить файлы с секретами

`kubectl create secret generic django-secret --from-env-file=django-secrets -n receipt-promo`

`kubectl create secret generic nodered-secret --from-env-file=nodered-secrets -n receipt-promo`

`kubectl create secret generic postgres-secret --from-env-file=postgres-secrets -n receipt-promo`

`kubectl create secret generic drf-token --from-env-file=django-nodered-token -n receipt-promo`

`kubectl create secret generic django-users --from-env-file=django-init-secrets -n receipt-promo`

# Создание секрета из файла

`kubectl create secret generic google-key --from-file=borjomi.json=borjomi.json  -n receipt-promo`

# Создание конфига для gitlab container registry

`kubectl create secret docker-registry self-gitlab-registry --docker-server="https://gitlab.levelupdev.ru:5050" --docker-username="user" --docker-password="password" --docker-email="example@gmail.com" -o yaml --dry-run=client | kubectl apply -n receipt-promo -f -`

# Применение yaml файлов

`kubectl apply -f django.yaml`

`kubectl apply -f nodered.yaml`

`kubectl apply -f postgres.yaml`

`kubectl apply -f redis.yaml`

`kubectl apply -f ingress.yaml`

`kubectl apply -f django-rqworker.yaml`

`kubectl apply -f django-start-draw.yaml`

`kubectl apply -f django-start-promotion.yaml`

`kubectl apply -f django-job.yaml`


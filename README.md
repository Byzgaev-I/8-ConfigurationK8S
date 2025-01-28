# Домашнее задание к занятию «Конфигурация приложений»

## Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

## Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (MicroK8s)
2. Установленный локальный kubectl
3. Редактор YAML-файлов с подключенным GitHub-репозиторием

## Задание 1: Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap

1) Создать Deployment приложения, состоящего из контейнеров nginx и multitool.
2) Решить возникшую проблему с помощью ConfigMap.
3) Продемонстрировать, что pod стартовал и оба конейнера работают.
4) Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
5) Предоставить манифесты, а также скриншоты или вывод необходимых команд.

### Решение

1. Создан Deployment с двумя контейнерами:
    nginx (порт 80)
    multitool (порт 8080)

### Создаем файл deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
  labels:
    app: nginx-multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 80
```













































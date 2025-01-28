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
Применяем и видим конфликт портов. 

![image](https://github.com/Byzgaev-I/8-ConfigurationK8S/blob/main/1-1%20конфликт%20портов.png)

### 2. Исправление проблемы с помощью ConfigMap

### Создаем configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-multitool-config
data:
  nginx.conf: |
    events {
      worker_connections  1024;
    }
    http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;

      server {
        listen 80;
        server_name localhost;

        location / {
          root /usr/share/nginx/html;
          index index.html;
        }
      }
    }
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Welcome to Nginx</title>
    </head>
    <body>
        <h1>Welcome to Nginx on Kubernetes!</h1>
        <p>This page is served via ConfigMap</p>
    </body>
    </html>
```

### Обновляем файл deployment.yaml

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
          name: http
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: index-html
          mountPath: /usr/share/nginx/html/
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
          name: multitool
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-multitool-config
      - name: index-html
        configMap:
          name: nginx-multitool-config
```

### Применяем конфигурации

```bash
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
```

![image](https://github.com/Byzgaev-I/8-ConfigurationK8S/blob/main/1-2%20ConfigMap.png)

### Проверяем статус пода

![image](https://github.com/Byzgaev-I/8-ConfigurationK8S/blob/main/1-3%20статус%20пода.png)


Создание Service и проверка доступности веб-страницы

Создаем файл service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-service
spec:
  selector:
    app: nginx-multitool
  ports:
    - name: http
      port: 80
      targetPort: 80
    - name: multitool
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

### Применяем Service

```bash
kubectl apply -f service.yaml
kubectl get services
```
### Проверка доступности подов

```bash
kubectl get pods -o wide
```
### Проверка доступа к веб-странице через IP пода

```bash
curl 10.1.46.6:80
```
### Результат:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Nginx</title>
</head>
<body>
    <h1>Welcome to Nginx on Kubernetes!</h1>
    <p>This page is served via ConfigMap</p>
</body>
</html>
```

![image](https://github.com/Byzgaev-I/8-ConfigurationK8S/blob/main/1-4-5.png) 


### Проверка работы multitool

```bash
curl 10.1.46.6:80
```
### Проверка работы multitool:

```text
WBITT Network MultiTool (with NGINX) - nginx-multitool-6fdf78bb44-lfbmg - 10.1.46.6 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
```

![image](https://github.com/Byzgaev-I/8-ConfigurationK8S/blob/main/1-4-2.png)


### Использованные манифесты

### deployment.yaml

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
          name: http
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        - name: index-html
          mountPath: /usr/share/nginx/html/
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
          name: multitool
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-multitool-config
      - name: index-html
        configMap:
          name: nginx-multitool-config
```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-service
spec:
  selector:
    app: nginx-multitool
  ports:
    - name: http
      port: 80
      targetPort: 80
    - name: multitool
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-multitool-config
data:
  nginx.conf: |
    events {
      worker_connections  1024;
    }
    http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;

      server {
        listen 80;
        server_name localhost;

        location / {
          root /usr/share/nginx/html;
          index index.html;
        }
      }
    }
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Welcome to Nginx</title>
    </head>
    <body>
        <h1>Welcome to Nginx on Kubernetes!</h1>
        <p>This page is served via ConfigMap</p>
    </body>
    </html>
```
















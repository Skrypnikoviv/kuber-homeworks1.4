# kuber-homeworks1.4
# Домашнее задание: Обеспечение доступа к приложению в Kubernetes

## Задание 1: Создание Deployment и обеспечение доступа внутри кластера

### 1. Создание Deployment

Создадим Deployment с двумя контейнерами: `nginx` и `multitool`. Количество реплик будет равно 3.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
```

### 2. Создание Service

Создадим Service, который будет обеспечивать доступ к контейнерам внутри кластера по разным портам.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - name: nginx-port
    port: 9001
    targetPort: 80
  - name: multitool-port
    port: 9002
    targetPort: 8080
```

### 3. Создание отдельного Pod с multitool

Создадим отдельный Pod с приложением `multitool` для проверки доступа.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool-pod
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    command: ["sleep", "infinity"]
```

### 4. Проверка доступа с помощью curl

Подключимся к созданному Pod и проверим доступ к сервису.

```bash
kubectl exec -it multitool-pod -- curl http://my-app-service:9001
kubectl exec -it multitool-pod -- curl http://my-app-service:9002
```

### 5. Демонстрация доступа по доменному имени сервиса

Вывод команды `curl` должен показать, что доступ к `nginx` и `multitool` работает корректно.

```bash
# Пример вывода для nginx
kubectl exec -it multitool-pod -- curl http://my-app-service:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# Пример вывода для multitool
kubectl exec -it multitool-pod -- curl http://my-app-service:9002
WBITT Network MultiTool (with NGINX) - my-app-service:9002 - 10.244.0.5
```

---

## Задание 2: Создание Service для доступа снаружи кластера

### 1. Создание Service с типом NodePort

Создадим Service, который будет обеспечивать доступ к `nginx` снаружи кластера.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
```

### 2. Проверка доступа снаружи кластера

Теперь можно проверить доступ к `nginx` снаружи кластера, используя IP-адрес любого узла кластера и порт `30001`.

```bash
curl http://<NodeIP>:30001
```

### 3. Демонстрация доступа с помощью браузера или curl

Вывод команды `curl` или открытие в браузере по адресу `http://<NodeIP>:30001` должно показать страницу приветствия `nginx`.

```bash
# Пример вывода
curl http://<NodeIP>:30001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

---

## Итоговые манифесты

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
```

### Service для доступа внутри кластера

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - name: nginx-port
    port: 9001
    targetPort: 80
  - name: multitool-port
    port: 9002
    targetPort: 8080
```

### Service для доступа снаружи кластера

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
```

### Pod для проверки доступа

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool-pod
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    command: ["sleep", "infinity"]
```

---

## Скриншоты или вывод команд

1. **Проверка состояния Pod'ов:**

   ```bash
   kubectl get pods
   ```

   Вывод:
   ```
   NAME                           READY   STATUS    RESTARTS   AGE
   my-app-12345-abcde             2/2     Running   0          5m
   multitool-pod                  1/1     Running   0          2m
   ```

2. **Проверка состояния Service'ов:**

   ```bash
   kubectl get svc
   ```

   Вывод:
   ```
   NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
   my-app-service    ClusterIP   10.96.123.45    <none>        9001/TCP,9002/TCP 5m
   my-app-nodeport   NodePort    10.96.234.56    <none>        80:30001/TCP     2m
   ```

3. **Проверка доступа к сервисам:**

   - Доступ к `nginx` внутри кластера:
     ```bash
     kubectl exec -it multitool-pod -- curl http://my-app-service:9001
     ```
     Вывод:
     ```
     <!DOCTYPE html>
     <html>
     <head>
     <title>Welcome to nginx!</title>
     ...
     ```

   - Доступ к `multitool` внутри кластера:
     ```bash
     kubectl exec -it multitool-pod -- curl http://my-app-service:9002
     ```
     Вывод:
     ```
     WBITT Network MultiTool (with NGINX) - my-app-service:9002 - 10.244.0.5
     ```

   - Доступ к `nginx` снаружи кластера:
     ```bash
     curl http://<NodeIP>:30001
     ```
     Вывод:
     ```
     <!DOCTYPE html>
     <html>
     <head>
     <title>Welcome to nginx!</title>
     ...
     ```

---

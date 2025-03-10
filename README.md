# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1» - Скрыпников Илья

## Задание 1: Создание Deployment и обеспечение доступа внутри кластера

### 1. Создание Deployment

Создадим Deployment с двумя контейнерами: `nginx` и `multitool`. Количество реплик будет равно 3.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
  labels:
    app: nginx-multi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-multi
  template:
    metadata:
      labels:
        app: nginx-multi
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
 
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env: 
          - name: HTTP_PORT
            value: "8090"
```

### 2. Создание Service

Создадим Service, который будет обеспечивать доступ к контейнерам внутри кластера по разным портам.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
spec:
  selector:
    app: nginx-multi
  ports:
    - name: nginx
      protocol: TCP
      port: 9001
      targetPort: 80
    - name: multitool
      protocol: TCP
      port: 9002
      targetPort: 8090
```

### 3. Создание отдельного Pod с multitool

Создадим отдельный Pod с приложением `multitool` для проверки доступа.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
```

### 4. Проверка доступа с помощью curl

Подключимся к созданному Pod и проверим доступ к сервису.

```bash
kubectl exec -it multitool -- curl nginx-multitool-svc:9001
kubectl exec -it multitool -- curl nginx-multitool-svc:9002
```
![image](https://github.com/user-attachments/assets/370f9991-0614-4645-88c0-9dbc0c75181e)

![image](https://github.com/user-attachments/assets/3600f24b-cbeb-4201-aad9-ade39dd77667)

---

## Задание 2: Создание Service для доступа снаружи кластера

### 1. Создание Service с типом NodePort

Создадим Service, который будет обеспечивать доступ к `nginx` снаружи кластера.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc-np
spec:
  selector:
    app: nginx-multi
  ports:
    - name: nginx
      protocol: TCP
      port: 80
      nodePort: 30001
    - name: multitool
      protocol: TCP
      port: 8090
      nodePort: 30002
  type: NodePort
```

### 2. Проверка доступа снаружи кластера

Теперь можно проверить доступ к `nginx` снаружи кластера, используя IP-адрес любого узла кластера и порт `30001`.

![image](https://github.com/user-attachments/assets/d350f29e-e15e-4ebe-b377-748b19f3831f)

![image](https://github.com/user-attachments/assets/219dc90b-5145-4e11-83d6-a8a2e5c97f0e)



---


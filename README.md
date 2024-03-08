### Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1» Баранов Сергей

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.


[deployment-01.yaml]()

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-01
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx_multitool
  template:
    metadata:
      labels:
        app: nginx_multitool
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          value: "1180"
        - name: HTTPS_PORT
          value: "11443"
        ports:
        - containerPort: 1180
          name: mt-http
        - containerPort: 11443
          name: mt-https
```

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

```
apiVersion: v1
kind: Service
metadata:
  name: svc-01
spec:
  ports:
    - name: nginx
      port: 9001
      protocol: TCP
      targetPort: 80
    - name: miltitool
      port: 9002
      protocol: TCP
      targetPort: 1180
  selector:
    app: nginx_multitool
```

3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: multitool
  name: pod-multitool
spec:
  containers:
  - image: wbitt/network-multitool
    imagePullPolicy: IfNotPresent
    name: multitool
```

```
root@baranov:/home/baranovsa/kube-1.4# kubectl exec --stdin --tty pod-multitool -- /bin/bash
pod-multitool:/# curl svc-01:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
pod-multitool:/# 
pod-multitool:/# 
pod-multitool:/# curl svc-01:9002
WBITT Network MultiTool (with NGINX) - deployment-01-5c9cdc8d6d-vpcbt - 10.244.0.15 - HTTP: 1180 , HTTP>
pod-multitool:/# 
pod-multitool:/# 
pod-multitool:/# exit
exit
root@baranov:/home/baranovsa/kube-1.4#

```

4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.

```
root@baranov:/home/baranovsa/kube-1.4# kubectl exec --stdin --tty pod-multitool -- /bin/bash
pod-multitool:/# curl svc-01.default.svc.cluster.local:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
pod-multitool:/# 
pod-multitool:/# curl svc-01.default.svc.cluster.local:9002
WBITT Network MultiTool (with NGINX) - deployment-01-5c9cdc8d6d-vpcbt - 10.244.0.15 - HTTP: 1180 , HTTP>
pod-multitool:/# 

```

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.


Манифест [deployment]

Манифест [service]

Вывод команды:

```
root@baranov:/home/baranovsa/kube-1.4# kubectl exec --stdin --tty pod-multitool -- /bin/bash
pod-multitool:/# curl svc-01.default.svc.cluster.local:9001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
pod-multitool:/# 
pod-multitool:/# curl svc-01.default.svc.cluster.local:9002
WBITT Network MultiTool (with NGINX) - deployment-01-5c9cdc8d6d-vpcbt - 10.244.0.15 - HTTP: 1180 , HTTP>
pod-multitool:/# 

```


------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.

```
apiVersion: v1
kind: Service
metadata:
  name: svc-02
spec:
  type: NodePort
  ports:
    - port: 80
      protocol: TCP
      nodePort: 30001
  selector:
    app: nginx_multitool
```


2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

```
root@baranov:/home/baranovsa/kube-1.4# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP             7d20h
svc-01       ClusterIP   10.96.146.211   <none>        9001/TCP,9002/TCP   4m47s
svc-02       NodePort    10.110.2.245    <none>        80:30001/TCP        15s
root@baranov:/home/baranovsa/kube-1.4#
```
```
root@baranov:/home/baranovsa/kube-1.4# sudo curl localhost:30001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@baranov:/home/baranovsa/kube-1.4# 

```


3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.


Манифест [service]

Вывод команды:

```
root@baranov:/home/baranovsa/kube-1.4# sudo curl localhost:30001
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@baranov:/home/baranovsa/kube-1.4# 

```



------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

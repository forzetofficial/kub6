### 1. Выявленная проблема и её описание

1.  **Отсутствуют пространства имён (Namespaces):**
    - Манифест ссылается на `namespace: web` и `namespace: data`, которых нет в кластере. Это вызвало ошибку `Error from server (NotFound): namespaces "web" not found`.

2.  **Ошибка загрузки образа (ImagePullBackOff):**
    - Поды `web-consumer` не запускались из-за `ImagePullBackOff`. Образ `radial/busyboxplus:curl` недоступен в реестре.

3.  **Ошибка DNS-разрешения (Could not resolve host):**
    - После исправления образа, `web-consumer` не мог подключиться к `auth-db`, так как использовал короткое имя сервиса (`auth-db`), которое не разрешается за пределами своего namespace (`web`). Сервис находится в `namespace: data`.

### 2. Описание выполненных действий по исправлению

1.  **Созданы пространства имён:**
    ```bash
    kubectl create namespace web
    kubectl create namespace data
    ```
2.  **Заменён образ контейнера:**  
    Образ в Deployment web-consumer был заменён с недоступного radial/busyboxplus:curl на рабочий curlimages/curl:latest с помощью команды:

    ```bash
    kubectl set image deployment/web-consumer -n web busybox=curlimages/curl:latest
    ```

    Исправлена команда подключения:
    В Deployment web-consumer команда curl auth-db была заменена на полное доменное имя сервиса curl auth-db.data.svc.cluster.local, чтобы обеспечить корректное DNS-разрешение между пространствами имён. Для этого был создан и применён новый манифест web-consumer-fixed.yaml со следующим содержимым:

    ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: web-consumer
        namespace: web
        spec:
        replicas: 2
        selector:
        matchLabels:
        app: web-consumer
        template:
        metadata:
        labels:
        app: web-consumer
        spec:
        containers: - command: - sh - -c - while true; do curl auth-db.data.svc.cluster.local; sleep        5; done
        image: curlimages/curl:latest
        name: busybox
    ```

    Применение исправленного манифеста:

    ```bash
        kubectl apply -f web-consumer-fixed.yaml
    ```

[img](img/img1.png)

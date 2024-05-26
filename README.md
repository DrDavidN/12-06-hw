# «Хранение в K8s. Часть 1» - Дрибноход Давид

### Задание 1 

**Что нужно сделать**

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.

#### Ответ:

Создал новый namespace ```kubectl create namespace 12-06-hw```

Создал deployment.yaml с общей папкой sharevolume в busybox, каждые 5 секунд в файл test.txt будет дописываться сообщение "Test message", применил его

![image](https://github.com/DrDavidN/12-06-hw/assets/128225763/e3cfffbf-d42a-46db-acce-aaa8a4feb5be)

4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.

#### Ответ:
![image](https://github.com/DrDavidN/12-06-hw/assets/128225763/1d7010bb-b007-459b-8ed0-d7aaace6dead)


5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

#### Ответ:

``` YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: volumes-test
  namespace: 12-06-hw
spec:
  selector:
    matchLabels:
      app: volumes1
  replicas: 1
  template:
    metadata:
      labels:
        app: volumes1
    spec:
      containers:
      - name: busybox
        image: busybox:1.28
        command: ['sh', '-c', 'mkdir -p /sharevolume && while true; do echo "$(date) - Test message" >> /sharevolume/test.txt; sleep 5; done']
        volumeMounts:
        - name: volume
          mountPath: /sharevolume
      - name: multitool
        image: wbitt/network-multitool
        command: ['sh', '-c', 'tail -f /sharevolume/test.txt']
        volumeMounts:
        - name: volume
          mountPath: /sharevolume
      volumes:
      - name: volume
        emptyDir: {}

```
------

### Задание 2

**Что нужно сделать**

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.

#### Ответ:

Создал файл daemomset.yaml и применил его
![image](https://github.com/DrDavidN/12-06-hw/assets/128225763/2e91cef9-0f98-4255-8a3b-e11350552486)

Для обеспечения возможности чтения файла /var/log/syslog кластера MicroK8S внутри контейнера буду использовать параметр subPath, он позволит монтировать не всю директорию /var/log, а именно один файл syslog. Также буду использовать параметр readOnly, чтобы избежать проблем с доступом к файлу syslog, находящемуся на кластере MicroK8S.

3. Продемонстрировать возможность чтения файла изнутри пода.

#### Ответ:

![image](https://github.com/DrDavidN/12-06-hw/assets/128225763/510f9aba-29d2-42a0-b1bc-d7cd08ff3e5a)

4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

#### Ответ:

``` YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: test-daemonset
  namespace: 12-06-hw
  labels:
    app: multitool
spec:
  selector:
    matchLabels:
      name: test-daemonset
  template:
    metadata:
      labels:
        name: test-daemonset
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
        - name: logdir
          mountPath: /nodes-logs/syslog
          subPath: syslog
        - name: varlog
          mountPath: /var/log/syslog
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: logdir
        hostPath:
          path: /var/log
      - name: varlog
        hostPath:
          path: /var/log
```
------

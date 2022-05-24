# stavdnb_microservices
stavdnb microservices repository

### HW-22 KUBERNETES-3

## Полезное

Далее подготовим сертификат используя IP как CN
```$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=35.190.66.90"
```
И загрузит сертификат в кластер kubernetes
```
$ kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
```

### HW-21 KUBERNETES-2

## Полезное
как подключить YC 

```
yc managed-kubernetes cluster get-credentials test-cluster --external

 ~/ kubectl config current-context
yc-test-cluster
```



Как быстро посмотреть полученный нод порт

```
kubectl describe service ui  -n dev | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  32729/TCP
```


Forwarding ports for local test

```
kubectl port-forward ui-745497c4f5-j858n 9292:9292
```

Проверяем Endpoints
```
 ~/git/stavdnb_microservices/kubernetes/reddit/ [kubernetes-2*] kubectl describe service comment  | grep Endpoints
Endpoints:         10.244.1.5:9292
 ~/git/stavdnb_microservices/kubernetes/reddit/ [kubernetes-2*] kubectl describe service post  | grep Endpoints
Endpoints:         10.244.1.3:5000
```
Запускаем комманду nslookup из под POD post и убеждаемся , что теперь мы видим comment по имени, благодаря описанным файлам comment-service.yml post-service.yml , мы описываем DNS имена, в Docker Swarm аналог , network alias
```
 ~/git/stavdnb_microservices/kubernetes/reddit/ [kubernetes-2*] kubectl exec -ti post-758c64d668-qt6pr nslookup comment
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
nslookup: can't resolve '(null)': Name does not resolve

Name:      comment
Address 1: 10.106.76.66 comment.default.svc.cluster.local
```
Смотрим список опубликованных сервисов 
```
 ~/git/stavdnb_microservices/kubernetes/reddit/ [kubernetes-2*] minikube service list
|----------------------|---------------------------|--------------|-----|
|      NAMESPACE       |           NAME            | TARGET PORT  | URL |
|----------------------|---------------------------|--------------|-----|
| default              | comment                   | No node port |
| default              | comment-db                | No node port |
| default              | kubernetes                | No node port |
| default              | mongodb                   | No node port |
| default              | post                      | No node port |
| default              | post-db                   | No node port |
| default              | ui                        |         9292 |     |
| kube-system          | kube-dns                  | No node port |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |
| kubernetes-dashboard | kubernetes-dashboard      | No node port |
|----------------------|---------------------------|--------------|-----|
```
А список установленных расширений через команду 

```
minikube addons list
```

### HW-21 KUBERNETES-1


Install for Mac OS
```
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version --client
brew install minikube
```
Start Minicube 

```
minikube start --driver=docker
minikube config set driver docker
```
kubectl get nodes

install network plugin 

```
minikube start --network-plugin=cni
kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
```
apply manifest app-post
list pods
```
kubectl apply -f post-deployment.yml
kubectl get pods --all-namespaces
```

### HW-20 LOGGING-1
из полезного

управление компоуз файлом с отличным именем, и управление отдельным контейнером
```
docker-compose -f docker-compose-logging.yml up -d fluentd
```
при написании docker-compose image лучше использовать с версией во избежании проблем

Изучил как работают grok шаблоны


### HW-19 MONITORING-1

из полезного

АКТИВНЕЕ ИСПОЛЬЗОВАТЬ docker inspect

При подключении mongodb-exporter столкнулся с проблемой и долго не мог понять почему он не цепляется к нему по сети

```
command:
          - '--web.listen-address=:9216'
          - '--mongodb.uri=mongodb://post_db:27017'
```
Также нужно явно указывать версию, латест не понимает.

### HW-18 GITLAB-CI-1
из полезного можно отметить регистрацию раннера в контейнере докер 

```
docker exec -it gitlab-runner gitlab-runner register \
    --url http://<your-ip>/ \
    --non-interactive \
    --locked=false \
    --name DockerRunner \
    --executor docker \
    --docker-image alpine:latest \
    --registration-token <your-token> \
    --tag-list "linux,xenial,ubuntu,docker" \
    --run-untagged
```
Также необходимо уделить внимание заданию со звездочкой 

Так как у нас развернут свой гитлаб, то приложение гитлаб для слак мы установить не можем, и передаем инфу через веб-хуки. Строка запроса должна быть след. вида,обращаем внимание на ковычки(!)
```
curl -X POST --data-urlencode "payload={\"text\""":"" \"GITLAB SAY  BUILD-DsssssONE.\",}" https://hooks.slack.com/services/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
```
### HW-17 DOCKER-4

1) Позапускали несколько раз контейнеры с разным типом сетей, и посмотрели вывод на докер машине.
тип драйвера HOST, единственный кто использует сеть хоста , поэтому создает контейнер в 1 экземпляре. 
Bridge и none можно создавать в различных кол-вах/

```
 ~/ docker-machine ssh super-host sudo ip netns
9220260da77a (id: 2)
cac11da39435 (id: 1)
f2ff01ab2214 (id: 0)
ba581c4f427f
dc87c857a2f7
default
```
Подключение к доп сетям осуществляется через комманду network connect где мы указываем имя сети и подключаемый к ней контейнер
```
docker network connect front_net post
```
Базовое имя проекта образуется исходя из имени директории в которой запускается docker-compose.yml

Для того чтобы заменить имя проекта можно запускать с флагом -p PROJECT_NAME либо переопределить имена контейнеров **container_name: CONT_NAME** и имена сетей используя **name: NET_NAME**

Также если требуется использовать несколько сетей можно использовать алиасы 

```
networks:
      back_net:
        aliases:
           - comment_db
           - post_db
```



### HW-16 DOCKER-3

####  Создание Volume
Создаем том 
 ```
 docker volume create reddit_db
 ```
Монтируем том к папке с БД
 ```
 docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
 ```


Для выполнения задания со ЗВЕЗДОЧКОЙ необходимо 

Вешаем новый алиас
```
 ~/git/stavdnb_microservices/src/ [docker-3+] docker run -d --network=reddit \
--network-alias=post2 stavdock/post:1.0
9e64fd386b19ef59884eef2296e58c8732e90ceec59a11ca79f9a1378ecf9dbe
```
Переопредяляем переменные окружения с помощью ключа -e 
```
 ~/git/stavdnb_microservices/src/ [docker-3+] docker run -d -e POST_SERVICE_HOST=post2 COMMENT_SERVICE_HOST=comment2 --network=reddit \                                
-p 9292:9292 <your-dockerhub-login>/ui:1.0
```

Изменение размера с помощью образа alpine 

### HW-15 DOCKER-2

В ходе задания установлена docker-machine 
```
brew install docker-machine

docker-machine -v
docker-machine version 0.16.2, build bd45ab1

```
Далее с помощью YC создаем инстанс и инициализируем докер-хост с полученным IP
```
yc compute instance create \
  --name docker-host \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/.ssh/appuser.pub
	
docker-machine create \
  --driver generic \
  --generic-ip-address=62.84.126.58 \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/id_rsa \
  docker-host
```
C помощью этой команды переключаемся на удаленный докер-хост и обратно на локальный
```
eval $(docker-machine env docker-host)

eval $(docker-machine env --unset)

```
Затем собираем образ и пушим на докер хаб

```
docker build -t reddit:latest .
docker run --name reddit -d --network=host reddit:latest
docker ps
docker login
docker tag reddit:latest stavdock/otus-reddit:1.0
docker images
docker push stavdock/otus-reddit:1.0
```




**ПОЛЕЗНОЕ**

полезная команда отображения контейнеров  
```
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Ports}}"

CONTAINER ID   NAMES              PORTS
5d13e3984e55   great_blackwell
8e3ece67f36a   busy_hodgkin
47eb09f44976   hardcore_meitner   27017/tcp
5e520618d282   laughing_rhodes    27017/tcp
```
Точечное использование inspect 
```
docker inspect stavdock/otus-reddit:1.0 -f '{{.ContainerConfig.Cmd}}'
```

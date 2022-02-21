# stavdnb_microservices
stavdnb microservices repository

###  Создание Volume
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

HW-15 DOCKER-2

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

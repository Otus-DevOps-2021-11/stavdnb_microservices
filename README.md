# stavdnb_microservices
stavdnb microservices repository



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
  --ssh-key ~/.ssh/id_rsa.pub
	
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
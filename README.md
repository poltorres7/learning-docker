Learning Docker
=======

## Practica 1

Crear un sitio wordpress, eliminar containers y recuperar sitio web

#### 1. Create a mysql container

```bash
docker run --name some-mysql \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -d mysql:latest

## log inside container
docker exec -it some-mysql bash

# log to mysql console
mysql -u root -p
```

**Create the database**

```sql
create database wp;
```

#### 2. Create a wordpress container

```bash
docker run --name some-wordpress \
  -e WORDPRESS_DB_HOST=172.17.0.2 \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=my-secret-pw \
  -e WORDPRESS_DB_NAME=wp \
-p 8080:80 -d wordpress 
```

##### Simular perdida de containers

```bash
docker ps -q | xargs docker stop
docker ps -aq | xargs docker rm
```

#### 3.Recover docker container with volume

```bash
# Buscar volumes creados previamente
docker volume ls

# Correr mysql con volumen anterior
docker run --name some-mysql \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -v 09a6d460b824a4184cb64981678fb5574bdfd0d70fe37b5330ff5af2f99e97b6:/var/lib/mysql \
  -d mysql:latest

# Correr wordpress con volumen anterior
docker run --name some-wordpress \
  -e WORDPRESS_DB_HOST=172.17.0.2 \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=my-secret-pw \
  -e WORDPRESS_DB_NAME=wp \
  -v 620b80e71e49a5ed4d6c54b90a06252a84aa82ccc63c435e16c4d31e9f196f51:/var/www/html \
-p 8080:80 -d wordpress 
```

-------

## Practica 2


Crear un sitio wordpress, eliminar containers y recuperar sitio web - With a custom Network and custom volume


1. Create a docker network

```bash
# Create network
docker network create misitio
# list networks
docker network ls
```

2. Run mysql

```bash
# Correr mysql con volumen anterior
docker run --name some-mysql \
  -e MYSQL_ROOT_PASSWORD=abc123-abc \
  -v db_data:/var/lib/mysql \
  --network misitio \
  -d mysql:latest
```

3. Run wordpress

```bash
# Correr wordpress con volumen anterior
docker run --name some-wordpress \
  -e WORDPRESS_DB_HOST=172.19.0.2 \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=abc123-abc \
  -e WORDPRESS_DB_NAME=wp \
  -v wp_data:/var/www/html \
  --network misitio \
-p 8080:80 -d wordpress 
```

##### Simular perdida de containers

```bash
docker ps -q | xargs docker stop
docker ps -aq | xargs docker rm
```

#### Recupera sitio con nuevos containers reutilizando los volumes

Ejecuta paso 2
Ejecuta paso 3

---------

## Práctica 3

Crea 2 redes:

- public
- data

1. En la red de `public` crea un container de base de datos

```bash
docker run --name some-mysql \
  --network data \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  -d mysql:latest
```

2. En la red `public` crea un container de wordpress con los datos de conexión del container mysql creado en el paso anterior

```bash
docker run --name some-wordpress \
  --network public \
  --restart always \
  -e WORDPRESS_DB_HOST=172.19.0.2 \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=my-secret-pw \
  -e WORDPRESS_DB_NAME=wp \
  -d wordpress 
#  -p 8080:80 -d wordpress 
```

### Modify iptables

```bash
## public
br-6cee27a8696a 172.18.0.1/16

## data
br-cfb22152d5c8 172.19.0.1/16

## example
sudo iptables -I DOCKER-USER -i br-########1 -o br-########2 -j ACCEPT
sudo iptables -I DOCKER-USER -i br-########2 -o br-########1 -j ACCEPT


sudo iptables -I DOCKER-USER -i br-6cee27a8696a -o br-cfb22152d5c8 -j ACCEPT
sudo iptables -I DOCKER-USER -i br-cfb22152d5c8 -o br-6cee27a8696a -j ACCEPT


sudo route -n delete -net 172.18.0.0/16 192.168.0.1

sudo route -n add 172.18.0.0/16 192.168.106.2


sudo iptables -A FORWARD -i col0 -o br-6cee27a8696a -p tcp --dport 80 -j ACCEPT
sudo iptables -A FORWARD -i br-6cee27a8696a -o col0 -p tcp --sport 80 -j ACCEPT

sudo iptables -A FORWARD -i col0 -o br-6cee27a8696a -j ACCEPT
sudo iptables -A FORWARD -i br-6cee27a8696a -o col0 -j ACCEPT


```

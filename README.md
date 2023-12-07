# DOCKER NETWORKING

### Yanis Hamdaoui et Wassil Fahem
  ![image](https://github.com/Ynshmd/DOCKER-TP/assets/118398845/55f3b021-df6e-413b-8909-b2be8c7485c8)



## INTRODUCTION ![icons8-ampoule-globe-48](https://user-images.githubusercontent.com/118398845/214812403-1cdb1c93-4937-4550-89cd-e32e7aee91eb.png)



Dans ce TP nous allons voir comment utilisé prestashop pour déployer une application de commerce électronique.



# Prérequis  ![icons8-paramètres-de-l #39;ordinateur-portable-50](https://user-images.githubusercontent.com/118398845/214807221-e5cd379a-5e09-4045-a6ec-bc2588691783.png)



Dans ce prerequis nous allons installer les éléments necessaires pour l'execution du projet .Cette application utilise deux composants : un site web frontend et une base de données pour stocker des données persistantes.

## Installation 

### Installatioln des conteneures 

- installation prestashop
``````
docker pull prestashop/prestashop
``````

``````
docker network create prestashop-network
``````

- On doit avoir 2 conteneures car on souhaite les faire communiquer dans un premier temps dans le même réseau :

- Conteneur pour la base de donnés avec l'image PostgreSql stocké dans le réseau prestashop-netwoprk:
  
````
docker run -d --name containerbdd -d --privileged --network prestashop-network -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=pass -e POSTGRES_DB=dbtp -e ALLOW_EMPTY_PASSWORD= -v tp_volume:/var/lib/postgresql/data -p 5433:5432 -d postgres
````

- Conteneur prestashop dans le réseau prestashop-network avce l'image prestashop/prestashop:

``````
docker run --name prestashop-container -d --privileged --network prestashop-network -e PRESTASHOP_DATABASE_PASSWORD= -e ALLOW_EMPTY_PASSWORD=yes -d prestashop/prestashop
``````

- Executez cette commande pour lister les conteneurs qui sont activés: 

``````
docker ps
``````

- Lister tous les conteneurs qui ne sont pas activé 

``````
docker ps -a
``````

- A utiliser pour "allumer" le conteneurs si il est éteint . 

`````
docker start nom du conteneur
``````


## Configuration TASK1

- Pour connecter le conteneur de base de donnée au réseau prestashop-network. 

````
docker network connect prestashop-network containerbdd
````

- Pour connecter le conteneur de prestashop au réseau prestashop-network. 
````
docker network connect prestashop-network prestashop-container
````

- Pour inspecter le réseau format json: 

````
docker inspect prestashop-network
````


## Phase test TASK1

- On rentre dans le conteneur prestashop-conteneur

````
winpty docker exec -it prestashop-container bash
````

- Ensuite on installe toute les mise à jour 
````
apt-get update && apt-get upgrade
````

- installation de ping 
````
apt-get install ping
````

- on entre dans le conteneur "conteneurbdd"
````
winpty docker exec -it containerbdd bash
````

- on installe iputils 
````
apt-get install iputils-ping
````

- on ping  depuis contenairbdd vers prestashop-contenair pour savoir si les deux conteneurs commuique 
````
winpty docker exec -it containerbdd ping prestashop-container
````

- on entre dans la vm prestashop-bash 
````
winpty docker exec -it prestashop-container bash
````

- on install postgre-client
````
apt-get install postgresql-client
````

- 
````
psql -h containerbdd -U admin -d dbtp -W
````

- On crée une table nom , prénomm et l'age 
````
CREATE TABLE tp_table (id SERIAL PRIMARY KEY, name VARCHAR (50), Prenom VARCHAR (50), age INT);
````

- On entre les données dans la table crée ci-dessus 
````
INSERT INTO tp_table (name, prenom, age) VALUES ('Fahem','Wassil', 22);
````

- cette commande permet de voir les données présente sur la table 
````
select * from tp_table;
````

## TASK2

- on crée le conteneur gateway afin de faire la passerelle entre les 2 autres conteneur précédent : 
````
docker run -d --name gateway-container -d --privileged nginx:latest
````

````
docker network create --subnet 10.0.0.0/24 ynov-backend-network
````

````
docker network create --subnet 10.0.1.0/24 ynov-frontend-network
````

- 
````
docker network connect ynov-backend-network gateway-containe
````


````
docker network connect ynov-frontend-network gateway-container
````

````
docker inspect ynov-frontend-network
````

````
docker network connect ynov-frontend-network prestashop-container
````

````
docker network connect ynov-backend-network containerbdd
````

````
docker inspect ynov-frontend-network
````

````
docker inspect ynov-backend-network
````

````
 winpty docker exec -it -u root gateway-container bash
````


````
apt-get update && apt-get upgrade
````



````
apt-get install iproute2
````



````
 ip route add 10.0.1.0/24 via 10.0.0.2
````

````
 ip route add 10.0.0.0/24 via 10.0.0.2
````

````
apt-get install iputils-ping
````

````
ping 10.0.1.3
````


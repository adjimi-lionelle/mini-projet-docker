# Mini projet docker : student-list

Veuillez trouver les spécifications en cliquant [Ici](https://github.com/diranetafen/student-list.git "ici")

!["Crédit image : eazytraining.fr"](https://eazytraining.fr/wp-content/uploads/2020/04/pozos-logo.png) ![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

------------

Prénom : Lionelle

Nom: ADJIMI ADAMOU

Pour le 20e Bootcamp DevOps d'Eazytraining

Période : juillet-août-septembre

Lundi 01 juillet 2024

LinkedIn : https://www.linkedin.com/in/lionelle-adjimi-adamou-01a782217/


## Application

Nous devions déployer une application appelée "student_list" pour POZOS, qui permet d'afficher la liste des étudiants avec leurs âges. Cette application se compose de deux modules :

- Un module API REST, nécessitant une authentification de base, qui fournit la liste des souhaits des étudiants à partir d'un fichier JSON.
- Un module d'application web, écrit en HTML et PHP, permettant aux utilisateurs finaux d'obtenir une liste des étudiants.


Notre tâche consistait à :

1) Créer un conteneur pour chaque module.
2) Assurer leur interaction correcte.
3) Mettre en place un registre privé.


## Description du travail effectué

Je commencerai par vous décrire les six fichiers du projet et leur rôle respectif.

Ensuite, je vous expliquerai comment j'ai conçu et testé l'architecture pour justifier mes choix.

Enfin, je vous présenterai le processus de déploiement proposé pour l'application.

### Rôle des fichiers
Dans la livraison, vous trouverez trois fichiers principaux :un Dockerfile , un docker-compose.yml et un docker-compose.registry.yml

- docker-compose.yml : Utilisé pour démarrer l'application, comprenant à la fois l'API et l'application web.
- docker-compose.registry.yml : Permet de lancer le registre local et son interface utilisateur (frontend).
- Dockerfile : Sert à créer l'image Docker de l'API en intégrant le code source.
- requirements.txt : contient tous les packages à installer pour exécuter l'application
- student_age.py : Contient le code source de l'API en Python.
- student_age.json : Fichier JSON qui stocke les noms des étudiants et leurs âges.
- index.php : Page PHP où les utilisateurs finaux peuvent se connecter pour interagir avec le service et consulter la liste des étudiants avec leurs âges.

### Contruction et tests
Construire et tester
Étant donné que vous venez de cloner ce référentiel, vous devez suivre ces étapes pour préparer l'application « student_list » :

1) Se placer dans le dossier contenant notre Dockerfile et créer l'image de l'API :
cd ./mini-projet-docker/simple_api
docker build . -t student_age_api
docker images

![image-1](https://github.com/user-attachments/assets/766f0503-635e-418d-bfa0-e5d1bc8798b4)



2) Créer un réseau de type bridge afin que les deux conteneurs puissent communiquer entre eux en utilisant leurs noms grâce aux fonctionnalités DNS.


```bash
docker network create student_list.network --driver=bridge
docker network ls
```
![image-2](https://github.com/user-attachments/assets/c75b5892-c121-4e2d-8c4b-4200f4fbe2fd)


3) Retournez au répertoire racine du projet et lancez le conteneur d'API backend en utilisant ces arguments :

```bash
cd ..
docker run --rm -d --name=api.student_list --network=student_list.network -v ./simple_api/:/data/ api.student_list.img
docker ps
```
![image-3](https://github.com/user-attachments/assets/bb5abfbb-efdb-4eee-a5b6-7f7db6810bba)


Comme vous pouvez le voir, le conteneur backend de l'API écoute sur le port 5000. Ce port interne est accessible depuis un autre conteneur du même réseau, c'est pourquoi j'ai choisi de ne pas l'exposer.

De plus, j'ai monté le répertoire local ./simple_api dans le répertoire /data du conteneur afin que l'API puisse accéder au fichier student_age.json.

![image-4](https://github.com/user-attachments/assets/973d848d-d384-4d94-a0ab-8a03611477d5)



4) Mettre à jour le fichier index.php :

Avant d'exécuter le conteneur du site Web, vous devez mettre à jour la ligne suivante pour que api_ip_or_name et le port correspondent à votre déploiement :
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Grâce aux fonctionnalités DNS de notre réseau de type bridge, nous pouvons facilement utiliser le nom du conteneur API et le port spécifié précédemment pour configurer correctement notre site Web.

```bash
sed -i s\<api_ip_or_name:port>\api:5000\g ./website/index.php
```
![image-5](https://github.com/user-attachments/assets/bb16523c-191c-47a2-86a4-f1af71a4d8e7)



5) Lancez le conteneur de l'application Web frontend :
Les informations de connexion contenant le nom d'utilisateur et le mot de passe sont disponibles dans le fichier `simple_api/student_age.py.`

![image-6](https://github.com/user-attachments/assets/4d49d39c-2507-4cdc-9852-9c175df3d21d)


```bash
docker run --rm -d --name=website -p 86:80 --network=student_network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php:apache
docker ps
```
> ![7-docker ps](https://user-images.githubusercontent.com/101605739/224591443-344fd2cd-ddbc-4780-bbc5-7cc0bdac156f.jpg)


6) Tester l'API via le frontend :

  6-a Utilisation de la ligne de commande :

La commande suivante demandera au conteneur frontend de solliciter l'API du backend et d'afficher le résultat. Cela permettra de vérifier à la fois le bon fonctionnement de l'API et la capacité du frontend à obtenir la liste des étudiants depuis celle-ci.

```bash
docker exec website curl -u toto:python -X GET http://api:5000/pozos/api/v1.0/get_student_ages
```
![image-1](https://github.com/user-attachments/assets/7dabcc59-5129-473d-9ac6-e0d0044e05d3)



6-b Utilisation d'un navigateur web avec l'adresse IP:86 :

-  Si vous exécutez l'application sur un serveur distant ou une machine virtuelle (par exemple, provisionnée par le fichier Vagrant d'eazytraining), trouvez votre adresse IP en tapant hostname -I.

![image-2](https://github.com/user-attachments/assets/bdbf917e-d8ad-4109-b833-cca490cbd526)


- Si vous utilisez votre machine local, tapez localhost:86
Dans mon cas j'utilise une machine virtuelle provisionnée par le fichier Vagrantfile d'eazytraining.

![image-4](https://github.com/user-attachments/assets/8ab3cb37-13e5-4fac-9c61-5adbce132061)


Ensuite cliquez sur le bouton List Student.

![image-3](https://github.com/user-attachments/assets/60c4ff1d-5c7b-419e-acd5-60452be4dac5)



7) Nettoyez l'espace de travail :

Grâce à l'argument --rm que nous avons utilisé lors du démarrage des conteneurs, ceux-ci seront automatiquement supprimés lors de leur arrêt. Il vous suffit maintenant de supprimer le réseau créé précédemment.


```bash
docker stop api
docker stop website
docker network rm student_network
docker network ls
docker ps
```

![image-5](https://github.com/user-attachments/assets/8688a32b-48e4-41df-9d0c-1a62e7d072b7)




## Deployment via docker-compose

Une fois les tests réussis, nous pouvons maintenant « composeriser » notre infrastructure en plaçant les paramètres docker run dans un fichier docker-compose.yml au format Infrastructure as Code.

![image-6](https://github.com/user-attachments/assets/c838dd12-99ac-42c4-8c08-26b9aa33d73f)


As the tests passed we can now 'composerize' our infrastructure by putting the `docker run` parameters in ***infrastructure as code*** format into a `docker-compose.yml` file.

1) Exécutez l'application (API + site web)

Étant donné que l'image de l'application a déjà été créée, il vous suffit maintenant de lancer les conteneurs avec la commande suivante : 

```bash
docker-compose up -d
```
![image-8](https://github.com/user-attachments/assets/0a862149-8c68-4cc4-9f64-b1e9438ff124)


Docker Compose permet de spécifier l'ordre de démarrage des conteneurs. Le conteneur d'API démarrera en premier, car j'ai indiqué que l'application web dépend de celui-ci via l'option depends_on.

Nous pouvons accéder à l'application via le navigateur en tapant: 
192.168.56.3:86 où 192.168.56.3 représente l'adresse ip de la machine virtuelle que j'utilise.

![image-9](https://github.com/user-attachments/assets/aa087067-ae8a-48f7-b227-33599acc2d2c)



2) Créez un registre et son interface :

Nous avons utilisé l'image registry:2.8.1 pour le registre back-end et joxit/docker-registry-ui:2 pour l'interface graphique. J'ai également configuré quelques variables d'environnement.

![image-10](https://github.com/user-attachments/assets/76ec1d72-208d-4f87-9ee7-029dea4ab63b)



Par exemple, nous pourrons supprimer des images du registre via l'interface graphique.

```bash
docker-compose -f docker-compose.registry.yml up -d
docker ps
```
![image-12](https://github.com/user-attachments/assets/0b769ed8-bb01-46a2-b0c9-4e208a13d928)


Nous pouvons accéder à l'interface frontale du registre via le navigateur en tapant:  192.168.56.3:8082

![image-11](https://github.com/user-attachments/assets/0cf683aa-af4a-4497-b674-5304818caf02)




3) Poussez une image sur le registre et testez l'interface graphique
- créez un tag de l'image : 
```bash 
docker tag 89803ed32624 localhost:5000/student_age_api:local
docker images
```
![image](https://github.com/user-attachments/assets/bc706b99-8d3c-4d52-9510-e3b766df1fd1)


- Poussez l'image sur le registre :
```bash 
docker push localhost:5005/student_age_api:local
```
![image-14](https://github.com/user-attachments/assets/d265a190-1356-4f2f-8d39-6044145b8418)

![image](https://github.com/user-attachments/assets/9c4cddbe-0567-4be9-8205-f06db79d1394)

![Capture d’écran du 2024-08-21 23-28-33](https://github.com/user-attachments/assets/1a457b1c-e919-4be5-b797-fd4451b0ee75)



------------


## Ceci conclut mon rapport d’exécution du mini-projet Docker. 
Au cours de ce projet, aborder plusieurs notions sur docker notamment la création d'image docker, la configuration des volume et réseaux, le déploiement avec docker-compose et enfin la création des registres.

Cette expérience fut très enrichissante, me permettant de comprendre et d'acquerir des compétences dans la configuration des microservices. 







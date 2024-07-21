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


6) Test the api through the frontend :

6a) Using command line :

The next command will ask the frontend container to request the backend api and show you the output back.
The goal is to test both if the api works and if frontend can get the student list from it.

```bash
docker exec webapp.student_list curl -u toto:python -X GET http://api.student_list:5000/pozos/api/v1.0/get_student_ages
```
> ![8-docker exec](https://user-images.githubusercontent.com/101605739/224593842-23c7f3a5-e5bc-4840-a6af-2eda0f622710.png)


6b) Using a web browser `IP:80` :

- If you're running the app into a remote server or a virtual machine (e.g provisionned by eazytraining's vagrant file), please find your ip address typing `hostname -I`
> ![9-hostname -I](https://user-images.githubusercontent.com/101605739/224594393-841a5544-7914-4b4f-91fd-90ce23200156.jpg)

- If you are working on PlayWithDocker, just `open the 80 port` on the gui
- If not, type `localhost:80`

Click the button

> ![10-check webpage](https://user-images.githubusercontent.com/101605739/224594989-0cb5bcb7-d033-4969-a12e-0b2aa9953a97.jpg)


7) Nettoyez l'espace de travail :

Grâce à l'argument --rm que nous avons utilisé lors du démarrage des conteneurs, ceux-ci seront automatiquement supprimés lors de leur arrêt. Il vous suffit maintenant de supprimer le réseau créé précédemment.


```bash
docker stop api
docker stop website
docker network rm student_network
docker network ls
docker ps
```
> ![11-clean-up](https://user-images.githubusercontent.com/101605739/224595124-3ea15f42-e6d5-462a-92a0-52af7c73c17a.jpg)




## Deployment

As the tests passed we can now 'composerize' our infrastructure by putting the `docker run` parameters in ***infrastructure as code*** format into a `docker-compose.yml` file.

1) Run the application (api + webapp) :

As we've already created the application image, now you just have to run :

```bash
docker-compose up -d
```

Docker-compose permits to chose which container must start first.
The api container will be first as I specified that the webapp `depends_on:` it.
> ![12-depends on](https://user-images.githubusercontent.com/101605739/224595564-e010cc3f-700b-4b3e-9251-904dafbe4067.png)

And the application works :
> ![13-check app](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/2002c41f-6590-4491-b571-65de7ba1457e)


2) Create a registry and its frontend

I used `registry:2` image for the registry, and `joxit/docker-registry-ui:static` for its frontend gui and passed some environment variables :

> ![14-gui registry env var](https://user-images.githubusercontent.com/101605739/224596117-76cda01c-f2f6-4a18-862f-95d56449f98a.png)


E.g we'll be able to delete images from the registry via the gui.

```bash
docker-compose -f docker-compose.registry.yml up -d
```
> ![15-check gui reg](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/99f182f1-bc73-458c-8b29-207a681a31fd)


3) Push an image on the registry and test the gui

You have to rename it before (`:latest` is optional) :

> NB: for this exercise, I have left the credentials in the **.yml** file.

```bash
docker login
docker image tag api.student_list.img:latest localhost:5000/pozos/api.student_list.img:latest
docker images
docker image push localhost:5000/pozos/api.student_list.img:latest
```

> ![16-push image to registry](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/4dbff256-72ae-4c6e-b076-65e705828f28)

> ![17-full reg](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/fbc9cd2b-ec4b-4211-ba26-1a454936b204)

> ![18-full reg details](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/3c89e9cc-f1f4-42f8-bea4-4e3ec1672cbe)


------------


# This concludes my Docker mini-project run report.

Throughout this project, I had the opportunity to create a custom Docker image, configure networks and volumes, and deploy applications using docker-compose. Overall, this project has been a rewarding experience that has allowed me to strengthen my technical skills and gain a better understanding of microservices principles. I am now better equipped to tackle similar projects in the future and contribute to improving containerization and deployment processes within my team and organization.

![octocat](https://myoctocat.com/assets/images/base-octocat.svg) 






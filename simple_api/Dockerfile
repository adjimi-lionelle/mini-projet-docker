# Utiliser une image de base CentOS 7.6
FROM python:3.8-buster

# Mainteneur
LABEL maintainer="lionelleadjimi@gmail.com"


# Mettre à jour et installer les dépendances nécessaires avec un miroir alternatif
RUN apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y

# Copier le fichier requirements.txt dans le répertoire racine
COPY requirements.txt /

# Installer les packages nécessaires
RUN pip3 install -r /requirements.txt

# Copier le code source de l'API dans le conteneur
COPY student_age.py /
COPY student_age.json /data/student_age.json

# Créer un dossier de données et le déclarer comme volume
VOLUME /data

# Exposer le port 5000 pour l'API
EXPOSE 5000

# Commande pour exécuter l'application
CMD ["python3", "/student_age.py"]

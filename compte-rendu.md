# TP1 Docker

Docker était préalablement installé directement sur mon ordinateur (windows 10). Je n'ai donc pas utilisé de VMs.

On commence par pull alpine avec la commande "docker pull alpine"
## 1. Container postgres
### 1.1. Création du container postgres
#### 1.1.1. Création du Dockerfile avec l'image de postgres
On crée un fichier Dockerfile qui va contenir la conf de l'image de postgres
à l'intérieur on y ajoute ces lignes: 

`FROM postgres:11.6-alpine`
    
On peut également y rajouter `ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd` , ça va permettre de définir les crédentials pour accéder à postgres. Mais cette solution n'est pas optimal car les credentials sont écris en dur dans un fichier texte ce qui pose des problèmes de sécurité.
#### 1.1.2. Build du Dockerfile
On build ce fichier pour obtenir un container avec postgres à l'intérieur: `docker build . -t thomasbeaud/postgres`
#### 1.1.3. Création du network partagé
Ensuite on crée un network pour que le adminer puisse accéder au container de postgres: `network create app1-network`
#### 1.1.4. Lancement du container postgres
On maintenant lancer notre container postgres: `docker run --name postgres-app1 -e POSTGRES_PASSWORD=pwd -e POSTGRES_USER=usr -e POSTGRES_DB=db --network app1-network -d thomasbeaud/postgres`
- On ne précise pas `-p 5432:5432` car on ne veut pas que notre image soit accessible de l'extérieur mais pas l'adminer
- On peut ne pas préciser certaines informations comme ` -e POSTGRES_PASSWORD=pwd` si ils sont précisé dans le fichier de conf. Mias pour des raisons de sécurité il est plus prudent de ne pas les stocker et donc de les passer directement lors du run !
- L'option -d permet de garder la container lancé même si on ferme le terminal
- le nom du container est intéressant à remplir car il va nous servir a retrouver le container et également à nous connecter à postgres (contrairement à l'ID qui change à chaque redémarrage, le nom ne change pas)
#### 1.1.5. Lancement du adminer pour accéder au container de postgres
Etant donné que je n'ai pas de SQL client, je vais lancer un adminer: `docker run -p 8080:8080 --network app1-network --name adminer-app1 adminer`

Dorénavant lorsque que je me rend sur http://localhost:8080 je peux me connecter à postgres. Pour cela je dois rentrer les bons crédentials que j'ai rempli dans le Dockerfile. Dans la case 'serveur' je rentre 'postgres-app1' pour que l'adminer sache qu'il doit utiliser ce container (il peut car ils sont dans le même réseau) !

Pour supprimer un container il faut faire `docker rm -f nom_du_container|ID_du_container`
Des fois il faut arrêter l'écoute sur le port pour qu'un autre container puisse écouter sur ce même port

### 1.2. Initialisation des bases de données
Pour faire en sorte que des scripts sql se lancent à l'initialisation de la base de donnée, on doit modifier le Dockerfile.
On rajoute les lignes: 

 - `COPY createSchema.sql /docker-entrypoint-initdb.d/` (Qui contient des création de tables SQL)
 - `COPY insertData.sql /docker-entrypoint-initdb.d/` (Qui contient des insertions de lignes dans des tables SQL)

tous les scripts trouvés dans "**docker-entrypoint-initdb.d**" seront exécutés à l'initialisation de la base de donnée.
On doit rebuild l'image postgres et relancer le container pour que les modification soient prisent en compte.

### 1.3. La persistance des données
En rajoutant l'option -v avec le bon dossier (path en mode absolu !), on peut sauvegarder les données enregistrées !
Ce qui nous donne cette commande pour lancer le container postgres: `docker run --name postgres-app1 -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pwd -e POSTGRES_USER=usr -e POSTGRES_DB=db --network app1-network -d thomasbeaud/postgres` 

Cette option permet d'attacher un volume au container postgres et donc de sauvegarder la bdd si ils y a des modifications.

## 2. Container JAVA

On crée notre Dockerfile pour l'image de java. On build l'image et on run, ça affiche "hello world" dans la console.
Au run, on ne doit pas mettre l'option -d car la tache doit s'exécuter au premier plan pour que l'on puisse voir le "hello world"

### 2.1. Multistage build

Un build en plusieurs étapes sert à mieux décomposer ce build. Il va permettre de rendre le Dockerfile plus lisible et plus facile à maintenir. Il va également permettre de séléctionner ce que l'on veut vraimeent et de garder à la fin seulement ce qui est nécessaire pour faire tourner notre application. On peut alors utilisé plusieurs instructions "FROM" dans un seul Dockerfile, ce qui nous permettra de build seulement un fichier (une image) et on aura toutes les ressources nécessaires!

### 2.2. backend connecté à la bdd
On modifie le fichier application.yml en complétant l'accès à la base de donnée

## 3. Http server
Ce server va nous permettre d'exposer une pageweb en local. Pour cela on va utiliser l'image httpd fournie par docker.

### 3.1 Reverse proxy
Sur ce server on va mettre en place un reverse proxy. on va rajouter ces lignes dans la configuration du server:
`ServerName localhost
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://backend:8080/
ProxyPassReverse / http://backend:8080/
</VirtualHost>`

Cela va permettre de mapper les requetes du back-end pour avoir un nom plus explicite et moins encombrant ! pour acceder aux departements on doit dorénavant faire "localhost/departments" au lieu de "localhost:8080/departments"


## 4. Docker-compose

Le docker-compose est très utile ! il évite d'écrire les commandes à la main, cela évite les erreurs et nous fait gagner énormémenet de temps. C4est un manager au dessus de docker quis'écrit sous la forme d'un fichier yml.IL va permettre entre autre de builder et de runner n'importe quel container.

## 5. Publish
Maintenant nous allons publier sur docker hub nos différentes images. Dans un premier temps il faut se connecter à docker avec  `docker login`. Ensuite pour publier les images nous utilisons la commande `docker push totobo47/postgres`par exemple. On peut également tagger l'image en rajoutant ":1.0"  pour que l'on utilise seulement la dernière version (`docker tag postgres totobo47/postgres:1.0`)


# TP2 CI CD
Tout d'abord nous allons push nos fichiers du TP1 dans le git. Pour cela nous faisons: `git init``git .......`
`git add -A`sur le dossier racine, puis `git commit -m "first commit`et enfin `git push origin main`

### CI
Dans ce TP nous voulons que notre code java soit testé et vérifié à chauqe fois qu el'on commit sur le git.
Pour cela on va utilsier la commande `mvn clean verify` de maven qui va nous pemettre de nettoyer la sortie du répertoire, builder le rojet et enfin vérifier la sortie.

Dans le fichier pom.xml, on remarque qu'il y a certaine dépendance qui s'appelent Testcontainers. C'est une librairie java qui supporte le test de JUnit. Il test notamment les accés aux bases de données. 

Nous devons donc appeler la commande `mvn clean verify` dans un fichier main.yml. Nous allons également créer un dossier .github à la racine du git.

### CD
Pour des raisons de sécurité, nous ajoutons les crédentials (le username et le token comme mdp) du docker hub dans des variables du github action
Ensuite on modifie le main.yml. Dorénavant chaque application se build et se push sur docker hub àacahque commit sur la branche main ! 

## 5. SonarCLoud
Pour avoir des statictics sur 












jre,jdk -> 	permet d'executer avec un fichier plus leger




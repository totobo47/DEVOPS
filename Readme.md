# DEVOPS
Le terme "Devops" désigne une philosophie qui a pour but de favoriser l'innovation continue en automatisant les process et en réorganisant les rôles. Pour cela, il existe plusieurs outils comme les outils de versionning de code, la CI/CD, les outils de monitoring/alerting. C'est également une philosophie qui permet d'améliorer la collaboration entre les équipes et de développer des applications de production robuste !
Elle repose sur 3 grands principes
 - les améliorations
 - le retour des utilisateurs
 - l'apprentissage et les expériences
 
Cette philosophie est basé sur de l'automatisation, des bonnes pratiques et du code ("Eveything as code": pipeline & deployment, environments, orchestration, infrastructure)

## Problématique
Comment faire pour  faciliter la maintenance et la déployabilité des applications tout en assurant une certaine sécurité et un cout faible ?
On a plusieurs solutions, on peut:
- La solution basique est de lancer toutes les applications dans un seul serveur, un seul environnement. On aurait donc toutes nos apps (front-end et back-end, les bdd, etc), éventuellement plusieurs versions des langages (si plusieurs app ne fonctionne pas sur la même version de Java par exemple)
- Avoir un serveur par app, de cette façon on isole bien les différentes apps, mais c'est très couteux pour l'entreprise et c'est dur à déployer
- Avoir une vm par app, On a toujours une très bonne isolation des apps, une meilleure déployabilité, c'est également moins couteux. Cependant, on a un os par vm donc c'est très couteux au niveau des performances du PC, il faudrait avoir un PC surpuissant si on a beaucoup d'apps. Les images sont aussi très lourdes
- Avoir des containers, de cette façon on a une très bonne isolation, c'est peu couteux, avec de bonnes performances, une excellente déployabilité avec la possibilité d'automatiser les tâches plus simple. On lance un seul OS qui contiendra des containers qui eux même contiendront un seul processus/ une seule app. On peut également choisir le port sur lequel l'app va être exposé ou non à l'extérieur


## Docker
C'est une plateforme open source qui permet de lancer certaines applications dans des conteneurs logiciels. Il permet de créer des images les plus petites possibles contenant tout le nécessaire pour que l'app puisse fonctionner. De plus, cette même image peut être utilisée à plusieurs endroits. Chaque conteneur possède un processus/une app 
Docker exécute les processus directement sur l'os, il y a donc très peu d'overhead, ils sont rapides à lancer, à arrêter et à recréer !
Docker gère également efficacement al couche d'isolation réseau (iptables, firewall, interfaces virtuelles)
Pour créer une container, on doit commencer avec une image de base appropriée (java, maven...) et officielle, on doit viser la plus petite taille d'image possible et fixer les versions.

|Dev| Ops |
|--|--|
| Je gère maintenant mon runtime| Je fais en sorte que l’infra marche|
| Je m’assure que tout est testé| Je mets en place et maintient le socle qui permet de tout orchestrer|
| Je suis responsable que le code marche |  |

## Git
Git est un VCS (Version Control System), un système pour gérer les différentes versions de fichiers.
L'algorithme utilisé pour le checksum est le SHA-1 (160 bit hashés).
Git est une solution collaborative, qui propse des commit immutabes (avec un historique) et permet de confilts.
- Fournie un dossier de travail avec la version actuelle
- 1 commit = 1 unit of change
- Snapshots (pas des diffs)
- dossier .git qui contient les Snapchots, commit ..
- git diff -> regarde la différence à l'intérieur des fichiers
- git status -> regarde les fichiers modifiés, ajouté, supprimé
- git statch -> revnenir à un ancien commmit
- git log

- git config --global user.name totobo
- git config --global user.email beaudthomas@gmail.com
- git init
- git add -A
- git commit -m "first commit"
- git branch -M main
- git remote add origin https://github.com/totobo47/test.git
- git push origin main

- .gitignore a remplir pour enlever des fichiers/dossiers d'un commit
- git checkout develop -> pour aler sur la branche develop

### Branches

## CI CD
Continuous Integration and Continuous Delivery
Pour automatiser, éviter de faire des erreurs et les choses plusieurs fois

# TPs
Finalité du module:
Avoir une appli qui tourne en ligne et qui respecte plusieurs critères. Pour cela nous allons:
- Mettre des applications dans des conteneurs avec docker
- Compiler, tester et déployer automatiquement les applis
- Voir et comprendre certaines statistiques comme la qualité avec SonarCLoud
- Provisionner un serveur avec ansible

# TP1 Docker

Docker était préalablement installé directement sur mon ordinateur (windows 10). Je n'ai donc pas utilisé de VMs.
Dans ce TP on va utiliser un serveur HTTP (apache), une base de donnée (PostreSQL) et un API backend (en Java)

On commence par pull alpine avec la commande "docker pull alpine". C'est une des distributions linux.
## 1. Container postgres
### 1.1. Création du container postgres
#### 1.1.1. Création du Dockerfile avec l'image de postgres
On crée un fichier Dockerfile qui va contenir la conf de l'image de postgres.
à l'intérieur on y ajoute cette ligne: 

`FROM postgres:11.6-alpine` Cela va permettre de récupérer l'image de postgres.
    
On peut également y rajouter `ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd` , ça va permettre de définir les crédentials pour accéder à postgres. Mais cette solution n'est pas optimal car les crédentials sont écrit en dur dans un fichier texte ce qui pose des problèmes de sécurité.
#### 1.1.2. Build du Dockerfile
On build ce fichier pour obtenir un container avec postgres à l'intérieur: `docker build . -t thomasbeaud/postgres`
#### 1.1.3. Création du network partagé
Ensuite on crée un network pour que le adminer puisse accéder au container de postgres: `network create app1-network`
#### 1.1.4. Lancement du container postgres
On maintenant lancer notre container postgres: `docker run --name postgres-app1 -e POSTGRES_PASSWORD=pwd -e POSTGRES_USER=usr -e POSTGRES_DB=db --network app1-network -d thomasbeaud/postgres`
- On ne précise pas `-p 5432:5432` car on ne veut pas que notre image soit accessible de l'extérieur mais uniquement par l'adminer
- On peut ommetre certaines options comme ` -e POSTGRES_PASSWORD=pwd` si elles sont précisé dans le fichier de conf. Mais pour des raisons de sécurité il est plus prudent de ne pas les stocker et donc de les passer directement lors du run !
- L'option -d permet de garder la container lancé même si on ferme le terminal
- le nom du container est intéressant à remplir car il va nous servir a retrouver le container (pour le supprimer par exemple) et également à nous connecter à postgres (contrairement à l'ID qui change à chaque redémarrage, le nom ne change pas)
#### 1.1.5. Lancement du adminer pour accéder au container de postgres
Etant donné que je n'ai pas de SQL client, je vais lancer un adminer: `docker run -p 8080:8080 --network app1-network --name adminer-app1 adminer`

Dorénavant lorsque que je me rend sur http://localhost:8080 je peux me connecter à postgres. Pour cela je dois rentrer les bons crédentials que j'ai rempli dans le Dockerfile. Dans la case 'serveur' je rentre 'postgres-app1' pour que l'adminer sache qu'il doit utiliser ce container (Attention, on doit bien préciser lors du run, que les deux app soient dans le même réseau, sinon l'adminer n'aura pas accès à la bdd) !

Pour supprimer un container il faut faire `docker rm -f nom_du_container|ID_du_container`

Attention, Il faut arrêter l'écoute sur le port pour qu'un autre container puisse écouter sur ce même port !

### 1.2. Initialisation des bases de données
Pour faire en sorte que des scripts sql se lancent à l'initialisation de la base de donnée, on doit modifier le Dockerfile.
On rajoute les lignes: 

 - `COPY createSchema.sql /docker-entrypoint-initdb.d/` (Qui contient des création de tables SQL)
 - `COPY insertData.sql /docker-entrypoint-initdb.d/` (Qui contient des insertions de lignes dans des tables SQL)

Tous les scripts trouvés dans "**docker-entrypoint-initdb.d**" seront exécutés à l'initialisation de la base de donnée.

Attention, on doit rebuild l'image postgres et relancer le container pour que les modification soient prisent en compte.

### 1.3. La persistance des données
En rajoutant l'option -v avec le bon dossier (path en mode absolu !), on peut sauvegarder les données enregistrées !
Ce qui nous donne cette commande pour lancer le container postgres: `docker run --name postgres-app1 -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pwd -e POSTGRES_USER=usr -e POSTGRES_DB=db --network app1-network -d thomasbeaud/postgres` 

Cette option permet d'attacher un volume au container postgres et donc de sauvegarder la bdd si ils y a des modifications.

## 2. Container Java
### 2.1. Basic
On crée notre Dockerfile pour l'image de java. On build l'image et on run, ça affiche "hello world" dans la console.
Au run, on ne doit pas mettre l'option -d car la tache doit s'exécuter au premier plan pour que l'on puisse voir le "hello world"

Le contenu du Dockerfile: 
> FROM openjdk:17-alpine3.13 
> COPY . . 
> RUN javac Main.java 
> CMD ["java", "Main"]

Les commandes:
`docker build . -t thomasbeaud/java`
`docker run --name java-app1 -p 8080:8080 --network app1-network -d thomasbeaud/java`

### 2.2. Multistage build

Un build en plusieurs étapes sert à mieux décomposer ce fameux build, l'optimiser. Il va permettre de rendre le Dockerfile plus lisible et plus facile à maintenir. Il va également permettre de sélectionner ce que l'on veut vraiment et de garder à la fin seulement ce qui est nécessaire pour faire tourner notre application. On peut alors utiliser plusieurs instructions "FROM" dans un seul Dockerfile, ce qui nous permettra de build seulement un fichier (une image) et on aura toutes les ressources nécessaires!

On se crée un compte sur le site de spring.io et on génère le projet avec les bonnes propriétés.
On ajoute le fichier greetingController fourni dans le dossier src\main\java\fr\takima\training\simpleapi\controller
On modifie le Dockerfile:
>#Build
>FROM maven:3.6.3-jdk-11 AS myapp-build #On importe l'image maven avec le jdk 11, et on lui donne un nom
>ENV MYAPP_HOME /opt/myapp #On définit une variable d'environnement
>WORKDIR $MYAPP_HOME #On change de dossier courant
>COPY pom.xml .	#On copie le fichier pom.xml dans le répertoire courant
>COPY src ./src #On copie le dossier src dans le dossier src de notre répertoire courant
>RUN mvn package -DskipTests #On exécute un package pour le télécharger
>RUN mvn dependency:go-offline #Cette ligne est utile si on a déjà téléchargé le package précédent (il va permettre de ne pas le retélécharger)
>#Run
>FROM openjdk:11-jre #on importe l'image maven avec le jre 11
>ENV MYAPP_HOME /opt/myapp On définit une variable d'environnement
>WORKDIR $MYAPP_HOME #On change de dossier courant
>COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar #On a seulement besoin des dépendances précédemment téléchargées, donc on part d'une image plus petite et on y rajoute les dépendances nécessaires du 1er FROM. C'est ce qui permet et fait toute la puissance du multistage build !
>ENTRYPOINT java -jar myapp.jar #On défnit l'image de départ

Les commandes:
`docker build . -t thomasbeaud/java-multistage`
`docker run --name java-multistage-app1 -p 8081:8080 --network app1-network -d thomasbeaud/java-multistage`

### 2.3. backend connecté à la bdd
On télécharge le.zip fourni, on y ajoute le Dockerfile du multistage
On ajoute le fichier greetingController fourni dans le dossier src\main\java\fr\takima\training\simpleapi\controller
On modifie le fichier src/main/ressources/application.yml en complétant l'accès à la base de donnée:

    spring:
	    jpa:
		    properties:
			    hibernate:
				    jdbc:
					    lob:
						    non_contextual_creation: true
		    generate-ddl: false
		    open-in-view: true
	    datasource:
		    url: jdbc:postgresql://localhost:5432/db
		    username: usr
		    password: pwd
		    driver-class-name: org.postgresql.Driver
	management:
		server:
		    add-application-context-header: false
    endpoints:
	    web:
		    exposure:
			    include: health,info,env,metrics,beans,configprops


Les commandes:
`docker build . -t thomasbeaud/java-bdd`
`docker run --name java-bdd-app1 -p 8082:8080 --network app1-network -d thomasbeaud/java-bdd`

## 3. Http server
Ce server va nous permettre d'exposer une pageweb en local. Pour cela on va utiliser l'image httpd fournie par docker.
On crée le Dockerfile avec la bonne image:

>FROM httpd:2.4

On crée également un index.html basique.

On build et run pour la première fois:
>docker build . -t thomasbeaud/http
>docker run --name http-app1 -p 80:80 thomasbeaud.http

Ensuite, on copie le fichier .conf de apache dans notre dossier courant:
>docker  cp http-app1:/usr/local/apache2/conf/httpd.conf .

On peut rajouter deux lignes dans notre Dockerfile:
>COPY . /usr/local/apache2/htdocs/ 
>COPY ./httpd.conf /usr/local/apache2/conf/httpd.conf #on utilise notre configuration d'apache et pas celle par défaut

### 3.1 Reverse proxy
Sur ce server on va mettre en place un reverse proxy. on va rajouter ces lignes dans la configuration du server (httpd.conf):
`ServerName localhost
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://backend:8080/
ProxyPassReverse / http://backend:8080/
</VirtualHost>`

Cela va permettre de mapper les requêtes du back-end pour avoir un nom plus explicite et moins encombrant ! 
Pour accéder aux départements on doit dorénavant faire "localhost/departments" au lieu de "localhost:8080/departments"

Attention, pour que le reverse proxy fonctionne, il faut décommenter quelques lignes dans le fichier httpd.conf. La ligne 123 pour activer le mod_proxy_html, la ligne 143 (mod_proxy), 146 (mod_proxy_http).


## 4. Docker-compose

Le docker-compose est très utile ! il évite d'écrire les commandes à la main, cela évite les erreurs et nous fait gagner énormément de temps. C'est un manager au dessus de docker qui s'écrit sous la forme d'un fichier yml. Il va permettre entre autre de builder et de runner n'importe quel container.
Tout d'abord il faut l'installer. Ensuite on crée un fichier docker-compose.yml et on le remplit. Voici les lignes les plus importantes:

	version: '3.3'
    services:
	    backend:
		    container_name: backend #On donne un nom au conteneur
		    build: ./java/api-bdd/simple-api #L'image à build pour ce service
		    networks: #Ce service a besoin d'être dans le même réseau que les autres apps
			    - app1-network
		    ports: #On définit le port sur lesquel on peut accéder au service
			    - 8080:8080
		    depends_on: #On controle l'ordre selon lesquel les services vont s'executer, celui ci s'excuteras après le service database
			    - database
	    database:
		    container_name: database
		    build: ./postgres
		    networks:
			    - app1-network
		    environment: #On définit les variables d'envirronement pour accéder à la bdd
			    - POSTGRES_DB=db
			    - POSTGRES_USER=usr
			    - POSTGRES_PASSWORD=pwd
		    volumes: #on définit le volume où va se sauvegarder la bdd
			    - /devops/postgres/data:/var/lib/postgresql/data
	    adminer:
		    container_name: adminer
		    image: adminer
		    networks:
			    - app1-network
		    ports:
			    - 8081:8080
		    depends_on:
			    - database
	    httpd:
		    container_name: httpd
		    build: ./http
		    ports:
			    - 80:80
		    networks:
			    - app1-network
		    depends_on:
			    - backend
	    frontend:
		    container_name: frontend
		    build: ./frontend
		    networks:
			    - app1-network
		    
    networks: #on crée un network pour que les différentes apps puissent se trouver
	    app1-network:
A chaque run du docker-compose, chacun des services vont s'executer (un build + un run).

Quelques commandes:
- Pour construire ou reconstruire les images des services dont le Dockerfile a été modifié on utilise la commande `docker-compose build`. Pour information, si l'on spécifie un nom d'un service après le mot build de la commande, celle ci va builder seulement ce service et pas tout le document yml.

- Pour vérifier la structure du document yml: `docker-compose config`

- Pour lancer les services du docker-compose on écrit `docker-compose up`. `docker-compose down`pour éteindre les services et supprimer les conteneurs.
## 5. Publish
Maintenant nous allons publier sur docker hub nos différentes images. Dans un premier temps il faut se connecter à docker avec  `docker login`. Ensuite pour publier les images nous utilisons la commande `docker push totobo47/postgres`par exemple. On peut également tagger l'image en rajoutant ":1.0"  pour que l'on utilise seulement la dernière version (`docker tag postgres totobo47/postgres:1.0`)

Mettre ses images en ligne est très utile car cela va permettre de les stocker, de pouvoir y accéder depuis d'autres ordinateurs, d'être utilisé par d'autres membres de l'équipe. 

# TP2 CI CD
Tout d'abord nous allons créer un repo git:

    git init
    git add -A #Si on est placé sur le bon dossier (la racine)
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/totobo47/devops.git
    git push -u origin main

## 1. CI
Dans ce TP nous voulons que notre code java soit testé et vérifié à chaque fois que l'on commit sur le git.
Pour cela on va utiliser la commande `mvn clean verify` de maven qui va nous permettre de nettoyer la sortie du répertoire, builder le projet et enfin vérifier la sortie.

Dans le fichier pom.xml, on remarque qu'il y a certaine dépendance qui s'appelle "Testcontainers". C'est une librairie java qui supporte le test de JUnit. Il test notamment les accès aux bases de données. 

Tout d'abord, nous allons effectuer la commande `mvn clean verify`manuellement. Pour cela on se place dans le répertoire où se trouve le fichier pom.xml et on exécute la commande.

Maintenant, nous allons faire notre première CI. Pour cela crée un dossier à la racine: `mkdir ./.github/workflows`. Puis un fichier à l'intérieur: `main.yml`. Ce fichier aura plusieurs jobs. On en a un pour chaque conteneur que nous avons créé. Chaque job se fait en parrallèle.
Mon main.yml:

    name: CI devops 2022 CPE
    on:
	    #to begin you want to launch this job in main and develop
	    push:
		    branches:
			    - main
			    - develop
		    pull_request:
    jobs:
	    test-backend:
		    runs-on: ubuntu-20.04
		    steps:
			    #checkout your github code using actions/checkout@v2.3.3
			    - uses: actions/checkout@v2.3.3
			    #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
			    - name: Set up JDK 11
			      uses: actions/setup-java@v2
			      with:
				    distribution: 'zulu'
				    java-version: '11'
				#finally build your app with the latest command
			    - name: Build and test with Maven
			      run: mvn clean verify
			      working-directory: java/simple-api
Ensuite on commit. Les tests vont se faire automatiquement à chaque commit. Pour vérifier le bon déroulement du test et sa validation, on se rends sur github dans l'onglet "action". Si tout se passe bien, le test se passe bien et devient vert ! C'est gagné!

## 2. CD
Le but de cette partie est d'automatiser le build à l'intérieur de la pipeline dans github action et l'automatisation de la publication des images dans docker hub.
On ajoute deux repository secrets dans gituhb (sur le projet DEVOPS, aller dans settings/secrets/action). Ce sont les crédentials (le username et le mot de passe (token que l'on crée dans docker hub)) du docker hub. En effet pour pouvoir pusher les images dans docker hub, il faut être connecté, il faut donc que l'on connaisse les identifiants. Dans notre main.yml, on rajoute ces lignes dans le job test-backend:

	- name: Login to Docker Hub
	  uses: docker/login-action@v1
	  with:
		username: ${{ secrets.DOCKERHUB_USERNAME }}
		password: ${{ secrets.DOCKERHUB_PASSWORD }}

Ensuite on modifie main.yml en créant un nouveau job qui va builder et pusher les images dans docker hub:

	build-and-push-docker-image:
		needs: test-backend
		#run only when code is compiling and tests are passing
		runs-on: ubuntu-latest
		#steps to perform in job
		steps:
			- name: Checkout code
			  uses: actions/checkout@v2
			- name: Login to DockerHub
			  run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_PASSWORD }}
			- name: Build image and push backend
			  uses: docker/build-push-action@v2
			  with:
				#relative path to the place where source code with Dockerfile is located
				context: ./java/simple-api
				#Note: tags has to be all lower-case
			    tags: ${{secrets.DOCKERHUB_USERNAME}}/backend
			    push: ${{ github.ref == 'refs/heads/main' }}
			- name: Build image and push database
			  uses: docker/build-push-action@v2
			  with:
				#relative path to the place where source code with Dockerfile is located
				context: ./postgres
				#Note: tags has to be all lower-case
		        tags: ${{secrets.DOCKERHUB_USERNAME}}/postgres
			    push: ${{ github.ref == 'refs/heads/main' }}
			- name: Build image and push http
			  uses: docker/build-push-action@v2
			  with:
			    #relative path to the place where source code with Dockerfile is located
				context: ./http
			    #Note: tags has to be all lower-case
			    tags: ${{secrets.DOCKERHUB_USERNAME}}/http
			    push: ${{ github.ref == 'refs/heads/main' }}

## 3. SonarCloud
Sonar cloud est utile pour avoir des statistiques sur la qualité du code. Pour voir si il y a des erreurs de sécurité, des failles connues, pour être sur que le code est maintenable. C'est un outil qui aide à développer de meilleures apps.
Tout d'abord on se crée un compte sur sonarcloud.io. On peut ensuite facilement lier ce compte à notre compte Github.
Sur le site internet, on choisit le repo que l'on veut qui provient de github. Ensuite on crée une organisation, ce qui nous donne un clé d'accès.
On peut maintenant accéder aux tests de qualité de notre Code.

Pour l'automatiser, c'est à dire qu'à chaque commit, le etst de qualité est relancé, on modifie le main.yml. Au lieu de la commande "run: mvn clean verify", on fait:

    run: mvn -B verify sonar:sonar -Dsonar.projectKey=totobo47_DEVOPS -Dsonar.organization=totobo47 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}
    env:
	    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN}}
	    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN}}

Bien entendu, il a fallu, préalablement rajouter une repository secret dans github (SONAR_TOKEN), qui a été généré lors de la création de l'organisation dans sonarCloud.

Attention, il faut penser à désactiver l'option SonarCloud Automatic dans la méthode d'analyse du dossier administration dans le projet sur le site web de sonarCloud.

Au final, mon application a quelques erreurs (ce qui est normal) mais passe le test de qualité ! En effet je suis à un taux de ~89% !

# TP3 Ansible
le but de ce TP est d'installer et de déployer automatiquement nos apps.
## 1. Inventories
Après avoir installé ansible, on crée le fichier setup.yml dans le dossier (préalablement créé) ansible/inventories.
On y ajoute ces lignes:

    all:
	    vars:
		    ansible_user: centos
		    ansible_ssh_private_key_file: /home/totobo/Documents/id_rsa
	    children:
		    prod:
			    hosts: thomas.beaud.takima.cloud

On se place dans le dossier ./ansible et on lance la commande `ansible all -i inventories/setup.yml -m ping`
## 2. Facts
Les facts de ansible permettent d'avoir des informations sur son système (@IP etc...)
par exemple, la commande suivante demande la distribution de  l'os: `ansible all -i inventories/setup.yml -m setup -a
"filter=ansible_distribution*"
`

Pour la suite du TP, on doit supprimer Apache de notre machine: `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent"
--become
`

Avec ansible, on décrit seulement l'état du serveur et ansible met à jour automatiquement les paquets etc, pour nous.

## 3. Playbooks
Les playbooks sont des fichiers yaml dans lesquels sont mentionnés toutes les tâches qu'Ansible doit exécuter. Notamment la configuration de l'environnement de production et le déploiement de l'app !

Toujours dans le dossier ansible, on va créer un fichier playbook.yml avec:

    - name: first-playbook
      hosts: all
      gather_facts: false
      become: yes
      tasks:
      - name: Test connection
		ping:
		#Install Docker
		- name: Clean packages
		  command:
		    cmd: dnf clean -y packages
		- name: Install device-mapper-persistent-data
		  dnf:
			name: device-mapper-persistent-data
			state: latest
		- name: Install lvm2
		  dnf:
			name: lvm2
			state: latest
		- name: add repo docker
		  command:
			cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
		- name: Install Docker
		  dnf:
			name: docker-ce
			state: present
		- name: install python3
		  dnf:
			name: python3
		- name: Pip install
		  pip:
		    name: docker
		- name: Make sure Docker is running
		  service: name=docker state=started
		  tags: docker

On exécute ce playbook avec la comande: `ansible-playbook -i inventories/setup.yml playbook.yml`
Maintenant, docker est installé sur notre serveur !

## 4. Rôles
Pour que le dossier ansible soit lisible et compréhensible, on va créer des rôles pour spliter les tâches à exécuter par ansible.
On va créer 5 rôles que l'on va renseigner dans le playbook.yml:

    roles:
	    - docker
	    - database
	    - app
	    - proxy
	    - frontend

Pour les créer on utilise la commande: `ansible-galaxy init roles/docker` pour le rôle "docker" par exemple. Cela va créer des dossiers dans ansible/roles.
Dans le rôle docker, dans le dossier tasks, on va créer un fichier main.yml on l'on va placer toute la partie tasks de notre playbook.yml (on la supprime donc du playbook.yml !). On commence notre factorisation !

## 5. Déploiement de l'app
Dans chaque autre rôle, on va créer le main.yml dans le dossier task. Ensuite on va les remplir selon ce qu'ils leur faut et leur configuration.

Par exemple pour le main.yml du rôle database (même principe pour l'app du back-end):

    - name: Run_database
      docker_container:
        name: database
        image: totobo47/postgres
        networks:
		    - name: app1-network

rôle network:

    - name: Create a network
    	docker_network:
    		name: app1-network

rôle proxy:

    - name: Run_HTTPD
      docker_container:
	    name: httpd
	    image: totobo47/http
	    networks:
		    - name: app1-network
	    ports:
		    - "80:80"
Si on exécute le playbook (`ansible-playbook -i inventories/setup.yml playbook.yml`), l'application sera déployé sur internet!

## 6. Front-end
On va rajouter un peu de design à notre site déployé !
Pour cela, on télécharge le front sur le github fourni.
Ensuite on doit modifier plusieurs fichiers.
Tout d'abord, dans le httpd.conf du proxy http, on rajoute `Listen 8080`pour pouvoir écouter le front-end. On y rajoute également:

    <VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://frontend:80/
    ProxyPassReverse / http://frontend:80/
    </VirtualHost>
Pour que le proxy connaisse le front-end.

Ensuite, dans ansible, on va créer un nouveau rôle "frontend" et y ajouter cette tache:

    - name: Run_frontend
	  docker_container:
		name: frontend
	    image: totobo47/frontend
	    networks:
		    - name: app1-network
	    pull: true
On oublie pas de rajouter le rôle dans notre playbook.yml

Ensuite, on rajoute `pull: true`dans toutes les taches des rôles de ansible.
Dans la tache du proxy dans le rôle de ansible, on ajoute également le port 80:80

Enfin, dans le main.yml du dossier ./.github.workflows on ajoute un job, qui va servir à builder et push l'image du front-end.

On peut désormais commit, ce qui a pour effet de builder l'image et de les publier sur dockerhub. Et on lance la commande ansible, ce qui va déployer l'appli sur les serveurs. On a maintenant accès à notre site avec un joli front-end !





# résolution du bug (hacker russe) 
Pour résoudre cette énigme, on a du exécuter la commande ansible pour lancer le déploiement. Une erreur apparait avec une commande pour la résoudre. Il suffit de la copier coller, de l'executer.
Enfin, on commit pour rebuild les images et les publier. et on relance la commande ansible. Le tour est joué!





# Liens utiles
## TP1
https://hub.docker.com/_/httpd
https://hub.docker.com/_/openjdk
https://hub.docker.com/_/postgres
https://hub.docker.com/_/adminer/
https://docs.docker.com/develop/develop-images/multistage-build/
https://start.spring.io/
https://github.com/Mathilde-lorrain/simple-api.git
http://httpd.apache.org/docs/2.4/getting-started.html
https://httpd.apache.org/docs/2.4/en/howto/reverse_proxy.html
https://docs.docker.com/compose/install/
https://docs.docker.com/compose/
https://hub.docker.com/

## TP2
https://sonarcloud.io/

## TP3
https://docs.docker.com/install/linux/docker-ce/centos/
https://docs.ansible.com/ansible/2.6/modules/docker_container_module.html#docker-container-module
https://docs.ansible.com/ansible/2.4/docker_network_module.html










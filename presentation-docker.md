# Qu'est ce que Docker ?

![](http://quentin-varquet.fr/articles/docker/img/docker.png)


Avant de commencer à parler de Docker, il est important de préciser ce qu'est une architecture virtualisée.

## Virtualisation

Commençons par la définition même de **virtualisation**

	La virtualisation consiste à faire fonctionner un ou plusieurs systèmes d'exploitation / applications comme un simple logiciel, sur un ou plusieurs ordinateurs - serveurs / système d'exploitation, au lieu de ne pouvoir en installer qu'un seul par machine. Ces ordinateurs virtuels sont appelés serveur privé virtuel (Virtual Private Server ou VPS) ou encore environnement virtuel (Virtual Environment ou VE).

Nous pouvons simplifier cette définition avec ce schéma :


<img src="http://yuml.me/diagram/scruffy;scale:110/class/[note: Quentin Varquet{bg:cornsilk}],[Serveur Physique]-[Serveur virtuel Windows Server 2012],[Serveur virtuel Windows Server 2012]-[Applications A  B + C],[Serveur Physique]-[Serveur virtuel Linux],[Serveur virtuel Linux]-[Applications D + E + F],[Serveur Physique]-[X serveurs],[X serveurs]-[X Applications ]" >

**Explications** 

Un serveur physique héberge:
	
* Un serveur virtuel avec pour système d'exploitation Linux Red Hat. Ce serveur héberge une application D, une application E et une application F.
* Un serveur virtuel avec pour système d'exploitation Windows Server 2012 qui héberge une application A, une application B et une application C.
* Plusieurs autres serveurs virtuels avec des systèmes d'exploitation différents et des applications différentes.


## Docker

note: Docker est un logiciel libre qui automatise le déploiement d'applications dans des conteneurs logiciels. Il permet la mise en œuvre de containers s'exécutant en isolation, via une API de haut-niveau. Construit sur des capacités du noyau Linux (surtout les cgroups et espaces de nommage), un container Docker, à l'opposé de machines virtuelles traditionnelles, ne requiert aucun système d'exploitation séparé et n'en fournit aucun

Contrairement à la virtualisation actuellement utilisée dans les entreprises, Docker ne créer pas des machines virtuelles mais des containers.

![](http://quentin-varquet.fr/articles/docker/img/docker_ship.jpg)

Ces containers partagent le noyau Linux du serveur hôte et isole complètement les services et processus à l'intérieur des containers. Les containers contiennent donc les applications uniquement et non le système d’exploitation.

### Comparaison machines virtuelles / containers

#### Machines virtuelles
![](http://quentin-varquet.fr/articles/docker/img/infra_virtual.png)




Chaque machine virtuelle contient donc les applications, les librairies, et également un système d'exploitation. L'ensemble nécessite beaucoup de ressources que ce soit en mémoire, et en espace de stockage.

#### Containers docker

![](http://quentin-varquet.fr/articles/docker/img/infra_docker.png)

Les containers contiennent uniquement une application ainsi que ses dépendances, cependant le noyau de l'hôte est partagé entre tous les containers. Ces derniers sont totalement isolés et indépendants des autres. 

Un des nombreux avantages de docker vient du fait qu'il est possible d'avoir plusieurs environnements sur un seul et unique serveur. Ces environnements sont totalement indépendants.

Par exemple, il est possible d'avoir les environnements suivants : 

* Un container web de développement.
* Un container web de recette.
* Un container web de production.

Ces containers fonctionneront exactement comme s'ils étaient sur des serveurs physiques différents. Nous reviendrons sur des exemples concrets plus tard.

## Quelques exemples

L'ensemble des containers existants sont accessibles sur le docker hub (https://hub.docker.com/). On y retrouve notamment les containers officiels, par exemple le container officiel de MySQL, de phpmyadmin, de debian, ...

Il est également possible de créer sa propre image et de la publier et donc de la partager sur le docker hub.

#### Quelques déploiements

Après avoir installé docker en suivant la procédure d'installation (https://docs.docker.com/v1.8/installation/) vous aurez la possibilité de déployer vos premiers containers.

Pour ces exemples, nous partirons du principe que le serveur physique a pour ip **10.20.30.40**

**Déployer un container MySQL** 

	 docker run --name container_mysql_dev -e MYSQL_ROOT_PASSWORD=mon_password_root -d MySQL:5.7.9

* **--name** : Permet de définir le nom du container
* **-e**  : Variable d'environnement, ici permet de définir le password root
* **-d** : Permet de définir la version de MySQL souhaitée

Avec cette commande, vous venez de déployer un container MySQL totalement opérationnel et configuré pour la base de données de votre application en moins de 5 secondes.

 
Vous pouvez comparer l'installation et la configuration de MySQL sur un serveur debian pour vous rendre compte du gain de temps réalisé : http://www.rackspace.com/knowledge_center/article/installing-MySQL-server-on-debian

**Déployer un container apache**

	docker run --name container_apache_dev -p 80:80 -d nimmis/apache



* **--name** : Permet de définir le nom du container
* **-p** : Permet de mapper le port 80 du container sur le port 80 du serveur physique. C'est à dire que pour accéder au serveur web, nous devons simplement depuis notre navigateur aller sur **http://10.20.30.40**. Si nous avions mappé le port 90 nous aurions dû accéder au serveur apache en tapant **http://10.20.30.40:90**

Votre serveur web apache est déployé.
Il est également possible de se connecter en SSH dans un container : **docker exec -it nom_container bash**

## Contraintes

#### Première contrainte

Nous avons donc deux containers de développements

<img src="http://yuml.me/diagram/scruffy;scale:110/class/[note: Quentin Varquet{bg:cornsilk}],[Serveur Physique - Linux]-[Container_mysl_dev],[Serveur Physique  Linux]-[Container_apache_dev]">

Comme dit précédemment, les containers sont totalement isolés et indépendants, ils ne communiquent donc pas entre deux.

Cependant Docker a pallié à cette problématique. Il s'agit d'un paramètre à ajouter lors du déploiement d'un container.

Reprenons le container apache


	docker run --name container_apache_dev -p 80:80 -d nimmis/apache --link container_mysql_dv:database_dev


* **--link container_mysql_dv:database** : On "link" le container apache au container **container_mysql_dv** en donner pour nom à cette liaison **database_dev**

Avec ce paramètre en plus, nos containers Apache et MySQL peuvent communiquer et échanger des informations. Par exemple avec un site wordpress sur le container apache, nous pouvons héberger sa base de données dans le container MySQL.

**Pour information** : Depuis la version 1.9 de Docker, il est possible de lier les containers directement avec leurs IP pour générer un réseau entre plusieurs containers.

Il existe beaucoup de paramètres utilisables lors du déploiement d'un container. Vous pouvez les retrouver ici : https://docs.docker.com/v1.8/reference/commandline/cli/


#### Deuxième contrainte

Si vous venez à supprimer par erreur votre container MySQL, vous perdrez l'ensemble des données qu'il contient et donc vos bases de données.

Pour éviter ces problèmes, il est possible de créer des " volumes " entre le serveur physique et le container. Par exemple, nous pouvons créer un volume entre le dossier **/var/lib/MySQL** du container et le dossier **/docker/data/MySQL** du serveur physique. Si le container est supprimé, les données sont conservées sur le serveur physique.

Plus d'information sur la documentation officielle de docker : https://docs.docker.com/engine/userguide/dockervolumes/

Vous n'êtes pas un spécialiste de la ligne de commande ? Ce n'est pas grave, Docker a développé un outil pour faciliter son utilisation, il s'agit de **Docker Compose** 


## Docker-Compose

![](http://quentin-varquet.fr/articles/docker/img/docker-compose.png)


Docker-compose est l'outil officiel développé par Docker. Il facilite grandement le déploiement des containers et surtout, il est plus simple de déployer des containers d'un même environnement en quelques secondes.

L'objectif principal est de gérer en une seule commande **docker-compose** le build, le démarrage et l'arrêt de plusieurs images et conteneurs simultanément.

Docker-compose permet donc de remplacer l'usage des commandes docker build et docker run par un fichier **docker-compose.yml** qui décrit les images à construire, tous les paramètres de démarrage des conteneurs ainsi que les liens entre ces conteneurs.

#### Exemples :

Reprenons nos deux containers apache et MySQL :

	docker run --name container_mysql_dev -e MYSQL_ROOT_PASSWORD=mon_password_root -d MySQL:5.7.9
	docker run --name container_apache_dev -p 80:80 -d eboraas/apache --link container_mysql_dv:database_dev

Pour l'utilisation de Docker-Compose, je vais prendre l'organisation que j'utilise sur mes serveurs physiques. Il s'agit de serveurs utilisant Red Hat linux enterprise.

	Racine_de_mon_serveur/docker/compose

Pour créer cette même architecture :

	cd / && mkdir docker && cd docker && mkdir compose && cd compose

Puis je créer le dossier avec le nom de mon application et son environnement, par exemple nous allons partir du principe que nous souhaitons déployer un site un serveur web de développement pour l'institut G4 :

	mkdir webg4_dev

Nous aurons donc arborescence suivante :

	/docker/compose/webg4_dev

Dans le dossier webg4_dev, nous créons un fichier **docker-compose.yml** ainsi qu'un dossier **data** et **www**

Contenu du fichier docker-compose.yml :


	apache: 
	    image: nimmis/apache
	    links:
	     - MySQL
	    volumes:
	      - ./www:/var/www/html
	    ports:
	     - "80:80"
	MySQL:
	    image: MySQL:5.7
	    environment:
	     - MYSQL_ROOT_PASSWORD=my_password
	     - MYSQL_DATABASE=institut_g4
	    volumes:
	      - ./data:/var/lib/MySQL


Puis nous pouvons créer ces containers. Pour cela il suffit de se rendre dans le dossier webg4_dev puis de lancer la commande **docker-compose up -d**

Avec cette commande, nos containers sont déployés, opérationnels et entièrement configuré

![](http://quentin-varquet.fr/articles/docker/img/deploiement.PNG)


Vous souhaitez créer un nouvel environnement avec exactement la même configuration que les containers **webg4dev_apache_1** et **webg4dev_mysql_1** ? Pas de soucis, il suffit de copier le contenu du dossier **webg4_dev** par exemple vers le dossier **webg4_recette**. Enfin il suffit de modifier les ports (impossible de lancer deux containers sur deux ports identiques). Par la suite il suffit de lancer **docker-compose up -d** dans le dossier webg4_recette  et nous avons notre nouvel environnement de recette qui est une copie de l'environnement de développement.

### Avantages

Comme vous avez pu le deviner, il y a plusieurs avantages à l'utilisation de docker au sein de votre système d'information :

* **Gain de temps** : En effet, le déploiement d'un serveur web avec une base de données MySQL ne prend que quelques secondes. Ce gain de temps est valable pour tout type d'application et ne peux pas être négligé par un manager.
* **Financier** : Grâce à Docker, vous pouvez avoir plusieurs environnements avec plusieurs applications sur un seul serveur physique. En temps normal il aurait été obligatoire de séparer les environnements de développement, de recette, et de production ce qui engendre des couts très important pour les entreprises que ce soit en terme de serveurs, de licences, de temps de mise en place/configuration des serveurs/applications, et de maintenance
* **Maintenance** : Il est plus simple pour une équipe de maintenir un seul serveur en condition opérationnel. Docker permet de séparer totalement les applications, librairies, et dépendances. Si l'un des containers a un problème, les autres continueront de fonctionner. Il n'y a pas de risques de conflits entre plusieurs librairies étant donné qu'un container contient uniquement les librairies dont l'application qu'il héberge a besoin.
* **Facilité de transfert de compétences** : Docker est une technologie simple à apprendre si on est curieux et motivé. La plus grande difficulté étant de se mettre à jour régulièrement sur les nouvelles versions car ces dernières ajoutent énormément de fonctionnalités.

## Retour d'expériences

Comment ai-je commencé à utiliser la technologie Docker ?

Petit rappel, je travaille au sein de la DSI de Lafarge France, dans le pôle infrastructure. Je suis rattaché aux équipes de production et également à l'architecte technique.

Mes missions principales consistent à maintenant la CMDB (Configuration Management Database, il s'agit du référentiel de la DSI) de l'entreprise, de réaliser ses montées de version, et d'être force de proposition pour y insérer de nouveaux besoins et mener l'ensemble des projets qui se rattachent à la CMDB. Je suis devenu l'un des référents de cette application. Je m'occupe également des outils de supervision (Centreon, nagvis)

Je participe également à la documentation du SI via l’application de méthodes d’Architecture et l’utilisation de la norme ArchiMate 2.1.

De plus, je dois étudier l'impact des nouveaux projets sur l'architecture du SI tout en participant à ces derniers.

Enfin je dois tenir une veille technologique constante pour l'entreprise afin d'être force de proposition.

Durant l'année 2015 nous avions un important projet de refonte de l'outil de supervision pour les équipements réseaux au niveau national.

Plusieurs semaines avant le lancement du projet, nous discutions avec mon tuteur [@jbsarrodie](https://twitter.com/jbsarrodie) de la technologie Docker et de ses avantages dès lors que nous avons eu le projet, nous avons décidé de nous lancer dans cette technologie.

Nous nous sommes formés pendant un mois complet puis avons commencé à déployer nos premières applications en production.

Nous sommes aujourd'hui les deux référents de toutes les applications utilisant Docker et nous souhaitons que Docker devienne un standard de l'entreprise.

Depuis plusieurs mois, nous avons déployés les applications suivantes sur un seul serveur, et nous n'avons pas encore eu le moindre problème :

* **CMDB Prod** : Container Tomcat + Container postgresql
* **CMDB Recette**: Container Tomcat + Container postgresql
* **CMDB Dev**: Container Tomcat + Container postgresql 
* **Doku Wiki** : Container Apache/Php
* **Pg Admin** : Container Apache/Php
* **Gitblit** : Container Postgresql
* **Portail Citrix** : Container Apache/Php
* **MySQL Interface** : Container MySQL en relation avec tous les containers pour nos interfaces
* **Icinga Prod** : Container Icinga + Container Nagvis + Container Grafana
* **Icinga Recette** : Container Icinga + Container Nagvis + Container Grafana
* **Icinga Dev** : Container Icinga + Container Nagvis + Container Grafana
* **Nginx Proxy** : Container Nginx Proxy

Il s'agit d'applications utilisées par plusieurs utilisateurs toute la journée. L'ensemble est hébergé sur un seul et unique serveur sans aucun problème de performance.

L'un des avantages de Docker que j'utilise très souvent est sa simplicité d'utilisation. En effet quand je souhaite tester un développement sur la CMDB mais que je ne souhaite pas spécialement travailler sur l'environnement de développement je peux créer un nouvel environnement en moins de 5 secondes, faire mes tests, puis supprimer mes containers une fois que j'ai terminé, un vrai bonheur !

Un autre avantage est sa facilité d'utilisation. Prenons un exemple que j'ai rencontré en entreprise. Au début du mois d'octobre, le siège social de Lafarge France où je travaille a eu une maintenance électrique. Tous les serveurs ont donc été coupés durant une journée.

Un ingénieur de production était chargé de relancer les serveurs et applications de production de toute la DSI le samedi soir. Grâce à une ligne de commande que je lui ai envoyée, il a pu relancer en quelques secondes l'ensemble de nos applications docker, sur tous les environnements.

Il s'agit d'une technologie que j'utilise depuis plusieurs mois lors de mes projets professionnels et personnels.


## Mes containers

Etant un grand partisan de l'open source et du partage, voici quelques-uns de mes fichiers docker-compose.yml pour les plus curieux. J'ajouterai des commentaires après ##

**Dossier core** : Il s'agit de ma boite à outils. Ce sont les containers que je déploie obligatoirement sur mes serveurs.

	nginxproxy: ## Cette image permet de définir un alias DNS en .xip.io pour vos containers. Au lieu d'avoir ip:port du container ça sera nom_application.ip.xip.io. C'est beaucoup plus simple, surtout quand vous commencez à avoir plusieurs containers. Plus d'informations ici https://github.com/jwilder/nginx-proxy
	  image: jwilder/nginx-proxy
	  ports:
	   - "80:80"
	   - "443:443"
	  volumes:
	   - ./nginx-proxy/vhost.d:/etc/nginx/vhost.d
	   - ./nginx-proxy/cache:/var/cache/nginx
	   - ./nginx-proxy/certs:/etc/nginx/certs
	   - /var/run/docker.sock:/tmp/docker.sock
	
	dockerui: ## Interface graphique pour administrer tous les containers/images d'un serveur. Par mesure de sécurité, il est conseillé de configurer l'authentification nginx. Plus d'informations ici https://github.com/crosbymichael/dockerui
	  image: dockerui/dockerui
	  privileged: true
	  ports:
	   - "9000:9000"
	  volumes:
	   - /var/run/docker.sock:/var/run/docker.sock
	  environment:
	   - VIRTUAL_HOST=dockerui.* ## L'application sera donc accessible à l'adresse suivante : dockerui.ip_serveur.xip.io

	sshdproxy: ## Permet de se connecter avec un tunnel SSH. Par exemple pour se connecter avec pg_admin sur une base de données postgresql
	  image: lsf/sshd-proxy
	  ports:
	   - "2222:22"
	  volumes:
	   - /var/run/docker.sock:/tmp/docker.sock
	  environment:
	   - AUTHORIZED_KEYS= ##CLE SSH
	  external_links:
	   - container_postgresql ## peut être n'importe quel type de container (MySQL, oracle, ...)


**Tomcat + Postgresql**

	tomcat:
	  image: tomcat:7
	  volumes:
	   - ./webapp:/usr/local/tomcat/webapps/ROOT
	   - ./log:/usr/local/tomcat/logs
	   - ./jdbc/MySQL-connector-java-5.1.34-bin.jar:/usr/local/tomcat/lib/MySQL.jar
	   - ./jdbc/postgresql-9.1-901.jdbc4.jar:/usr/local/tomcat/lib/postgresql.jar
	  environment:
	   - VIRTUAL_HOST=tomcat.*
	  links:
	   - pgsql:database
	
	pgsql:
	  image: postgres:9.3
	  volumes:
	   - ./pgdata:/var/lib/postgresql/data
	   - ./pgdump:/var/lib/postgresql/dump ## Permet de gérer les backups
	  environment:
	   - POSTGRES_PASSWORD=my_password
	  ports:
	   - 5432:5432


**MySQL + PhpMyAdmin**

	MySQL:
	    image: MySQL:5.7
	    volumes:
	     - ./database:/var/lib/MySQL
	    environment:
	     - MYSQL_ROOT_PASSWORD=my_root_password
	    external_links:
	     - mon_container_apache ## peut être n'importe quel autre container
	    ports:
	     - 3306:3306
	
	phpmyadmin:
	   image: corbinu/docker-phpmyadmin
	   ports :
	    - "8180:80"
	   environment:
	    - MYSQL_USERNAME=root
	    - MYSQL_PASSWORD=my_root_password
	    - VIRTUAL_HOST=phpmyadmin.*
	   links:
	    - MySQL



**Wordpress + MySQL + PhpMyAdmin**

	wordpress:
	    image: wordpress
	    links:
	     - MySQL
	    environment:
	     - VIRTUAL_HOST=template.*
	     - WORDPRESS_DB_PASSWORD=my_database_password
	    volumes:
	      - ./www:/var/www/html
	    ports:
	     - "8080:80"
	MySQL:
	    image: MySQL:5.7
	    environment:
	     - MYSQL_ROOT_PASSWORD=my_root_password
	     - MYSQL_DATABASE=wordpress
	    volumes:
	      - ./database:/var/lib/MySQL
	      - ./dump:/opt/dump
	
	phpmyadmin:
	   image: corbinu/docker-phpmyadmin
	   ports :
	    - "8180:80"
	   environment:
	    - MYSQL_USERNAME=root
	    - MYSQL_PASSWORD=my_root_password
	   links:
	    - MySQL
	

Pour ce dernier container, j'ai également créé un script qui permet de cloner un site wordpress sur plusieurs environnements tout en gérant les backups automatiques

Je mets régulièrement à jour mes docker-file en fonction des mises à jours de docker, pour les plus curieux n'hésitez pas à me contacter.

## Conclusion

Pour conclure, Docker va devenir une technologie incontournable dans l'exploitation des systèmes d'information. Cette technologie m'a très rapidement séduite et je souhaite me spécialiser d'avantage pour y orienter ma carrière professionnelle.

Les grandes entreprises comme Microsoft et Google commencent à s’intéresser sérieusement à Docker, Google a également commencé à participer au projet en proposant plusieurs applications, c'est une application incontournable aujourd'hui et elle le sera encore demain ! 

Je pourrai parler pendant plusieurs heures de docker, si vous avez des questions ou même des problèmes quant à son utilisation je serai revis de vous aider !

Je n'ai pas parlé de comment faire pour gérer les sauvegardes de nos bases de données et fichiers, si vous souhaitez savoir comment procéder ça sera avec plaisir que j’échangerai avec vous.

Mon twitter [@quentinvarquet](https://twitter.com/QuentinVarquet)

Mail : quentin.varquet@gmail.com
 
Un grand merci à Jean-Baptiste Sarrodie pour avoir nourri ma veille technologique !
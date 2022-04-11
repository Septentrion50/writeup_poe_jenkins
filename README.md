# Utiliser Jenkins avec une image Docker

> Ce projet provient d'un TP originellement écrit par
[fpicot31](https://github.com/fpicot31). Ce repo n'existe qu'à des fins
éducatives et est inspiré de [celui-ci](https://github.com/fpicot31/Jenkins-docker).

## Marche à suivre

### Variables d'environnement

Si vous avez suivi ce qui est indiqué dans la première partie du tutoriel,
vous avec dû créer un repo spécifique sur GitHub pour ce projet, ainsi qu'un
autre repo sur DockerHub. Nous allons avoir besoin de conserver votre nom
d'utilisateur DockerHub et le nom de votre repo sur DockerHub dans deux
variables d'environnement différentes. Rentrez SVP les commandes suivantes
dans le terminal où vous avez votre dossier de travail :

`export DHNAME=<Votre nom utilisateur DockerHub>`
`export DHREPO=<Votre nom de repo DockerHub>`

**Ne fermez pas le terminal avant d'avoir terminé le tutoriel, vous devriez à
nouveau rentrer ces variables.**

### Réseau

Nous devons d'abord créer un réseau pour notre container afin qu'il puisse
correctement fonctionner (nous allons avoir besoin d'accéder au service sur
le navigateur de notre machine - donc on doit mapper le port 80 de docker
vers notre port hôte 8080 par exemple). Ici nous créons un réseau appelé
"jenkins".

- `docker network create jenkins`

### Image

Une fois notre réseau paramétré, il nous faut télécharger l'image, lui accorder
les bons privilèges, lui préciser quel réseau elle doit utiliser, quels
certificats elle doit avoir, etc.

- `docker run --name jenkins-docker --rm --detach --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home --publish 2376:2376 docker:dind --storage-driver overlay2`

### Build

Nous devons maintenant construire l'image. La commande suivante permet de
le faire en utilisant le moteur blueocean 1.1 de Jenkins. Le fichier de
configuration utilisé pour le build est notre Dockerfile-Jenkins.

`docker build -t myjenkins-blueocean:1.1 -f Dockerfile-Jenkins .`

### Lancement de l'image

Le build est effectué, nous pouvons maintenant lancer l'image.

`docker run --name jenkins-blueocean --rm --detach --network jenkins --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 --volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro $DHNAME/myjenkins-blueocean:1.2` <!--Cette dernière ligne est peut être à changer pour une variable env-->


Pour vérifier que cela fonctionne, nous pouvons aller sur
`http://localhost:8080` et si nous avons une fenêtre qui nous demande un mot
de passe, nous pouvons exécuter la commande suivante pour l'obtenir :

`docker exec -it jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword`

### Pour tester seul nginx

`docker run -d --name montp -p 8081:80 $DHNAME/$DHREPO`


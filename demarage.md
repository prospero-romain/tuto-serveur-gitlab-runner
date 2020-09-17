# Demarage d'un serveur pour utiliser GitLab et GitLab Runner via Docker

### Le serveur :

Les capacités du serveur pour réaliser le document sont les suivantes : 
> - 2 vCore 
> - 12 Go
> - 50 Go SSD NVMe avec un Ubuntu 20.04 Server (en version 64 bits)

Adresses : 
> - IPv4 du VPS est : XX.XXX.XX.XXX
> - IPv6 du VPS est : XXXXX

## Préparation :

### Premiere connection :

En prenant compte que le serveur est à jour, que vous avez des connaisances en systeme et que vous disposez d'un nom de domaine et au moins 3 sous domaine .

- Pour établir une connexion   
> `$ ssh root@XXX.XXX.XXX.XXX`

- Installation de docker (source : https://docs.docker.com/engine/install/ubuntu/):

    - Purge des anciennes versions : 

> `$ sudo apt-get remove docker docker-engine docker.io containerd runc`

- Installation de Docker en utilisant le repo : 

    - Installation des outils : 

> `$ sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common`

- Ajoute de la clé GPG officiel (verifier si elle est à jour) :

> `$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

- Ajouter le repot 'stable : 

> `$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`

- Installation de docker : 

> `sudo apt-get install docker-ce docker-ce-cli containerd.io`

- verifier l'installation et la version : 

> `$ docker version`

- Installation de Nginx en proxy (source : https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_ubuntu) :

Pour installer nginx : 

> `$ sudo apt-get install nginx`

Il faut rajouter notre sous domaine pour gitlab dans la config de nginx  : 

> `cd /etc/nginx/conf.d && cat default.conf `


et rajouter : 


> server { 
    listen 80;
    server_name sous.domaine.xyz; 
    location / { proxy_set_header X-Real-IP $remote_addr; 
    proxy_set_header Host $http_host; 
    proxy_pass http://127.0.0.1:PORT; 
    } 
}

Ensuite il faut redemarrer Nginx : 

> `systemctl reload nginx`


- Installation de GitLab (source : https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-engine) : 

    - Preparation des volume :

        - GitLab utilise un volumes persistant situé ici :

         	![persistent_data_gitlab](assets\persistent_data_gitlab.PNG)


- Création des variable de path pour `/srv/gitlab` :

> `$ export GITLAB_HOME=/srv/gitlab`

- Installation de GitLab : 

> `sudo docker run --detach --hostname gitlab.chevalier-prospero.tech --publish 443:443 --publish 8080:8080 --publish 22:222 --name gitlab --restart always --volume $GITLAB_HOME/config:/etc/gitlab --volume $GITLAB_HOME/logs:/var/log/gitlab --volume $GITLAB_HOME/data:/var/opt/gitlab gitlab/gitlab-ee:latest`

- Pour suivre l'installation : 

> `docker logs -f gitlab`

Une fois l'installation finis rendez vous sur : gitlab.mondomaine.fr ou XXX:XXX:XXX:XXX:8080 pour mettre votre mot de passe.

L'ensemble des parametre de gitlab sont réunis dans le fichier `gitlab.rb`

pour l'éditer : `sudo docker exec -it gitlab editor /etc/gitlab/gitlab.rb`
 activier les modif : `sudo docker restart gitlab`


# Installation des runners gitlab dans un contenaire (souce : https://docs.gitlab.com/runner/install/docker.html):


- Création du volume docker : 

> `docker volume create gitlab-runner-config`

- Installation des runners via docker : 

> `docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest`

- pour lancer la configuration des runners : 

> `docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest register`

Note : rendez vous sur http://gitlab.mondomaine.fr/admin/runners pour trouver les informations demandés 




# Prochaine étape : conteneriser Nginx
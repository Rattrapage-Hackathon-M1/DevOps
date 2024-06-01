# DevOps

## Description

Ce répository contient les fichiers de configuration, la CI/CD et les scripts de déploiement pour le projet DevOps.
Il sera également utilisé pour la mise en place de NGINX et Docker Swarm.

## Les Services
### auth

Le service d'authentification est un service qui permet de gérer les utilisateurs et les rôles. Il est basé sur une API développée en Java Spring Boot et une base de données PostgreSQL.

### postgres_auth

Le service postgres_auth est une base de données PostgreSQL qui est utilisée par le service auth.

### api-crud

Le service api-crud est l'api développée également en Java qui est utilisée pour générer des tâches ou des utilisateurs.

### postgres_crud

Le service postgres_crud est une base de données PostgreSQL qui est utilisée par le service api-crud.

### nginx

Le service NGINX est un reverse proxy qui permet de rediriger les requêtes vers les différents services.

### front

Le service front est une application web développée en REACT qui permet d'interfacer les deux APIs.

## CI / CD

Chaque répository de l'application est configuré pour utiliser GitHub Actions pour la CI/CD. Les fichiers de configuration se trouvent dans le répository de chaque service.
Chaque repository contient un fichier `Dockerfile` qui permet de construire une image Docker de l'application.
Sur chaque repository, lors d'un push sur la branche `main`, une github action est déclenchée pour vérifier que l'applications build correctement et que les tests passent.

Sur chaque repository, lors d'un push d'un nouveau tag, une github action est déclenchée pour construire l'image Docker et la pousser sur le registry.

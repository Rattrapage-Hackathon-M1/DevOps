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


## Créer et configurer le Docker Swarm

**Étape 1 : Initialiser le Docker Swarm**

```sh
docker swarm init
```

Cette commande va initialiser un Docker Swarm sur votre machine actuelle. La sortie de la commande vous donnera un token à utiliser pour ajouter des nœuds supplémentaires au cluster Swarm.

**Étape 2 : Ajouter des nœuds (optionnel)**

Si vous avez d'autres machines et souhaitez les ajouter au Swarm, utilisez le token fourni par la commande précédente :

```sh
docker swarm join --token <TOKEN> <MANAGER_IP>:2377
```

### 2. Déployer les services avec Docker Compose

Créez un fichier nommé `docker-compose.yml` avec le contenu que vous avez fourni, puis utilisez la commande suivante pour déployer vos services dans le Swarm :

```sh
docker stack deploy -c docker-compose.yml your_stack_name
```

### 3. Configurer Prometheus pour superviser les services

**Étape 1 : Créer un fichier de configuration pour Prometheus**

Créez un fichier nommé `prometheus.yml` avec la configuration suivante :

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['node1:9090', 'node2:9090', 'node3:9090'] # Ajoutez ici les adresses IP et ports de vos nœuds Docker
    metrics_path: /metrics
```

**Étape 2 : Ajouter Prometheus à votre Docker Compose**

Ajoutez la configuration pour Prometheus et un exporter Node (node-exporter) dans votre fichier `docker-compose.yml` :

```yaml
version: "3.8"
services:
  postgres_auth:
    container_name: postgres_auth
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: postgres
    ports:
      - 5432:5432

  auth:
    image: ghcr.io/rattrapage-hackathon-m1/authentification:v1.1.8
    ports:
      - 8001:8080
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres_auth:5432/postgres
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=1234
    depends_on:
      - postgres_auth
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  postgres_crud:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: crud
    ports:
      - 5433:5432

  api-crud:
    image: ghcr.io/rattrapage-hackathon-m1/api-crud:v1.3.0
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres_crud:5432/crud
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=1234
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    depends_on:
      - postgres_crud

  nginx:
    image: nginx:1.25.5-alpine-slim
    ports:
      - 8000:80
    volumes:
      - "./nginx/nginx.conf:/etc/nginx/conf.d/default.conf"

  front:
    image: ghcr.io/rattrapage-hackathon-m1/frontend:v0.0.1
    ports:
      - "5173:5173"
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      replicas: 1

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
    deploy:
      mode: global
```

Ce fichier Docker Compose ajoutera Prometheus et node-exporter à votre déploiement. Node-exporter sera déployé sur chaque nœud du Swarm.

**Étape 3 : Déployer à nouveau les services**

Pour déployer les nouvelles configurations, utilisez la commande :

```sh
docker stack deploy -c docker-compose.yml your_stack_name
```


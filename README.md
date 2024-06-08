# TP Liaison SGBDR - Elasticsearch

## Partie 1: Installation et Configuration du SGBDR & Peuplement des Données

### 1. Choix de l'environnement

On choisit Docker pour faciliter la gestion de notre environnement de développement et déploiement. Docker permet de créer des environnements isolés et reproductibles.

### 2. Installation du SGBDR

On commence par créer un fichier `docker-compose.yml` pour définir les services nécessaires à notre environnement. On utilise l'image officielle de PostgreSQL pour le SGBDR.

```yaml
version: '3.1'

services:
  db:
    image: mysql:latest
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
        - ./data:/var/lib/mysql
```

On lance ensuite le service MySQL avec la commande `docker-compose up -d --build`.

### 3. Création de données fictives

On crée un fichier `data.sql` pour peupler la base de données avec des données fictives.

```sql
CREATE DATABASE IF NOT EXISTS db;
USE db;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name, email) VALUES ('Cyno', 'cyno@cyno.cyno');
```

### 4. Validation

On peuple la base de données avec les données fictives en exécutant le script SQL avec la commande `docker exec -i mysql mysql -u user -p db < data.sql`.

On peut maintenant vérifier que les données ont bien été insérées en se connectant à la base de données avec la commande `docker exec -it mysql mysql -u user -p db` puis en exécutant la requête SQL `SELECT * FROM users;`.

## Partie 2 : Synchronisation des données du SGBDR avec ElasticSearch

### 1. Choix de l'environnement

On choisit d'utiliser Logstash pour synchroniser les données entre le SGBDR et Elasticsearch.  
**Logstash** est un outil de traitement et d'ingestion de données qui permet de collecter, transformer et charger des données de différentes sources vers différentes destinations.

### 2. Installation de Logstash

On ajoute un service Logstash à notre fichier `docker-compose.yml`.

```yaml
  logstash:
    image: docker.elastic.co/logstash/logstash:7.17.3
    container_name: logstash
    volumes:
      - ./pipeline:/usr/share/logstash/pipeline
      - ./mysql-connector-java-8.0.23.jar:/usr/share/logstash/mysql-connector-java-8.0.23.jar
    depends_on:
      - elasticsearch
      - db
    ports:
      - "5044:5044"
```

On ajoute également le connecteur MySQL Java à notre projet en téléchargeant le fichier `mysql-connector-j-8.4.0.tar.gz` et en le plaçant dans le répertoire du projet.

### 3. Configuration de pipeline de données

On crée un fichier `pipeline/logstash.conf` pour définir le pipeline de données.

```conf
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://db:3306/db"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/usr/share/logstash/mysql-connector-j-8.4.0.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    statement => "SELECT id, name, email, created_at FROM users"
    schedule => "* * * * *" # Exécute toutes les minutes
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "users_index"
    document_id => "%{id}"
  }
  stdout { codec => json_lines }
}
```

### 4. Validation

On lance le service Logstash avec la commande `docker-compose up -d --build logstash`.

On peut vérifier que les données ont bien été synchronisées en consultant l'index `users_index` dans Elasticsearch avec la commande `curl -X GET "localhost:9200/users_index/_search?pretty"`.
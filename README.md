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
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydatabase
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

On lance ensuite le service PostgreSQL avec la commande `docker-compose up -d --build`.

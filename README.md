# TP Part 01 - Docker

Projet réalisé dans le cadre du TP DevOps (Takima) : mise en place d'une application **3-tiers** entièrement conteneurisée avec Docker, composée d'une base de données, d'une API backend et d'un serveur HTTP en reverse proxy.

## Architecture

```
                 ┌─────────────┐
   Client  ───▶  │   HttpServer │  (Apache httpd - reverse proxy)
                 └──────┬──────┘
                        │
                 ┌──────▼──────┐
                 │  simpleapi   │  (Spring Boot - API REST)
                 └──────┬──────┘
                        │
                 ┌──────▼──────┐
                 │   Database   │  (PostgreSQL)
                 └─────────────┘
```

Les trois services communiquent entre eux via un réseau Docker interne (`app-network`). Seul le `HttpServer` est exposé sur la machine hôte (port `80`), conformément aux bonnes pratiques de sécurité (la base de données et le backend ne sont pas accessibles directement depuis l'extérieur).

## Structure du projet

```
.
├── BackendAPI/         # Étape 1 : Hello World Java (JRE minimal + multistage build)
├── Database/            # Image PostgreSQL + scripts d'initialisation SQL
├── HttpServer/          # Image Apache httpd + configuration reverse proxy
├── simpleapi/           # API Spring Boot connectée à la base de données
├── docker-compose.yml   # Orchestration des 3 services
└── .gitignore
```

## Prérequis

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) (inclus avec Docker Desktop)

## Lancer le projet

Cloner le repo puis, à la racine :

```bash
docker compose up -d --build
```

Vérifier que les 3 conteneurs tournent :

```bash
docker compose ps
```

L'application est ensuite accessible sur :

```
http://localhost
```

Pour arrêter et supprimer les conteneurs :

```bash
docker compose down
```

Pour tout supprimer, y compris les volumes (données de la base) :

```bash
docker compose down -v
```

## Détails par composant

### Database (PostgreSQL)

- Image basée sur `postgres:17.2-alpine`.
- Initialisation automatique du schéma et des données via des scripts SQL placés dans `/docker-entrypoint-initdb.d` (exécutés en ordre alphabétique au premier démarrage).
- Les données sont persistées grâce à un volume Docker nommé, afin de survivre à la suppression du conteneur.

### simpleapi (Backend API)

- Application Spring Boot (Java 21) exposant une API REST au-dessus de la base de données (ex: `/departments/{code}/students`).
- Image construite avec un **multistage build** : une première étape avec un JDK compile le projet via Maven, une seconde étape avec un JRE seul exécute le `.jar` final — ce qui réduit significativement la taille de l'image publiée.
- La connexion à la base de données se fait via le nom du conteneur/service `database` sur le réseau interne (pas de port exposé à l'hôte).

### HttpServer (Apache httpd)

- Image basée sur `httpd:2.4-alpine`.
- Configuré en **reverse proxy** : toutes les requêtes reçues sur `/` sont transférées vers le backend (`mod_proxy` / `mod_proxy_http`), qui centralise ainsi l'accès à l'application derrière un point d'entrée unique.

## Commandes utiles

| Commande | Description |
|---|---|
| `docker compose up -d --build` | Build et démarre tous les services en arrière-plan |
| `docker compose logs -f [service]` | Affiche les logs en continu (tous les services, ou un seul) |
| `docker compose ps` | Liste les conteneurs du projet et leur statut |
| `docker compose down` | Arrête et supprime les conteneurs (conserve les volumes) |
| `docker compose down -v` | Arrête et supprime aussi les volumes |
| `docker exec -it <container> sh` | Ouvre un shell dans un conteneur en cours d'exécution |


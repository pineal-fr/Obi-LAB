## 📋 Mémento des Commandes et Configurations

Ce guide rassemble les commandes et configurations essentielles pour gérer, superviser et dépanner l'infrastructure Docker.

### 🚀 Gestion de l'Environnement Docker

  * **Lancer l'infrastructure** (avec le fichier d'environnement) :

    ```bash
    docker-compose --env-file wordpress.env up -d
    ```

  * **Lancer en reconstruisant les images** (après une modification) :

    ```bash
    docker-compose --env-file wordpress.env up -d --build
    ```

  * **Vérifier le statut des conteneurs en cours** :

    ```bash
    docker-compose ps
    ```

 * **Arrêter les conteneurs sans supprimer les volumes** (garde les données) :

    ```bash
    docker-compose down stop
    ```

 * **Relancer les conteneurs avec les sauvegardes** (garde les données) :

    ```bash
    docker-compose down start
    ```

  * **Arrêter les conteneurs et supprimer les volumes** (réinitialise les données) :

    ```bash
    docker-compose down -v
    ```

  * **Nettoyage complet du système Docker** (supprime conteneurs, volumes et images inutilisés) :

    ```bash
    docker system prune -a --volumes
    ```

-----

### 💻 Accès aux Services et Conteneurs

  * **Accéder à la base de données MySQL** :

    ```bash
    docker exec -it mysql mysql -u root -prootpassword
    ```

      * **Commandes SQL utiles** une fois connecté :
        ```sql
        SHOW DATABASES;
        USE my_database;
        SELECT User, Host FROM mysql.user;
        EXIT;
        ```

  * **Ouvrir un terminal (shell) dans un conteneur** (ici, le conteneur WordPress) :

    ```bash
    docker exec -it wordpress /bin/bash
    ```

-----

### 🎨 Configuration de Grafana

  * **Connexion** :

      * **URL** : `http://localhost:3000`
      * **Utilisateur** : `admin`
      * **Mot de passe** : `M9b!zT$vK@c60nF`

  * **Sources de Données (Data Sources)** :

      * **Prometheus** : `http://prometheus:9090` (normalement déjà provisionné)
      * **Loki** : `http://loki:3100` (à ajouter si non présent)

  * **Tableaux de Bord (Dashboards) à Importer** :

      * **`1860`** : **Node Exporter Full** - Métriques complètes du système (CPU, RAM, disque).
      * **`13639`** : **Loki / Promtail Dashboard** - Vue d'ensemble de la chaîne de logging.
      * **`3662`** : **Prometheus 2.0 Stats** - Supervision de Prometheus lui-même.

-----

### 🔍 Exemples de Requêtes de Supervision

À utiliser dans l'onglet **Explore** de Grafana ou directement dans Prometheus.

  * **Prometheus (PromQL)** :

      * **Vérifier que NGINX est bien supervisé** (doit retourner `1`) :
        ```promql
        up{job="nginx-exporter"}
        ```
      * **Calculer le pourcentage d'utilisation du CPU** :
        ```promql
        100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
        ```

  * **Loki (LogQL)** :

      * **Détecter la création d'un utilisateur WordPress** :
        ```logql
        {container_name="wordpress"} |= "POST /wp-admin/user-new.php"
        ```
      * **Surveiller les actions sensibles (création d'utilisateur WP ou SQL)** :
        ```logql
        {container_name=~"wordpress|mysql"} |~ "CREATE USER|POST /wp-admin/user-new.php"
        ```

-----

### 🐛 Dépannage (Troubleshooting)

  * **En cas de bug au redémarrage d'un conteneur** :
    1.  Arrêter et supprimer proprement l'environnement :
        ```bash
        docker-compose down -v
        ```
    2.  Forcer la suppression des données persistantes :
        ```bash
        rm -rf ./wordpress/*
        rm -rf ./db_data/*
        ```
    3.  Relancer l'infrastructure :
        ```bash
        docker-compose --env-file wordpress.env up -d
        ```
    4.  **Si l'erreur persiste**, supprimez et recréez les dossiers de volume :
        ```bash
        rm -rf ./wordpress ./db_data
        mkdir ./wordpress ./db_data
        ```

-----

### 🔧 Commandes Utilitaires

  * **Personnaliser l'apparence du terminal (Prompt)** pour une meilleure lisibilité :
    ```bash
    export PS1='\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]${PWD##*/}\[\e[0m\]\[\e[1;32m\]\$\[\e[0m\] '
    ```



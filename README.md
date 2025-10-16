# Infrastructure DevOps : WordPress, Nginx & Monitoring

Ce projet est une infrastructure complète, conteneurisée via **Docker Compose**, conçue pour la pratique des compétences DevOps, de la supervision (Observability) et de la gestion d'applications web.

Il déploie une stack web moderne (NGINX/PHP-FPM) avec une chaîne de monitoring complète (Prometheus, Loki, Grafana). Les credentials se trouvent dans le fichier `wordpress.env`; vous pouvez les garder ou les modifier.

-----

## 🌟 Fonctionnalités et Compétences Clés

| Domaine | Outils | Compétences acquises |
| :--- | :--- | :--- |
| **Conteneurisation** | Docker, Docker Compose | Maîtrise du déploiement multi-conteneurs, gestion des volumes persistants et des réseaux isolés. |
| **Architecture Web** | WordPress (PHP-FPM), NGINX, MySQL | Compréhension de l'architecture NGINX Reverse Proxy pour la performance. |
| **Monitoring (Métriques)** | **Prometheus**, **NGINX Exporter**, Node Exporter | Collecte et analyse des métriques de performance système (CPU, RAM) et applicatives (Trafic, requêtes). |
| **Logging (Logs)** | **Loki**, **Promtail** | Centralisation des logs de tous les conteneurs et maîtrise des requêtes **LogQL** pour le débogage et l'audit. |
| **Visualisation** | **Grafana** | Création de tableaux de bord personnalisés pour la surveillance temps réel. |
| **Sécurité/Audit** | MySQL General Log, Analyse des requêtes SQL | Apprentissage de l'audit et de la traçabilité des actions sensibles en base de données. |

-----

## 🎓 Un Laboratoire d'Apprentissage Complet

Au-delà d'un simple projet de déploiement, cette infrastructure est un véritable bac à sable pour développer des compétences variées :

  * **Apprentissage de WordPress :** Expérimentez avec l'administration, l'installation de thèmes et de plugins dans un environnement sécurisé et isolé. Idéal pour comprendre le fonctionnement interne de ce CMS sans risque.
  * **Maîtrise du SQL et de PhpMyAdmin :** En accédant directement à la base de données via phpMyAdmin, vous pouvez visualiser comment WordPress structure ses données. C'est une excellente occasion pour pratiquer le langage SQL, en exécutant des requêtes pour lister des utilisateurs, analyser des contenus ou comprendre les relations entre les tables.
  * **Analyse de Logs et Débogage :** Grâce à la centralisation des logs avec Loki, vous apprenez à traquer les erreurs, à surveiller les accès et à comprendre le comportement des applications. C'est une compétence essentielle pour le débogage et l'analyse de sécurité.
  * **Fondamentaux DevOps :** Manipulez une chaîne d'outils professionnels pour comprendre les principes de l'observabilité (monitoring et logging) et de l'Infrastructure as Code (IaC) avec Docker Compose.

-----

## ⚙️ Démarrage rapide de l'infrastructure

### 1. Prérequis

Assurez-vous d’avoir **Docker** et **Docker Compose** installés sur votre machine.

- [🧩 Télécharger VSCode](https://code.visualstudio.com/download)
- [🐧 Installer WSL (Windows)](https://learn.microsoft.com/fr-fr/windows/wsl/install)
- [🐳 Installer Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/)

---

### 2. Placement des Fichiers

1. Téléchargez le dossier complet du projet.  
   Déplacez-le dans le répertoire de votre choix, puis décompressez-le.  
   Ouvrez le dossier extrait avec **VSCode** ou tout autre éditeur de texte.  
   Vérifiez que tous les fichiers de configuration (`docker-compose.yaml`, `nginx.conf`, `prometheus.yml`, etc.) sont présents à la racine.
2. Assurez-vous que votre fichier **`wordpress.env`** est également à la racine.

---

### 3\. Lancement de la Stack

Lancez l'intégralité de l'infrastructure avec cette commande. Le flag `--build` est utilisé pour la première fois ou après des modifications du code source.

```bash
docker-compose --env-file wordpress.env up -d --build
```

> **Astuce :** Le flag `--env-file` est crucial car il charge vos secrets.

### 4\. Accès aux services

| Service | Adresse locale | Identifiants par défaut |
| :--- | :--- | :--- |
| **WordPress (Site)** | `http://localhost:8080` | (Installation via la page) |
| **phpMyAdmin** | `http://localhost:8081` | User: `user_wordpress` / Pwd: `${WORDPRESS_DB_PASSWORD}` |
| **Grafana** | `http://localhost:3000` | User: `${GRAFANA_ADMIN_USER}` / Pwd: `${GRAFANA_ADMIN_PASSWORD}` |
| **Prometheus (Métriques brutes)** | `http://localhost:9090` | N/A |

-----

## 🔬 Guide d'Apprentissage et d'Exercices Pratiques

Utilisez les requêtes ci-dessous dans l'onglet **Explore** de Grafana pour valider votre LAB.

### A. Apprendre le Monitoring (Prometheus)

  * **Valider la connexion NGINX :**
    ```promql
    up{job="nginx-exporter"}
    ```
    *(Doit retourner la valeur `1`.)*
  * **Mesurer le trafic en temps réel :**
    ```promql
    rate(nginx_http_requests_total[5m])
    ```
    *(Naviguez sur le site pour voir le graphique monter.)*
  * **Utilisation du CPU de la machine :**
    ```promql
    100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
    ```

### B. Apprendre le Logging (Loki)

  * **Rechercher les actions SQL sensibles :**
      * **Action :** Exécutez une requête `UPDATE` ou `INSERT` dans phpMyAdmin.
      * **Prompt LogQL :**
        ```logql
        {container_name="mysql"} |~ "(?i)SELECT|UPDATE|INSERT|DELETE|CREATE"
        ```
        *(Doit retourner votre commande SQL, confirmant l'audit de la base de données.)*
  * **Rechercher les accès à l'administration WordPress :**
      * **Action :** Connectez-vous à `http://localhost:8080/wp-admin`.
      * **Prompt LogQL :**
        ```logql
        {container_name="wordpress"} |= "POST /wp-admin"
        ```

### C. Gestion des Données (SQL) via PhpMyAdmin

1.  **Dans le panneau de gauche de phpMyAdmin,** cliquez sur le nom de votre base `obi_database`.
2.  **Ouvrez l'onglet `SQL`** dans le menu supérieur.
3.  **Exécutez la requête :** Collez l'une des requêtes ci-dessous, puis cliquez sur **Exécuter**.

<!-- end list -->

  * **Lister les utilisateurs :**
    ```sql
    SELECT ID, user_login, user_email FROM wp_users;
    ```
  * **Compter les articles publiés :**
    ```sql
    SELECT COUNT(*) FROM wp_posts WHERE post_status = 'publish';
    ```

-----

## 🛑 Nettoyage de l'environnement

Pour arrêter et supprimer tous les conteneurs, les réseaux et les **données** associées (volumes persistants), utilisez cette commande :

```bash
docker-compose --env-file wordpress.env down -v
```

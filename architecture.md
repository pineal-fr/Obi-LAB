### Schéma de l'Infrastructure


***

## Plan Détaillé de l'Infrastructure

L'architecture est organisée autour de trois flux principaux : le **flux applicatif** (la manière dont le site WordPress est servi), le **flux de métriques** (la supervision de la performance), et le **flux de logs** (la centralisation des journaux).

Tous les conteneurs communiquent entre eux sur un réseau Docker unique nommé **`monitoring`**.

---

### 1. Flux Applicatif : Servir le Site WordPress

Ce flux décrit comment un visiteur accède à votre site.

1.  **Utilisateur** ➔ **NGINX**
    * L'utilisateur se connecte à `http://localhost:8080` depuis son navigateur.
    * La requête arrive sur le conteneur **`nginx`** (port `80`).

2.  **NGINX** ➔ **WordPress (PHP-FPM)**
    * **Si la requête concerne un fichier statique** (image, CSS, JS), **`nginx`** le sert directement depuis le volume partagé `wordpress_data`.
    * **Si la requête est pour une page PHP**, **`nginx`** la transmet au conteneur **`wordpress`** sur le port `9000` via FastCGI, comme configuré dans `nginx.conf` (`fastcgi_pass wordpress:9000;`).

3.  **WordPress** ➔ **MySQL**
    * Le conteneur **`wordpress`** exécute le code PHP. Pour récupérer ou écrire des données (articles, utilisateurs, etc.), il se connecte à la base de données.
    * La connexion se fait vers le service **`mysql`** sur le port `3306` (le port par défaut de MySQL).

4.  **MySQL** ➔ **WordPress** ➔ **NGINX** ➔ **Utilisateur**
    * **`mysql`** renvoie les données à **`wordpress`**.
    * **`wordpress`** génère la page HTML et la renvoie à **`nginx`**.
    * **`nginx`** envoie la réponse finale au navigateur de l'utilisateur.

---

### 2. Flux de Métriques (Monitoring)

Ce flux explique comment les données de performance sont collectées et visualisées.

1.  **Collecte des Métriques**
    * **`node-exporter`** expose en continu les métriques du système hôte (CPU, RAM, disque) sur son port `9100`.
    * **`nginx`** expose des métriques brutes (connexions actives) sur sa page `/stub_status`.
    * **`nginx-exporter`** "scrape" (récupère) en permanence les données de `http://nginx/stub_status` et les traduit dans un format compréhensible pour Prometheus, les exposant sur son port `9113`.

2.  **Centralisation avec Prometheus**
    * Le conteneur **`prometheus`** est configuré via `prometheus.yml` pour scraper à intervalle régulier :
        * **`node-exporter:9100`** pour les métriques système.
        * **`nginx-exporter:9113`** pour les métriques NGINX.
    * Prometheus stocke ces données dans sa base de données temporelle. Vous pouvez les consulter directement sur `http://localhost:9090`.

3.  **Visualisation avec Grafana**
    * Le conteneur **`grafana`** utilise **`prometheus`** comme source de données (`datasource.yml`).
    * Lorsque vous consultez un tableau de bord dans **`grafana`** (`http://localhost:3000`), il envoie des requêtes à **`prometheus`** pour afficher les graphiques de performance.

---

### 3. Flux de Logs

Ce flux montre comment les journaux de tous les conteneurs sont centralisés et explorés.

1.  **Collecte des Logs (Promtail)**
    * Le conteneur **`promtail`** est configuré via `config.yaml` pour surveiller le socket Docker de l'hôte (`/var/run/docker.sock`).
    * Il détecte et lit automatiquement les flux de logs (`stdout`/`stderr`) de **tous les autres conteneurs** (`nginx`, `wordpress`, `mysql`, etc.).

2.  **Agrégation avec Loki**
    * **`promtail`** envoie ensuite ces lignes de log, enrichies avec des étiquettes (comme `container_name`), au conteneur **`loki`** sur son port `3100`.
    * **`loki`** indexe les métadonnées (les étiquettes) et stocke les logs de manière efficace.

3.  **Exploration avec Grafana**
    * **`grafana`** est également connecté à **`loki`** comme source de données.
    * Depuis l'onglet "Explore" de Grafana, vous pouvez exécuter des requêtes **LogQL** pour rechercher, filtrer et analyser les logs de n'importe quel conteneur de l'infrastructure.

---

### 4. Outils Annexes

* **phpMyAdmin**
    * Accessible sur `http://localhost:8081`, ce service se connecte directement au conteneur **`mysql`** pour vous offrir une interface web de gestion de la base de données.

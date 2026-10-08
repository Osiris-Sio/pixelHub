# 🎮 PixelHub — R5.A.10

> Dépôt pour le module **R5.A.10 : Nouveaux paradigmes de bases de données** (BUT Informatique - 3ᵉ année).
>
> **Auteur :** Louis AMEDRO

PixelHub est un projet fil rouge illustrant la coexistence et l'utilisation conjointe de plusieurs moteurs de bases de données au sein d'une même application (.NET / C#). Chaque moteur est exploité selon son cas d'usage optimal :

- **PostgreSQL** (Relationnel) : Gestion des comptes joueurs, transactions et inventaires.
- **MongoDB** (Document) : Catalogue des jeux vidéo avec métadonnées flexibles.
- **Redis** (Clé-Valeur / In-Memory) : Cache rapide, sessions et compteurs temps réel.
- **Neo4j** (Graphe) : Réseau social d'amis et moteur de recommandation.

---

## 📁 Structure du dépôt

```text
pixelHub/
├── .gitignore          # Fichiers exclus du suivi Git (bin, obj, .env, IDE...)
├── cours/              # Supports de cours, slides et fiches théoriques
└── TP/
    ├── TP1/            # Énoncés, guides d'installation et notes/réponses
    └── pixelHub/       # Application et environnement de développement
        ├── .env.example        # Modèle des variables d'environnement (ports)
        ├── docker-compose.yml   # Orchestration des 4 SGBD
        ├── data/                # Jeux de données d'exemple et scripts d'import
        └── src/
            └── PixelHub.Api/    # API Web ASP.NET Core
```

---

## 🚀 Démarrage rapide

### 1. Prérequis

- [Docker](https://www.docker.com/) & Docker Compose
- [.NET SDK](https://dotnet.microsoft.com/) _(optionnel si vous utilisez le profil Docker)_

### 2. Configuration des variables d'environnement

Positionnez-vous dans le dossier de l'application :

```bash
cd TP/pixelHub
```

Créez votre fichier `.env` local à partir du modèle (ce fichier est ignoré par Git pour préserver vos paramètres locaux) :

```bash
# Sous Linux / macOS / Git Bash :
cp .env.example .env

# Sous Windows (PowerShell) :
Copy-Item .env.example .env
```

### 3. Lancer l'environnement

Démarrer les moteurs de bases de données :

```bash
docker compose up -d
```

Lancer l'API en local :

```bash
cd src/PixelHub.Api
dotnet run
```

> L'API écoute par défaut sur `http://localhost:5199`.

_(Alternative) Si vous n'avez pas le SDK .NET sur votre machine :_

```bash
docker compose --profile app up -d --build
```

---

## 🔌 Ports et services

| Service          | Moteur           | Port d'écoute par défaut |
| ---------------- | ---------------- | ------------------------ |
| **PostgreSQL**   | Relationnel      | `15432`                  |
| **MongoDB**      | Document         | `27018`                  |
| **Redis**        | Clé-Valeur       | `16379`                  |
| **Neo4j (Web)**  | Graphe (Browser) | `17474`                  |
| **Neo4j (Bolt)** | Graphe (Driver)  | `17687`                  |
| **PixelHub API** | Backend .NET     | `5199`                   |

> ℹ️ Les ports peuvent être modifiés dans votre fichier `.env` si l'un d'eux est déjà utilisé sur votre machine (adaptez également `appsettings.json` en conséquence).  
> **Identifiants de dev :** `pixelhub` / `pixelhub_dev` (utilisateur `neo4j` pour Neo4j).

---

## 🛑 Arrêt des conteneurs

- **Mettre en pause (conserve les données) :**
  ```bash
  docker compose stop
  ```
- **Réinitialiser de zéro (supprime les volumes) :**
  ```bash
  docker compose down -v
  ```

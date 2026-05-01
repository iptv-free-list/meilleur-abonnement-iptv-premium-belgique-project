# Belgian-IPTV-StreamCheck : Outil d'Analyse et de Validation M3U/EPG

![Version](https://img.shields.io/badge/version-v2.4.1-blue.svg)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)
![Python](https://img.shields.io/badge/python-3.9%2B-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

Bienvenue sur le dépôt officiel de **Belgian-IPTV-StreamCheck**. Ce projet open-source est une suite logicielle en ligne de commande (CLI) développée en Python, spécialement conçue pour le parsing, la validation de connectivité et le scraping automatisé de guides de programmes électroniques (EPG). 

Pour paramétrer vos environnements de test réseau de manière optimale, cet outil se base sur les métriques de qualité et les standards de diffusion généralement requis pour évaluer les performances et identifier le [meilleur abonnement IPTV Premium Belgique](https://www.reddit.com/user/numciben/comments/1sz3re2/meilleur_abonnement_iptv_premium_belgique_suisse/), garantissant ainsi une fluidité sans faille des paquets vidéo sur le marché du Benelux et de la Suisse.

## 📋 Fonctionnalités Principales

Notre outil n'est pas un lecteur vidéo, mais bien un **validateur de flux HLS (HTTP Live Streaming)** conçu pour les administrateurs réseaux et les développeurs.

*   **Analyseur de syntaxe M3U/M3U8 :** Vérification stricte des balises `#EXTM3U`, `#EXTINF`, et de la validité des URL des segments `.ts`.
*   **Test de Latence et de Bande Passante :** Mesure automatisée du *Ping*, du *Jitter*, et de la perte de paquets (Packet Loss) sur les serveurs de diffusion européens.
*   **Scraping EPG (XMLTV) :** Synchronisation asynchrone des métadonnées des chaînes avec gestion dynamique des fuseaux horaires (CET/CEST).
*   **Détection de Géoblocage :** Simulation d'en-têtes HTTP (User-Agent, X-Forwarded-For) pour vérifier si le flux nécessite un routage VPN spécifique à la Belgique.
*   **Génération de Rapports :** Exportation des résultats d'analyse aux formats JSON, CSV ou HTML pour intégration dans des dashboards de monitoring (Grafana/Kibana).

## ⚙️ Prérequis Système

Avant de déployer le script dans votre environnement de production, assurez-vous de disposer des dépendances suivantes :

*   **Python 3.9** ou supérieur.
*   **FFmpeg** (pour l'analyse des codecs vidéo/audio sans téléchargement complet du flux).
*   Un système d'exploitation basé sur UNIX (Linux Debian/Ubuntu, macOS) ou Windows avec WSL2.

## 🚀 Installation

Clonez ce dépôt dans votre répertoire de travail local et installez les dépendances requises via `pip`.

```bash
# Cloner le dépôt GitHub
git clone https://github.com/votre-org/belgian-iptv-streamcheck.git
cd belgian-iptv-streamcheck

# Créer un environnement virtuel
python3 -m venv venv
source venv/bin/activate

# Installer les dépendances (aiohttp, lxml, pytest, etc.)
pip install -r requirements.txt
```

## 💻 Guide d'Utilisation

L'utilisation de base nécessite une liste de lecture valide (fichier local ou URL). L'outil utilise la programmation asynchrone (`asyncio`) pour traiter des milliers de chaînes en quelques secondes.

### Commande de base pour la validation M3U

```bash
python streamcheck.py --input /chemin/vers/playlist.m3u --region BE --threads 10
```

### Exemple de sortie console (Terminal)

```text
[INFO] Chargement de la playlist... 4021 flux détectés.
[INFO] Filtrage par région : 'BE' (Belgique). 84 flux isolés.
[INFO] Démarrage du thread d'analyse HLS (Concurrency: 10)...

[OK] La Une HD (tvg-id: laune.be) - Latence: 42ms - Codec: H.264/AAC - Bitrate: 1080p@50fps
[OK] RTL-TVI HD (tvg-id: rtltvi.be) - Latence: 45ms - Codec: H.264/AAC - Bitrate: 1080p@50fps
[WARN] Tipik (tvg-id: tipik.be) - Latence: 120ms - Jitter élevé détecté (15ms).
[ERROR] VTM (tvg-id: vtm.be) - Erreur HTTP 403 Forbidden (Géoblocage suspecté).

[INFO] Rapport généré : ./reports/scan_be_169824.json
```

## 🛠 Architecture Technique et Scraping EPG

Le module `epg_scraper.py` est conçu pour extraire les données XMLTV de manière non-intrusive. Pour éviter d'engorger les serveurs fournisseurs, nous utilisons un système de limitation par sémaphores (`asyncio.Semaphore(5)`). 

### Nettoyage des balises TVG

Les fournisseurs de listes M3U utilisent souvent des identifiants non standard. Le script intègre un algorithme de *Fuzzy Matching* (basé sur la distance de Levenshtein) pour faire correspondre automatiquement une chaîne comme `[BE] La Une FHD` à son identifiant EPG canonique `laune.be`.

```python
# Extrait du module de Fuzzy Matching (matcher.py)
from thefuzz import process

def align_tvg_id(channel_name, epg_database):
    clean_name = sanitize_string(channel_name)
    best_match, score = process.extractOne(clean_name, epg_database.keys())
    
    if score > 85:
        return epg_database[best_match]
    return None
```

## 🤝 Contribution et Licence

Les contributions à la base de code sont les bienvenues, en particulier pour l'ajout de nouveaux profils de scraping EPG ou l'amélioration de la détection des proxys. Veuillez soumettre une *Pull Request* détaillée décrivant vos modifications.

Ce projet est distribué sous la licence **MIT**. Vous êtes libre de l'utiliser, de le modifier et de l'intégrer dans vos propres projets de monitoring réseau ou de validation de serveurs de streaming. Consultez le fichier `LICENSE` pour plus d'informations.
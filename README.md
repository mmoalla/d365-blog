# D365 F&O Tech Blog — Starter Hugo

Structure de départ pour un blog technique sur Dynamics 365 Finance & Operations,
Power Platform et l'écosystème Microsoft, basée sur [Hugo](https://gohugo.io) et
le thème [PaperMod](https://github.com/adityatelange/hugo-PaperMod).

## Contenu du starter

```
d365-blog-starter/
├── config.toml                      # Configuration du site (titre, menu, options)
├── archetypes/
│   └── default.md                   # Modèle utilisé pour chaque nouvel article
├── content/
│   └── posts/
│       └── creer-event-handler-xpp-d365-fo.md   # Premier article, prêt à publier
└── static/
    └── images/                      # Place tes captures d'écran ici
```

## Installation en local

### 1. Installer Hugo

- **Windows** : `winget install Hugo.Hugo.Extended`
- **Mac** : `brew install hugo`
- **Linux** : `sudo apt install hugo` (ou snap, voir doc officielle)

Vérifie l'installation : `hugo version`

### 2. Initialiser le dépôt Git et ajouter le thème

```bash
cd d365-blog-starter
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod --depth=1
```

### 3. Lancer le serveur local

```bash
hugo server -D
```

Le site est accessible sur `http://localhost:1313`. Le flag `-D` inclut les brouillons (`draft: true`).

### 4. Créer un nouvel article

```bash
hugo new posts/mon-nouvel-article.md
```

Le fichier généré reprend automatiquement le modèle défini dans `archetypes/default.md`.

### 5. Publier

- **GitHub Pages** : pousse le dépôt sur GitHub, active GitHub Pages avec une Action Hugo (template officiel disponible dans la doc Hugo)
- **Netlify** : connecte le repo, build command `hugo`, publish directory `public`
- **Vercel** : idem, avec le preset Hugo

## Idées de prochains articles (backlog suggéré)

- Chain of Command en X++ : quand l'utiliser plutôt qu'un Event Handler
- Dual-write : synchroniser D365 F&O et Dataverse pas à pas
- Créer un connecteur Power Automate personnalisé pour D365 F&O
- Sécuriser un rôle custom : duties, privileges et permissions expliqués
- Déployer via Azure DevOps : pipeline de build et release LCS

## Personnalisation rapide

- Modifie `title`, `author` et `description` dans `config.toml`
- Ajoute ton propre logo/favicon dans `static/`
- Les catégories utilisées dans les articles (`categories: [...]`) génèrent automatiquement
  les pages de taxonomie accessibles via le menu "Catégories"

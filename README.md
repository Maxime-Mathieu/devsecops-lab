# 🔒 DevSecOps Lab

![DevSecOps Pipeline](https://github.com/Maxime-Mathieu/devsecops-lab/workflows/DevSecOps%20Pipeline/badge.svg)
![CodeQL](https://github.com/Maxime-Mathieu/devsecops-lab/workflows/CodeQL/badge.svg)

Pipeline CI/CD sécurisé sur une application Node.js volontairement vulnérable, avec détection et correction automatique des failles de sécurité via GitHub Actions.

---

## 📋 Objectifs

- Mettre en place un pipeline DevSecOps complet
- Détecter automatiquement les vulnérabilités (SAST, SCA, Secrets, Container Scan)
- Corriger les failles de sécurité courantes
- Appliquer les bonnes pratiques DevSecOps en pratique

---

## 🏗️ Structure du projet

```
devsecops-lab/
├── .github/
│   └── workflows/
│       └── security.yml       # Pipeline DevSecOps
├── src/
│   ├── server.js              # Application Node.js sécurisée
│   └── package.json           # Dépendances à jour
├── Dockerfile                 # Image Alpine sécurisée
├── .env.example               # Variables d'environnement requises
├── .gitignore                 # .env exclu du dépôt
└── README.md
```

---

## 🔄 Pipeline DevSecOps

Le pipeline se déclenche à chaque `push` et `pull_request` sur `main`.

```
Build → ┬→ SAST (Semgrep)
        ├→ SCA (npm audit)
        ├→ Secrets (Gitleaks)
        └→ Container Scan (Trivy) → Report
```

| Job | Outil | Périmètre |
|-----|-------|-----------|
| 🏗️ Build | Docker | Construction de l'image |
| 🔍 SAST | Semgrep | Analyse statique du code source |
| 📦 SCA | npm audit | Vulnérabilités dans les dépendances |
| 🔐 Secrets | Gitleaks | Détection de secrets dans l'historique Git |
| 🐳 Container Scan | Trivy | CVE dans l'image Docker (CRITICAL) |
| 📊 Report | - | Rapport JSON consolidé |

---

## 🚨 Vulnérabilités détectées (avant corrections)

L'application initiale contenait les failles suivantes :

| Catégorie | Faille | Outil détecteur |
|-----------|--------|-----------------|
| Secrets | 3 clés API hardcodées (MongoDB, Stripe, SendGrid) | Semgrep + Gitleaks |
| OWASP A02 | Endpoint `/debug` exposant les secrets et `process.env` | Semgrep |
| SCA | `express@4.17.1` — CVE-2022-24999 (prototype pollution) | npm audit |
| SCA | `jsonwebtoken@8.5.1` — CVE-2022-23529 (injection via secret) | npm audit |
| Container | Image `node:14` obsolète avec multiples CVE CRITICAL | Trivy |

---

## ✅ Corrections appliquées

### Code (`src/server.js`)
- Suppression de tous les secrets hardcodés → variables d'environnement via `dotenv`
- Suppression de l'endpoint `/debug` en production
- Ajout de `helmet` pour les headers HTTP sécurisés
- Rate limiting sur `/api/login` (5 tentatives / 15 min)
- Validation des entrées avec `express-validator`
- JWT signé avec secret ≥ 32 caractères depuis les variables d'environnement

### Dépendances (`src/package.json`)
- `express` : `4.17.1` → `^4.18.2`
- `jsonwebtoken` : `8.5.1` → `^9.0.2`
- Ajout : `helmet`, `express-rate-limit`, `express-validator`, `dotenv`

### Dockerfile
- Image de base : `node:14` → `node:22-alpine`
- Exécution en utilisateur non-root (`nodejs:1001`)
- `npm ci --only=production` + nettoyage du cache
- Healthcheck sur `/health`

### Secrets
- Tous les secrets de production dans **GitHub Secrets** (Settings > Actions)
- `.env` ajouté au `.gitignore`

---

## ⚙️ Installation locale

### Prérequis
- Docker
- Node.js 18+
- Git

### Lancer l'application

```bash
# Cloner le repo
git clone https://github.com/Maxime-Mathieu/devsecops-lab.git
cd devsecops-lab

# Copier et remplir les variables d'environnement
cp .env.example .env
# Éditer .env avec vos valeurs

# Construire et lancer via Docker
docker build -t devsecops-lab .
docker run -p 3000:3000 --env-file .env devsecops-lab
```

### Variables d'environnement requises

```bash
# .env (ne jamais committer ce fichier)
JWT_SECRET=<secret-aleatoire-min-32-caracteres>
ADMIN_USER=admin
ADMIN_PASS=<mot-de-passe-fort>
NODE_ENV=production
```

Générer un JWT_SECRET sécurisé :
```bash
openssl rand -base64 32
```

---

## 🔑 GitHub Secrets requis

Configurer dans **Settings > Secrets and variables > Actions** :

| Secret | Description |
|--------|-------------|
| `JWT_SECRET` | Secret JWT (min. 32 caractères) |
| `ADMIN_USER` | Nom d'utilisateur admin |
| `ADMIN_PASS` | Mot de passe admin fort |

---

## 📊 Résultats du pipeline

| Check | Avant | Après |
|-------|-------|-------|
| SAST (Semgrep) | ❌ 4 findings | ✅ 0 finding |
| SCA (npm audit) | ❌ 2 CVE High | ✅ 0 CVE |
| Secrets (Gitleaks) | ❌ 3 secrets détectés | ✅ Aucun secret |
| Container Scan (Trivy) | ❌ Multiples CVE CRITICAL | ⚠️ 1 CVE image de base* |

> *CVE-2026-22184 (zlib) — présente dans Alpine 3.23.3, indépendante du code applicatif, corrigée dans la prochaine mise à jour de l'image de base.

---

## 📚 Ressources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Semgrep Rules](https://semgrep.dev/explore)
- [Trivy Documentation](https://trivy.dev/)
- [Gitleaks](https://github.com/gitleaks/gitleaks)

---

## ✅ Checklist de sécurité

- [x] Pipeline s'exécute sans erreur de validation
- [x] Tous les secrets dans GitHub Secrets (aucun dans le code)
- [x] Dépendances à jour (0 CVE npm)
- [x] Dockerfile sécurisé (utilisateur non-root, image Alpine)
- [x] README complet avec instructions
- [x] Badge de build dans le README
- [x] `.gitignore` contient `.env`
- [x] SAST, SCA, Secrets : tous les checks passent ✅

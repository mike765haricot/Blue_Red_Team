# 🔐 DevSecOps Mission Impossible — "Supply Chain Guardian"

> **Projet académique ESAIP — DevSecOps IR4 — 2026**
> Audit offensif (Red Team) et implémentation défensive (Blue Team) d'une application Flask conteneurisée avec Docker.

---

## 👥 Équipe

| Membre | Branche Git | Mission Blue Team |
|--------|-------------|-------------------|
| **Mike DAYAWA** | [`mike`](../../tree/mike) | 🛡️ B1 — Mitigation SSRF (défense en profondeur 5 couches) |
| **Andrisca MABIKA** | [`andrisca`](../../tree/andrisca) | 🐳 B4 — Durcissement Docker & suppression `/debug` |
| **Damien LANING** | [`damien`](../../tree/damien) | 🚂 B2 — Pipeline CI/CD fortifiée & auth Vault |
| **Yohann EKAMBIE SOUAMY** | [`yohann`](../../tree/yohann) | 🔒 B3 — Hygiène des secrets & auth `/admin` |

> ⚠️ **Chaque membre a travaillé sur sa propre branche.** Cliquez sur les liens ci-dessus pour voir le travail individuel de chacun, incluant le code corrigé, le rapport (`docs/<nom>/RAPPORT.md`) et la documentation dans le README de chaque branche.

---

## 📖 Description du Projet

Ce projet est un **Escape Game DevSecOps** : une application web volontairement vulnérable composée de 2 micro-services Flask (`web` + `vault`) orchestrés par Docker Compose.

### Objectif pédagogique

1. **Phase Red Team** (attaque) : Identifier et exploiter les vulnérabilités pour récupérer des flags cachés.
2. **Phase Blue Team** (défense) : Corriger chaque vulnérabilité identifiée en appliquant les bonnes pratiques DevSecOps.

### Architecture de l'application

```
┌─────────────────────────────────────────────────┐
│                Docker Network (bridge)           │
│                                                  │
│  ┌──────────────┐         ┌──────────────┐      │
│  │   web:5000   │ ──────► │  vault:7000  │      │
│  │  (Flask App) │  HTTP   │  (Secrets)   │      │
│  │  - /fetch    │         │  - /secret   │      │
│  │  - /admin    │         │  - /health   │      │
│  │  - /status   │         │  - /debug ❌ │      │
│  └──────┬───────┘         └──────────────┘      │
│         │ port 5001                              │
└─────────┼────────────────────────────────────────┘
          │
    ┌─────┴─────┐
    │  Client   │
    │ (Browser) │
    └───────────┘
```

---

## 🚀 Démarrage Rapide

### Prérequis

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Git + VS Code
- (Optionnel) Trivy, pip-audit, gitleaks, bandit

### Lancer l'application (état vulnérable)

```bash
# Branche master = état vulnérable d'origine
git checkout master
docker compose up --build
```

Ouvrir : [http://localhost:5001](http://localhost:5001)

### Tester les corrections d'un membre

```bash
# Exemple : tester les corrections de Mike (SSRF)
git checkout mike
docker compose down -v && docker compose up --build
```

---

## 🔴 Phase Red Team — Résumé des Attaques

Le rapport Red Team complet est disponible dans `Rapport_RedTeam_Complet.pdf`.

### Flags récupérés

| Mission | Flag | Méthode | Rédacteur |
|---------|------|---------|-----------|
| M1 — SSRF | Accès au vault interne | `SSRF /fetch → vault:7000` | Mike |
| M2 — Exfiltration | `VAULT_TOKEN` exfiltré | `SSRF → /debug → env vars` | Andrisca |
| M3 — Flag Vault | `FLAG{ssrf_reached_vault}` | `SSRF → /secret?token=...` | Damien |
| M4 — Admin Flag | `FLAG{supply_chain_guardian}` | `.env → /admin?token=...` | Yohann |
| M5 — Supply Chain | 12 faiblesses identifiées | Audit Dockerfile + CI + deps | Tous |

### Vulnérabilités identifiées

| # | Vulnérabilité | CWE | Sévérité |
|---|---------------|-----|----------|
| 1 | SSRF sur `/fetch` (aucune validation) | CWE-918 | 🔴 CRITIQUE |
| 2 | Endpoint `/debug` exposant `os.environ` | CWE-200 | 🔴 CRITIQUE |
| 3 | Token admin dans `.env` commité | CWE-798 | 🔴 CRITIQUE |
| 4 | Auth par GET parameter (visible dans logs) | CWE-522 | 🟠 HAUTE |
| 5 | Comparaison non time-safe (`!=`) | CWE-208 | 🟡 MOYENNE |
| 6 | Dockerfile sans `.dockerignore` | CWE-200 | 🟠 HAUTE |
| 7 | Exécution en root (pas de `USER`) | CWE-269 | 🟠 HAUTE |
| 8 | Pipeline CI sans quality gates | CWE-1357 | 🟠 HAUTE |
| 9 | Tag `:latest` (pas de SHA pinning) | CWE-829 | 🟡 MOYENNE |
| 10 | Pas de SBOM ni signature d'image | — | 🟡 MOYENNE |

---

## 🛡️ Phase Blue Team — Corrections Appliquées

Chaque membre a corrigé les vulnérabilités sur sa branche dédiée :

### B1 — Mitigation SSRF (Branche `mike`)

| Correctif | Description |
|-----------|-------------|
| ✅ Allowlist de domaines | Seuls les domaines autorisés sont accessibles |
| ✅ Blocage IP privées | RFC 1918 (10.x, 172.16.x, 192.168.x) bloquées |
| ✅ Validation DNS | Anti-rebinding, résolution et vérification pré-requête |
| ✅ Désactivation redirections | `allow_redirects=False` |
| ✅ Timeout strict | Limitation à 2 secondes |

### B2 — Pipeline CI/CD Fortifiée (Branche `damien`)

| Correctif | Description |
|-----------|-------------|
| ✅ SAST (Bandit) | Analyse statique du code Python |
| ✅ Audit dépendances | `pip-audit` contre les CVE connues |
| ✅ Scan d'image (Trivy) | Vérification des CVE HIGH/CRITICAL |
| ✅ Tag immutable | Hash du commit au lieu de `:latest` |
| ✅ Auth Vault sécurisée | Header `Authorization: Bearer` + `hmac.compare_digest` |

### B3 — Hygiène des Secrets (Branche `yohann`)

| Correctif | Description |
|-----------|-------------|
| ✅ Suppression `.env` | Fichier retiré du repo et ajouté au `.gitignore` |
| ✅ `.env.example` | Template anonymisé pour l'onboarding |
| ✅ Auth `/admin` sécurisée | Migration GET → Header `Authorization: Bearer` |
| ✅ Comparaison time-safe | `hmac.compare_digest()` contre les timing attacks |

### B4 — Durcissement Docker (Branche `andrisca`)

| Correctif | Description |
|-----------|-------------|
| ✅ `.dockerignore` | Exclusion de `.env`, `.git/`, `scripts/`, `docs/` |
| ✅ Utilisateur non-root | `USER appuser` dans le Dockerfile |
| ✅ HEALTHCHECK | Surveillance automatique de la santé du conteneur |
| ✅ Suppression `/debug` | Endpoint d'exfiltration entièrement retiré |

---

## 🧪 Guide de Test Rapide

Pour vérifier que les corrections fonctionnent, suivez ce parcours branche par branche :

```bash
# 1. Tester Yohann (B3) — L'ancienne attaque GET doit échouer
git checkout yohann && docker compose down -v && docker compose up --build -d
curl "http://localhost:5001/admin?token=bSXdxNlOVFk8tEPgmqRWNwOibH6wxJVx"
# → 403 Forbidden ✅

# 2. Tester Andrisca (B4) — /debug doit être supprimé
git checkout andrisca && docker compose down -v && docker compose up --build -d
curl "http://localhost:5001/fetch?url=http://vault:7000/debug"
# → 404 Not Found ✅

# 3. Tester Damien (B2) — Token GET vers vault doit échouer
git checkout damien && docker compose down -v && docker compose up --build -d
curl "http://localhost:5001/fetch?url=http://vault:7000/secret?token=7Db33sFjTDvB8ILDMJeOYwdy"
# → 403 Forbidden ✅

# 4. Tester Mike (B1) — SSRF vers vault doit être bloquée
git checkout mike && docker compose down -v && docker compose up --build -d
curl "http://localhost:5001/fetch?url=http://vault:7000/health"
# → Blocked / Error ✅
```

---

## 📁 Structure du Projet

```
DevSecOps_Mission_Impossible/
├── web/
│   ├── app.py              # Application web Flask (vulnérable sur master)
│   └── requirements.txt
├── vault/
│   └── app.py              # Service interne Vault
├── docs/
│   ├── mike/RAPPORT.md     # Rapport Blue Team - SSRF
│   ├── andrisca/RAPPORT.md # Rapport Blue Team - Docker
│   ├── damien/RAPPORT.md   # Rapport Blue Team - Pipeline
│   └── yohann/RAPPORT.md   # Rapport Blue Team - Secrets
├── scripts/
│   └── pipeline.sh         # Script CI/CD
├── Dockerfile
├── docker-compose.yml
├── .env.example            # Template de secrets (anonymisé)
├── .dockerignore            # Exclusion build context
├── .gitignore
└── Rapport_RedTeam_Complet.pdf
```

---

## 📊 Checklist DevSecOps — 10 Règles d'Or

1. ✅ **Ne jamais commiter de secrets** → `.gitignore` + `.env.example` + gitleaks
2. ✅ **Valider toutes les entrées utilisateur** → Allowlist, blocage IP privées
3. ✅ **Principe de moindre privilège** → `USER` non-root dans Docker
4. ✅ **Authentification forte** → Headers HTTP, `hmac.compare_digest()`
5. ✅ **Supprimer les endpoints de debug** → Pas de `/debug` en production
6. ✅ **Scanner le code** → SAST (Bandit) dans la CI
7. ✅ **Auditer les dépendances** → `pip-audit` / `trivy`
8. ✅ **Immutabilité des images** → SHA digest au lieu de `:latest`
9. ✅ **Quality gates** → Échec du pipeline si vulnérabilité détectée
10. ✅ **Réduire la surface d'attaque** → `.dockerignore` + HEALTHCHECK

---

## 📝 Licence

Projet académique — ESAIP DevSecOps IR4 — 2026

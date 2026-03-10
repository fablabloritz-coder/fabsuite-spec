# Contexte Projet — FabLab Suite

## Qui
Compte GitHub : **fablabloritz-coder**
Workflow : travail LOCAL → test → push GitHub (jamais push sans validation)

## Quoi
Suite d'applications Flask/SQLite/Bootstrap 5 pour la gestion d'un FabLab.
**FabHome est le hub central** qui agrège et monitore toutes les autres apps.

## Architecture

```
FabLab Suite
├── FabHome      → Hub central (Docker, port 3001:3000)  github: FabHome
├── Fabtrack     → Machines & consommations (.bat, port 5555)  github: Fabtrack
├── PretGo       → Emprunts équipements (.bat, port 5000)  github: PretGo
├── FabBoard     → Missions/tâches (.bat, port 5050)  github: FabBoard
└── fabsuite-spec → Spécification du manifest inter-apps  github: fabsuite-spec

Hors suite (projet personnel, NE PAS inclure) :
└── FrigoScan
```

## Stack technique
- **Backend** : Python Flask + SQLite (WAL mode)
- **Frontend** : Bootstrap 5.3 + Vanilla JavaScript (PAS de React/Vue)
- **FabHome** : tourne dans Docker (`docker compose up -d --build`)
- **Autres apps** : lancées avec leurs `.bat` respectifs sur Windows
- **Communication Docker→host** : `host.docker.internal` (configuré dans docker-compose.yml)

## FabLab Suite Manifest (spec v1.0.0)
Chaque app expose :
- `GET /api/fabsuite/manifest` — identité, capabilities, widgets
- `GET /api/fabsuite/health` — health check rapide (<500ms)
- CORS activé uniquement sur `/api/fabsuite/*`

Capabilities disponibles : stats, machines, consumptions, loans, inventory, stock, calendar, tasks, display, scan (+ custom `x-*`)

Widget types : counter, list, status, chart, text, table

## État actuel des implémentations

### FabHome (hub)
- Table `suite_apps` dans models.py — stocke apps enregistrées
- Endpoints `/api/suite/*` — register/delete/refresh/proxy/dashboard/notifications
- Widget "FabLab Suite" sur le dashboard (type `fabsuite`)
- Services : Docker, Portainer, Proxmox, Pi-hole, AdGuard, Uptime Kuma, Plex, Radarr, Sonarr, TrueNAS, **Repetier-Server** (nouveau)
- PretGo et Fabtrack retirés des services (remplacés par FabLab Suite)
- Effet de rebond (translateY hover) supprimé sur les cartes

### Fabtrack
- Endpoints FabSuite : manifest, health, 4 widgets, notifications
- Widgets : monthly-consumptions (counter), machine-status (status), top-machines (chart), recent-activity (list)

### PretGo
- Endpoints FabSuite : manifest, health, 3 widgets, notifications
- CSRF exemptions ajoutées pour `/api/fabsuite/*`

### FabBoard
- Endpoints FabSuite : manifest, health, 3 widgets

## Décisions prises (NE PAS remettre en question)
- Thème **par app** (pas de thème global)
- FrigoScan = projet personnel, exclu de la FabLab Suite
- Mobile = pour plus tard
- Pas de dépendances externes (tout local)
- Pas de React — Vanilla JS uniquement

## Prochaines étapes envisagées
1. **Notifications topbar FabHome** — badge 🔔 agrégeant alertes de toutes les apps (machines en panne, emprunts en retard)
2. **Support Raise3D** — API locale port 10800 (à tester sur les machines du FabLab)
3. **FabStock** — nouvelle app (même base Flask/SQLite) pour gestion stock matières

## Lancer l'environnement de travail
```bash
# FabHome
cd FabHome && docker compose up -d --build

# Autres apps (depuis Windows, lancer les .bat)
# Fabtrack  → port 5555
# PretGo    → port 5000
# FabBoard  → port 5050

# Ajouter les apps dans FabHome :
# Réglages → FabLab Suite → http://host.docker.internal:5555 (Fabtrack)
#                         → http://host.docker.internal:5000 (PretGo)
#                         → http://host.docker.internal:5050 (FabBoard)
```

## Convention git
- Branches feature/* pour les nouveaux développements
- Merge sur main après test et validation
- Push sur GitHub uniquement après validation locale
- Co-authored-by: Claude Haiku 4.5 <noreply@anthropic.com>

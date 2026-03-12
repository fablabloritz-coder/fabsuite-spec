# Contexte Projet — fabsuite-spec

## Qui
Compte GitHub : **fablabloritz-coder**
Workflow : travail LOCAL → test → push GitHub (jamais push sans validation)

## Quoi
Spécification du manifest inter-apps FabLab Suite (v1.0.0) + module `fabsuite_core/` vendored.

## Architecture FabLab Suite

```
FabLab Suite
├── FabHome      → Hub central (Docker, port 3001:3000)  github: FabHome
├── Fabtrack     → Machines & consommations + stock + missions (.bat, port 5555)  github: Fabtrack
├── PretGo       → Emprunts équipements (.bat, port 5000)  github: PretGo
├── FabBoard     → Dashboard TV (.bat, port 5580)  github: FabBoard
├── FabStock     → Stock matières consommables (.bat, port 5500)  github: FabStock
└── fabsuite-spec → Spécification du manifest inter-apps  github: fabsuite-spec
```

## fabsuite_core/ — Module Python vendored

Maintenu ici dans `fabsuite_core/`, copié vers chaque app.

```
fabsuite_core/
├── __init__.py          # Version, constantes
├── manifest.py          # Blueprint Flask auto-configuré (manifest + health + widgets + CORS)
├── widgets.py           # Builders conformes : counter, status_list, item_list, chart, text, table, notification
├── http_client.py       # Client HTTP (fetch_json, check_health, fetch_manifest, fetch_widget)
├── security.py          # Secret key persistée (env > fichier > génération)
├── config.py            # Paramètres DB (get_param, set_param, ensure_parametres_table)
└── VERSION              # Numéro de version du core
```

**Synchronisation** : modifier ici, copier vers chaque app, committer.

## FabLab Suite Manifest (spec v1.0.0)
Chaque app expose :
- `GET /api/fabsuite/manifest` — identité, capabilities, widgets
- `GET /api/fabsuite/health` — health check rapide (<500ms)
- CORS activé uniquement sur `/api/fabsuite/*`

Capabilities : stats, machines, consumptions, loans, inventory, stock, calendar, tasks, display, scan (+ custom `x-*`)
Widget types : counter, list, status, chart, text, table

## Convention git
- Branches feature/* pour les nouveaux développements
- Merge sur main après test et validation
- Push sur GitHub uniquement après validation locale
- Co-authored-by: Claude Sonnet 4.6 <noreply@anthropic.com>

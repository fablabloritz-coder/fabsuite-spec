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

## Journal local — 2026-03-12 (session en cours)

### Fait
- **FabHome**
	- Ajout d'un test FabBoard côté backend via endpoint `POST /api/suite/test-url` (évite les erreurs CORS du fetch navigateur).
	- Onglet **FabBoard** confirmé dans les réglages (URL, widget par défaut, actions rapides).
	- Action “Ajouter un raccourci FabBoard” : placement sur première case libre (2x1) + message explicite si aucune place.
	- Ajout de deux liens lors de la création du raccourci : `Dashboard TV` et `Paramètres FabBoard` (`/parametres`).
	- Durcissement des validations de grille (groupes + widgets) : contrôle des bornes et collisions en create/update/move.
	- Normalisation de l'URL FabBoard côté UI (`host.docker.internal` → `localhost` pour usage navigateur local).

- **Fabtrack**
	- Renforcement du module stock : auto-seed minimal des catégories (`types_activite`) si table vide pour éviter `/stock/categories` vide.
	- Amélioration de l'extraction Google Business : fallback OSM conservé mais contraint géographiquement (FR par défaut) pour réduire les faux positifs US.

### Reste à tester (tests manuels prévus plus tard)
- **FabHome**
	- Bouton “Tester FabBoard” avec URL locale et URL réseau.
	- Création du raccourci FabBoard sur plusieurs pages/grilles (cas plein, cas partiellement occupé).
	- Tentatives de resize/move en collision pour vérifier les refus backend.
- **Fabtrack**
	- Vérifier l'affichage de `/stock/categories` sur base vide et base existante.
	- Re-tester l'extraction depuis lien `share.google` + nom fournisseur (cas "3D Advance").

### Reste à faire
- Valider en local les tests ci-dessus.
- Finaliser commit/push repo par repo après validation.

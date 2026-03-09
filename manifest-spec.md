# FabLab Suite — Spécification du Manifest

**Version : 1.0.0**

---

## Vue d'ensemble

Le manifest est le contrat d'interopérabilité entre les applications FabLab Suite. Chaque application compatible expose un endpoint REST qui décrit son identité, son état, ses capacités et les widgets qu'elle propose.

**FabHome** interroge ces endpoints pour découvrir, surveiller et agréger les applications de la suite.

---

## Endpoint obligatoire

```
GET /api/fabsuite/manifest
Content-Type: application/json
```

Cet endpoint **ne nécessite aucune authentification** et doit répondre en moins de **2 secondes**.

---

## Structure du manifest

### Champs obligatoires

| Champ | Type | Description |
|-------|------|-------------|
| `app` | `string` | Identifiant unique de l'application (minuscules, sans espaces). Ex: `"fabtrack"` |
| `name` | `string` | Nom d'affichage de l'application. Ex: `"Fabtrack"` |
| `version` | `string` | Version de l'application au format semver. Ex: `"1.2.0"` |
| `suite_version` | `string` | Version de la spécification FabLab Suite supportée. Ex: `"1.0.0"` |
| `status` | `string` | État actuel : `"running"`, `"degraded"`, `"maintenance"` |
| `description` | `string` | Description courte de l'application (max 200 caractères) |

### Champs optionnels

| Champ | Type | Description |
|-------|------|-------------|
| `icon` | `string` | Classe d'icône Bootstrap Icons. Ex: `"bi-printer"` |
| `color` | `string` | Couleur identitaire de l'app (hex). Ex: `"#0d6efd"` |
| `url` | `string` | URL de base de l'application. Ex: `"http://192.168.1.10:5555"` |
| `capabilities` | `array[string]` | Liste des fonctionnalités exposées (voir section dédiée) |
| `widgets` | `array[Widget]` | Widgets disponibles pour FabHome (voir section dédiée) |
| `notifications` | `object` | Configuration des notifications (voir section dédiée) |
| `uptime` | `number` | Temps de fonctionnement en secondes depuis le dernier démarrage |
| `started_at` | `string` | Date/heure de démarrage au format ISO 8601 |

---

## Détail des structures

### Capabilities

Les capabilities décrivent ce que l'application sait faire. Elles permettent à FabHome (et aux futures apps) de savoir quelles interactions sont possibles.

**Capabilities standardisées :**

| Capability | Description |
|-----------|-------------|
| `stats` | L'app expose des statistiques consultables |
| `machines` | L'app gère des machines / équipements |
| `consumptions` | L'app suit des consommations |
| `loans` | L'app gère des prêts |
| `inventory` | L'app gère un inventaire |
| `stock` | L'app gère un stock de matières |
| `calendar` | L'app gère un calendrier / planning |
| `tasks` | L'app gère des tâches / missions |
| `display` | L'app gère un affichage (TV, écran) |
| `scan` | L'app supporte le scan de codes-barres |

Les applications peuvent déclarer des capabilities personnalisées en les préfixant avec `x-` :

```json
"capabilities": ["stats", "machines", "x-weather"]
```

### Widget

Un widget est un bloc d'information qu'une application propose à FabHome pour affichage sur son dashboard.

```json
{
  "id": "string",
  "label": "string",
  "description": "string",
  "endpoint": "string",
  "type": "string",
  "refresh_interval": "number"
}
```

| Champ | Obligatoire | Type | Description |
|-------|:-----------:|------|-------------|
| `id` | ✅ | `string` | Identifiant unique du widget dans l'app |
| `label` | ✅ | `string` | Nom d'affichage du widget |
| `description` | | `string` | Description courte |
| `endpoint` | ✅ | `string` | Chemin relatif de l'API retournant les données du widget |
| `type` | ✅ | `string` | Type de widget (voir types ci-dessous) |
| `refresh_interval` | | `number` | Fréquence de rafraîchissement recommandée en secondes (défaut: `60`) |

**Types de widgets standardisés :**

| Type | Rendu attendu | Données retournées |
|------|---------------|-------------------|
| `counter` | Un chiffre avec label | `{ "value": number, "label": string, "unit": string }` |
| `list` | Liste d'éléments | `{ "items": [{ "label": string, "value": string, "status": string }] }` |
| `status` | Indicateur d'état | `{ "items": [{ "label": string, "status": "ok" \| "warning" \| "error" }] }` |
| `chart` | Graphique simple | `{ "type": "bar" \| "line" \| "pie", "labels": [...], "values": [...] }` |
| `text` | Texte libre | `{ "content": string }` |
| `table` | Tableau de données | `{ "headers": [...], "rows": [[...], ...] }` |

### Notifications

Configuration optionnelle pour que FabHome puisse récupérer les alertes de l'application.

```json
{
  "endpoint": "string",
  "types": ["string"]
}
```

| Champ | Type | Description |
|-------|------|-------------|
| `endpoint` | `string` | Chemin relatif pour récupérer les notifications actives |
| `types` | `array[string]` | Types de notifications émises : `"info"`, `"warning"`, `"error"` |

**Format de réponse de l'endpoint notifications :**

```json
GET /api/fabsuite/notifications

{
  "notifications": [
    {
      "id": "string",
      "type": "info | warning | error",
      "title": "string",
      "message": "string",
      "created_at": "string (ISO 8601)",
      "link": "string (URL optionnelle vers le détail)"
    }
  ]
}
```

---

## Endpoint de santé (recommandé)

En complément du manifest, il est recommandé d'exposer un endpoint de vérification rapide :

```
GET /api/fabsuite/health

Réponse : { "status": "ok" }
```

Cet endpoint doit répondre en moins de **500ms** et ne fait aucune opération coûteuse. Il sert au monitoring temps réel par FabHome.

---

## Exemples concrets

### Fabtrack

```json
{
  "app": "fabtrack",
  "name": "Fabtrack",
  "version": "1.3.0",
  "suite_version": "1.0.0",
  "status": "running",
  "description": "Suivi des consommations machines et matériaux du FabLab",
  "icon": "bi-printer",
  "color": "#198754",
  "capabilities": ["stats", "machines", "consumptions"],
  "widgets": [
    {
      "id": "monthly-consumptions",
      "label": "Consommations du mois",
      "description": "Nombre total de consommations ce mois-ci",
      "endpoint": "/api/fabsuite/widget/monthly-consumptions",
      "type": "counter",
      "refresh_interval": 300
    },
    {
      "id": "machine-status",
      "label": "État des machines",
      "description": "Disponibilité des machines du FabLab",
      "endpoint": "/api/fabsuite/widget/machine-status",
      "type": "status",
      "refresh_interval": 60
    },
    {
      "id": "top-machines",
      "label": "Top machines du mois",
      "endpoint": "/api/fabsuite/widget/top-machines",
      "type": "chart",
      "refresh_interval": 600
    }
  ],
  "notifications": {
    "endpoint": "/api/fabsuite/notifications",
    "types": ["warning", "error"]
  },
  "started_at": "2026-03-09T08:00:00Z"
}
```

### PretGo

```json
{
  "app": "pretgo",
  "name": "PretGo",
  "version": "1.0.0",
  "suite_version": "1.0.0",
  "status": "running",
  "description": "Gestion des prêts de matériel pour établissements",
  "icon": "bi-box-arrow-right",
  "color": "#0d6efd",
  "capabilities": ["loans", "inventory"],
  "widgets": [
    {
      "id": "active-loans",
      "label": "Prêts en cours",
      "endpoint": "/api/fabsuite/widget/active-loans",
      "type": "counter",
      "refresh_interval": 120
    },
    {
      "id": "overdue-loans",
      "label": "Prêts en retard",
      "description": "Liste des prêts dépassant la date de retour prévue",
      "endpoint": "/api/fabsuite/widget/overdue-loans",
      "type": "list",
      "refresh_interval": 120
    }
  ],
  "notifications": {
    "endpoint": "/api/fabsuite/notifications",
    "types": ["warning"]
  }
}
```

---

## Règles d'implémentation

1. **Autonomie** — L'application DOIT fonctionner normalement même si FabHome n'est pas présent sur le réseau.
2. **Pas d'authentification** — Le manifest et les endpoints de widgets sont publics (réseau local uniquement).
3. **Réponses rapides** — Le manifest doit répondre en < 2s, le health en < 500ms, les widgets en < 5s.
4. **Graceful degradation** — Si une donnée n'est pas disponible, le widget retourne un objet valide avec des valeurs par défaut plutôt qu'une erreur.
5. **CORS** — Les endpoints FabSuite doivent autoriser les requêtes cross-origin (FabHome interroge depuis le navigateur).
6. **Préfixe d'URL** — Tous les endpoints de la suite utilisent le préfixe `/api/fabsuite/` pour éviter les conflits avec les API existantes de l'application.
7. **Rétrocompatibilité** — Les champs ajoutés dans les versions futures de la spécification seront toujours optionnels. Une app implémentant la v1.0.0 restera compatible avec un FabHome v2.0.0.

---

## Versioning de la spécification

Cette spécification suit le [Semantic Versioning](https://semver.org/) :

- **MAJOR** — Changements incompatibles (modification de champs obligatoires)
- **MINOR** — Ajouts rétrocompatibles (nouveaux champs optionnels, nouveaux types de widgets)
- **PATCH** — Corrections et clarifications

---

*FabLab Suite Manifest Spec v1.0.0*

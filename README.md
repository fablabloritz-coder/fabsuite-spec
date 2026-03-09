# FabLab Suite

**Suite d'outils numériques open-source pour FabLabs**

---

## Vision

FabLab Suite est un écosystème d'applications web **auto-hébergées**, **gratuites** et **open-source**, conçues pour répondre aux besoins quotidiens d'un FabLab. Chaque application fonctionne de manière autonome, mais peut s'intégrer aux autres via un protocole de communication standardisé.

**FabHome** joue le rôle de **hub central** : il découvre, surveille et agrège les informations de toutes les applications de la suite.

## Philosophie

- **Local-first** — Aucun cloud, aucune dépendance externe obligatoire. Vos données restent chez vous.
- **Zéro friction** — Tout se configure via l'interface web. Aucun fichier YAML, aucune ligne de commande requise pour l'utilisateur final.
- **Autonomie** — Chaque application fonctionne seule. L'intégration avec la suite est un bonus, pas une obligation.
- **Open-source** — Gratuit, modifiable, et ouvert aux contributions de la communauté.
- **Léger** — Python + Flask + SQLite. Pas de base de données externe, pas de broker de messages, pas d'usine à gaz.

## Applications

| Application | Description | Repo |
|------------|-------------|------|
| **[FabHome](https://github.com/fablabloritz-coder/FabHome)** | Hub central — portail et dashboard de la suite | [GitHub](https://github.com/fablabloritz-coder/FabHome) |
| **[Fabtrack](https://github.com/fablabloritz-coder/Fabtrack)** | Suivi des consommations machines et matériaux | [GitHub](https://github.com/fablabloritz-coder/Fabtrack) |
| **[FabBoard](https://github.com/fablabloritz-coder/FabBoard)** | Dashboard TV temps réel pour le FabLab | [GitHub](https://github.com/fablabloritz-coder/FabBoard) |
| **[PretGo](https://github.com/fablabloritz-coder/PretGo)** | Gestion des prêts de matériel | [GitHub](https://github.com/fablabloritz-coder/PretGo) |

## Architecture de la suite

```
                    ┌─────────────┐
                    │   FabHome   │
                    │    (Hub)    │
                    └──────┬──────┘
                           │ Découverte & agrégation
              ┌────────┬───┴───┬────────┐
              ▼        ▼       ▼        ▼
          Fabtrack  FabBoard  PretGo  ...
```

Chaque application expose un **manifest standardisé** (`/api/fabsuite/manifest`) qui permet à FabHome de :
- Détecter automatiquement les applications disponibles
- Afficher leur état de santé
- Intégrer leurs widgets sur le dashboard central
- Relayer les notifications importantes

## Spécification technique

Voir **[manifest-spec.md](manifest-spec.md)** pour la spécification complète du protocole d'interopérabilité.

## Stack technique commune

| Composant | Technologie |
|-----------|-------------|
| Backend | Python 3.10+ / Flask |
| Base de données | SQLite (WAL mode) |
| Frontend | Bootstrap 5.3 + Vanilla JavaScript |
| Déploiement | Docker Compose ou Python direct |
| Serveur WSGI | Waitress |

## Contribuer

FabLab Suite est un projet communautaire. Vous pouvez :

1. **Utiliser** les applications dans votre FabLab
2. **Signaler des bugs** via les Issues GitHub de chaque repo
3. **Proposer des améliorations** via Pull Request
4. **Créer votre propre application** compatible FabLab Suite en implémentant le [manifest standardisé](manifest-spec.md)

## Licence

Chaque application dispose de sa propre licence. Consultez le repo correspondant pour les détails.

---

*Projet porté par [FabLab Loritz](https://github.com/fablabloritz-coder)*

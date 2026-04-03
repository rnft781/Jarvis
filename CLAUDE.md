# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Jarvis Local V1** — assistant IA local-first en français avec trois routes d'exécution : chat, code, website. Utilise Ollama pour l'inférence locale. Pas de dépendance cloud obligatoire.

**Prérequis :**
- Ollama lancé sur `http://localhost:11434` avec le modèle `llama3` chargé
- `pip install -r requirements.txt` (dépendance unique : `requests==2.31.0`)

## Commands

```bash
# Lancer l'assistant
python main.py

# Tests
python -m pytest tests/
python -m pytest tests/test_v1_core.py          # core : router, LLM client, file tools
python -m pytest tests/test_commander_v4.py     # Commander principal (version stable)
python -m pytest tests/test_website_studio_*.py # Studio mode
python -m pytest tests/test_web_experience_*.py # Web experience architecture
```

## Architecture

### Flux d'exécution principal

```
User Input (CLI)
    ↓
main.py — boucle REPL
    ↓
Jarvis.process()
    ├─→ Commander.execute()       # missions structurées
    │    ├─→ Router.route()       # détection : chat | code | website
    │    ├─→ MissionPlanner       # crée Mission + BranchTasks
    │    ├─→ BranchSelector       # choisit la branche d'exécution
    │    ├─→ PermissionLayer      # vérifie les capabilities
    │    ├─→ CoordinatorMode      # orchestration multi-étapes
    │    └─→ Agent dispatch       # CodeAgent | WebsiteAgent
    │
    └─→ Router.route()            # chat simple → LLMClient (Ollama)

Generated artifacts → data/websites/  |  data/code_snippets/
```

### Composants clés

| Composant | Fichier | Rôle |
|---|---|---|
| **Router** | `core/router.py` | Routing par mots-clés vers chat/code/website |
| **LLMClient** | `core/llm_client.py` | Appels Ollama (config-driven) |
| **Commander** | `core/commander.py` | Orchestrateur principal — stable V4, ne pas refactorer sans raison explicite |
| **DeepPlanBuilder** | `core/deep_plan.py` | Planification multi-étapes avec work items, dépendances, points d'approbation |
| **CoordinatorMode** | `core/coordinator_mode.py` | Exécution séquentielle des work items avec gestion des dépendances |
| **Mission** | `schemas/mission.py` | Objet central : request, objective, status, branches, artifacts |

### Agents

- **CodeAgent** (`agents/code_agent.py`) — génération de code → sauvegarde dans `data/code_snippets/`
- **WebsiteAgent** (`agents/website_agent.py`) — génération de sites avec deux chemins :
  - **Quick Build** (défaut) : `BriefBuilder → PagePlanner → RenderEngine`
  - **Studio Mode** (premium) : chaîne de rôles spécialisés (voir ci-dessous)

### Website System — Studio Mode (`agents/website_system/`)

Chemin premium activé par `studio_mode`. Rôles enchaînés dans l'ordre :

1. `ExperienceStrategist` — stratégie d'expérience
2. `CreativeDirectionBuilder` — direction créative
3. `ArtDirectionBuilder` — direction artistique
4. `DesignSystemBuilder` — design tokens
5. `BrandSiteSpecialist` — spécialisation marque
6. `ComponentEngineer` — specs composants
7. `MotionInteractionDesigner` — motion/interactions
8. `MediaVisualSystemsAgent` — stratégie média
9. `RevisionManager` — raffinement
10. `StudioBlueprintBuilder → PagePlanner → RenderEngine` — rendu final

### Web Experience (`agents/web_experience/`)

Architecture premium émergente (63 fichiers). Supporte `make_a_site` et `make_a_store` (stub). Suit un pattern Intake/Discovery → rôles spécialistes.

### Design Knowledge (`docs/design/registry/` et `data/`)

Registres JSON pour les patterns de design :
- `experience_types.json` — 8 types (local_service, premium_brand, etc.)
- `style_profiles.json`, `layout_profiles.json`, `component_patterns.json`
- `page_recipes.json`, `premium_starter_kits.json`, `sector_signals.json`

## Règles importantes (AGENTS.md)

- **Ne pas modifier le Router** sans besoin clair
- **Ne pas refactorer Commander** sans demande explicite — version stable V4
- Garder `quick_build` compatible avec le chemin existant
- `studio_mode` est le chemin premium — travailler de façon incrémentale
- Documenter les changements d'architecture

## État actuel et priorités (gap_map.md)

- **Commander** : stable V4 + deep plan → pas de travaux prévus
- **Website system** : avancé → connecter les rôles premium de façon incrémentale
- **Design knowledge** : commencé → enrichir et lier aux builders
- **Rôles à connecter** : `experience_strategist`, `creative_director`, `art_director`, `design_system_builder`, `revision_manager`, `brand_site_specialist`
- **make_a_site** : système de base prêt → besoin du workflow studio end-to-end
- **Premium templates** : absents → créer 5 bases solides (travail futur)

## Configuration

`config.json` à la racine — clés importantes :

```json
{
  "ollama_url": "http://localhost:11434",
  "ollama_model": "llama3",
  "openai_api_key": "YOUR_OPENAI_API_KEY",
  "timeout": 60
}
```

## Artefacts générés

- `data/websites/{slug}/` — sites générés (HTML/CSS/JS + README)
- `data/code_snippets/` — snippets de code générés
- `archive/` — code legacy conservé pour référence, non actif

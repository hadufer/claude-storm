# STORM : recherche approfondie en 1 prompt pour Claude Code

Plugin Claude Code qui reproduit la méthode de recherche STORM du Stanford OVAL Lab (NAACL 2024) en une seule commande. Il y ajoute l'adaptation en 4 prompts publiée par Nav Toor (@heynavtoor).

Au lieu de copier-coller quatre prompts l'un après l'autre, vous tapez `/storm:storm <sujet>`. Claude mène alors une recherche multi-perspectives ancrée dans de vraies sources web, puis rédige un rapport cité.

Inspiré de [stanford-oval/storm](https://github.com/stanford-oval/storm) (licence MIT).

## Ce que fait le plugin

`/storm:storm` exécute le pipeline STORM, étape par étape.

1. Découverte de perspectives. Cinq angles (praticien, académique, sceptique, économiste, historien) plus le « rédacteur de faits de base » de STORM, adaptés au sujet.
2. Interviews ancrées en parallèle. Un sous-agent `storm-researcher` par perspective mène une interview entre un rédacteur et un expert sur trois tours, fondée sur de vraies recherches web. Chaque affirmation est citée.
3. Carte des contradictions. Les points où les perspectives s'opposent, ce sur quoi elles s'accordent toutes (donc probablement vrai), et l'angle mort que personne n'aborde.
4. Plan hiérarchique, d'abord esquissé puis affiné avec les trouvailles des interviews.
5. Article cité de type Wikipédia, rédigé section par section (sous-agents `storm-writer` en parallèle). Les citations `[n]` sont globalisées et dédupliquées par URL.
6. Briefing de synthèse. Un résumé pour décideur en 60 secondes, cinq conclusions classées par fiabilité, une connexion non évidente, une action concrète, et la question ouverte qui changerait la donne.
7. Auto-relecture critique. Scores de confiance de 1 à 10, maillon le plus faible, vérification des biais, perspective manquante, note finale. C'est ce qui corrige la faiblesse connue de STORM : le système ne se critique pas lui-même.

Le rapport est enregistré dans `storm-<sujet>.md`, avec une section References cliquable.

## Les deux commandes

| Commande | Web | Sous-agents | Sortie |
|----------|:---:|:-----------:|--------|
| `/storm:storm` | oui | oui, en parallèle | Rapport complet cité, briefing et relecture critique, enregistré sur disque |
| `/storm:storm-brief` | non | non | Briefing rapide en 4 phases, non sourcé, affiché dans le chat |

`/storm:storm-brief` reprend la méthode exacte du tweet de Nav Toor en une seule passe, sans appel web. Pratique quand vous voulez le briefing de cinq minutes sans attendre la recherche.

## Installation

### Depuis GitHub (recommandé pour partager)

```bash
/plugin marketplace add hadufer/claude-storm
/plugin install storm@storm-marketplace
/reload-plugins
```

### Depuis un dossier local

```bash
/plugin marketplace add C:/chemin/vers/claude-storm
/plugin install storm@storm-marketplace
/reload-plugins
```

### Test le temps d'une session, sans installer

```bash
claude --plugin-dir C:/chemin/vers/claude-storm
```

Pour valider le plugin avant publication :

```bash
claude plugin validate .
```

## Utilisation

```bash
# Recherche complète, ancrée et citée
/storm:storm l'impact de l'IA générative sur l'éducation

# Réglages optionnels
/storm:storm voitures électriques --depth deep --lang fr
/storm:storm quantum computing --depth quick

# Briefing rapide en une passe, sans web
/storm:storm-brief stratégie de pricing SaaS --role fondateur
```

### Arguments

- `--depth quick | standard | deep` : profondeur de la recherche. Par défaut `standard`, soit cinq perspectives plus le rédacteur de base et trois tours d'interview.
- `--lang <code>` : langue du livrable. Par défaut, la langue de votre requête.
- `--no-web` : ignore la recherche web. À éviter ; pour ce besoin, préférez `storm-brief`.
- `--role <rôle>` (mode brief) : votre rôle, utilisé pour calibrer l'action concrète.

Les commandes d'un plugin portent toujours le préfixe du plugin, ici `storm`. D'où `/storm:storm` et `/storm:storm-brief`.

## Coût et performance

`/storm:storm` est volontairement lourd. Il lance plusieurs sous-agents et de nombreuses recherches web pour atteindre une qualité proche de 40 à 60 heures de travail humain. Pour une réponse en quelques secondes sans appel web, utilisez `/storm:storm-brief`.

Conseil : lancez la commande complète sous Opus pour la qualité de synthèse. Les sous-agents `storm-researcher` tournent sous Sonnet par défaut, pour la vitesse et le parallélisme. STORM procède de la même façon, avec un modèle léger pour les conversations et un modèle plus fort pour la rédaction.

## Structure du plugin

```
claude-storm/
├── .claude-plugin/
│   ├── plugin.json          # manifeste (nom : "storm")
│   └── marketplace.json     # catalogue de distribution
├── skills/
│   ├── storm/
│   │   ├── SKILL.md         # /storm:storm, l'orchestrateur complet
│   │   └── reference.md     # prompts STORM, conventions, gabarit du rapport
│   └── storm-brief/
│       └── SKILL.md         # /storm:storm-brief, la méthode 4 prompts
├── agents/
│   ├── storm-researcher.md  # interviewer ancré, guidé par perspective
│   └── storm-writer.md      # rédacteur de section cité
├── README.md
└── LICENSE
```

## Crédits

STORM vient de Shao et al., « Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models », NAACL 2024 ([arXiv:2402.14207](https://arxiv.org/abs/2402.14207)), code sur [stanford-oval/storm](https://github.com/stanford-oval/storm) (licence MIT). Démo en ligne : [storm.genie.stanford.edu](https://storm.genie.stanford.edu).

L'adaptation en 4 prompts vient de Nav Toor ([@heynavtoor](https://x.com/heynavtoor/status/2067194761446920264)).

Projet indépendant, sans lien avec l'Université de Stanford. Licence MIT.

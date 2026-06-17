# 🌩️ STORM — Deep Research en 1 prompt pour Claude Code

Plugin Claude Code qui reproduit, **en une seule commande**, la méthode de
recherche **STORM** du Stanford OVAL Lab (NAACL 2024) — combinée à l'adaptation
en 4 prompts popularisée par [Nav Toor (@heynavtoor)](https://x.com/heynavtoor/status/2067194761446920264).

> Au lieu de copier-coller 4 prompts à la suite, tapez **`/storm:storm <sujet>`**
> et Claude mène une vraie recherche multi-perspectives, **ancrée dans des
> sources web réelles**, puis produit un rapport cité de niveau « étudiant en
> doctorat ».

Inspiré de [stanford-oval/storm](https://github.com/stanford-oval/storm) (MIT).

---

## Ce que ça fait

`/storm:storm` exécute le pipeline STORM complet, automatiquement :

1. **Découverte de perspectives** — 5 angles (Praticien, Académique, Sceptique,
   Économiste, Historien) + le « Basic fact writer » généraliste de STORM,
   adaptés au sujet.
2. **Interviews ancrées en parallèle** — un sous-agent `storm-researcher` par
   perspective mène une interview « rédacteur ↔ expert » (3 tours) **fondée sur
   de vraies recherches web** ; chaque affirmation est citée.
3. **Carte des contradictions** — où les voix s'affrontent, accord universel
   (probablement vrai), et l'**angle mort** que personne n'aborde.
4. **Plan** hiérarchique (brouillon puis affiné avec les trouvailles).
5. **Article cité** type Wikipédia, rédigé section par section (sous-agents
   `storm-writer` en parallèle), citations `[n]` globalisées et dédupliquées par
   URL.
6. **Briefing de synthèse** — résumé CEO 60 s, 5 findings classés par fiabilité,
   connexion cachée, insight actionnable, question frontière.
7. **Auto peer-review** — scores de confiance 1–10, maillon faible, biais,
   perspective manquante, note « professeur de Stanford » (corrige la faiblesse
   connue de STORM : absence d'auto-critique).

Le rapport final est sauvegardé en `storm-<sujet>.md` avec une section
**References** cliquable.

---

## Installation

### Option A — Marketplace local (persistant)

```bash
# 1. Ajouter ce dépôt comme marketplace (chemin local ou URL git)
/plugin marketplace add C:/Users/Zero/Desktop/storm-plugin-claude
# 2. Installer le plugin
/plugin install storm@storm-marketplace
# 3. Recharger
/reload-plugins
```

### Option B — Test rapide (le temps d'une session)

```bash
claude --plugin-dir C:/Users/Zero/Desktop/storm-plugin-claude
```

### Option C — Depuis GitHub (recommandé pour partager)

```bash
/plugin marketplace add hadufer/claude-storm
/plugin install storm@storm-marketplace
/reload-plugins
```

Vérifier le plugin avant publication :

```bash
claude plugin validate C:/Users/Zero/Desktop/storm-plugin-claude
```

---

## Utilisation

```bash
# Recherche complète, ancrée et citée (commande phare)
/storm:storm l'impact de l'IA générative sur l'éducation

# Réglages optionnels
/storm:storm voitures électriques --depth deep --lang fr
/storm:storm quantum computing --depth quick

# Briefing rapide en 1 passe, SANS web (la méthode exacte du tweet)
/storm:storm-brief stratégie de pricing SaaS --role fondateur
```

| Commande | Web ? | Sous-agents | Sortie |
|----------|:-----:|:-----------:|--------|
| `/storm:storm` | ✅ oui | ✅ parallèles | Rapport complet cité + briefing + peer-review, sauvegardé |
| `/storm:storm-brief` | ❌ non | ❌ non | Briefing 4-phases rapide (non sourcé) en chat |

### Arguments

- `--depth quick \| standard \| deep` — profondeur (défaut : `standard` =
  5 perspectives + basic, 3 tours d'interview).
- `--lang <code>` — langue du livrable (défaut : la langue de votre requête).
- `--no-web` — ignorer la recherche web (déconseillé ; préférez `storm-brief`).
- `--role <rôle>` (brief) — votre rôle, pour l'insight actionnable.

> 💡 Les commandes d'un plugin sont toujours préfixées par le nom du plugin :
> ici `storm`. D'où `/storm:storm` et `/storm:storm-brief`.

---

## Coût & performance

`/storm:storm` est **volontairement lourd** : il lance plusieurs sous-agents et
de nombreuses recherches web pour atteindre la qualité « 40–60 h de travail
humain ». Pour une réponse en quelques secondes sans appels web, utilisez
`/storm:storm-brief`.

Recommandé : lancer la commande phare sous **Opus** (qualité de synthèse) ; les
sous-agents `storm-researcher` tournent sous **Sonnet** par défaut pour la
vitesse et le parallélisme (comme STORM qui utilise un modèle léger pour les
conversations et un modèle fort pour la rédaction).

---

## Structure du plugin

```
storm-plugin-claude/
├── .claude-plugin/
│   ├── plugin.json          # manifeste (nom : "storm")
│   └── marketplace.json     # catalogue de distribution
├── skills/
│   ├── storm/
│   │   ├── SKILL.md         # /storm:storm — orchestrateur complet
│   │   └── reference.md     # prompts STORM exacts, conventions, gabarit
│   └── storm-brief/
│       └── SKILL.md         # /storm:storm-brief — méthode 4-prompts rapide
├── agents/
│   ├── storm-researcher.md  # interviewer ancré, guidé par perspective
│   └── storm-writer.md      # rédacteur de section cité
├── README.md
└── LICENSE
```

---

## Crédits

- **STORM** — Shao et al., *Assisting in Writing Wikipedia-like Articles From
  Scratch with Large Language Models*, NAACL 2024
  ([arXiv:2402.14207](https://arxiv.org/abs/2402.14207)),
  [stanford-oval/storm](https://github.com/stanford-oval/storm) (MIT).
  Démo live : [storm.genie.stanford.edu](https://storm.genie.stanford.edu).
- **Adaptation 4-prompts** — Nav Toor
  ([@heynavtoor](https://x.com/heynavtoor/status/2067194761446920264)).

Plugin indépendant, sans affiliation avec l'Université de Stanford. Licence MIT.

# Fluence : plan d'implémentation

Application web vocale d'entraînement à la récupération lexicale et à la fluence verbale.
Cible : iPhone (Safari et web app installée sur l'écran d'accueil), utilisation en marchant en forêt, seul et à voix haute, écouteurs avec micro.
Nom de travail : **Fluence**. Version du plan : 1, 26/09/2026.

Ce document est destiné à Claude Code. Place-le à la racine d'un dépôt vide sous le nom `PLAN.md`.

---

## 0. Mode d'emploi (pour Sacha)

1. Crée un dépôt GitHub vide, dépose ce fichier en `PLAN.md`.
2. Lance Claude Code avec le prompt de la section 10.1. Il exécute la **phase 0 seulement** (page de tests iPhone) et s'arrête.
3. Tu fais les tests sur ton iPhone, dont un en forêt sur ton parcours habituel, en marchant avec tes écouteurs. Tu colles les résultats dans `spike-report.md`.
4. Décision GO / NO-GO (section 8, phase 0). Ensuite une phase = une session Claude Code = une validation par toi sur l'iPhone.

---

## 1. Ce qui change par rapport au prompt Gemini

| # | Prompt Gemini | Problème | Correction |
|---|---|---|---|
| 1 | « L'app compte les mots reconnus » | La reconnaissance renvoie des phrases : « euh », « et », doublons, erreurs de transcription et mots hors sujet seraient comptés. Le score ne mesurerait rien. | Normalisation, lexique par catégorie, liste de candidats « hors lexique » à valider après la marche. |
| 2 | « Sprint Synonymes / Circonlocution » | Deux compétences distinctes fusionnées. Synonyme : remplacer le mot. Circonlocution : expliquer sans le mot, ce que tu fais sur scène quand il ne vient pas. | Deux exercices séparés. |
| 3 | Pas d'exercice « du sens au mot » | Le mot sur le bout de la langue, c'est : je sais ce que je veux dire, le mot ne sort pas. L'exercice le plus proche de ton problème est absent. | Exercice définition → mot avec indices graduels (contexte, puis première syllabe). Il devient l'exercice central. |
| 4 | Aucune mémoire | Un mot raté disparaît et ne revient jamais. | Révision espacée des mots ratés + carnet des mots perdus en situation réelle. |
| 5 | « Eyes-free / Hands-free » annoncé, pas conçu | Aucun signal sonore de début/fin d'écoute, rien sur la mise en veille (écran verrouillé = micro coupé), rien contre les appuis involontaires en poche. | Signaux sonores dédiés, écran maintenu allumé, appui long obligatoire, commandes vocales. |
| 6 | Écrit implicitement pour Chrome | Sur iPhone, tous les navigateurs utilisent le moteur WebKit, et la reconnaissance vocale web y est moins fiable (mode continu, résultats intermédiaires). | Phase 0 de test sur ton iPhone avant tout code. Architecture qui reste utile si la reconnaissance échoue. |
| 7 | 10 s pour 3 synonymes | Sur iOS, la transcription arrive souvent en fin de segment : la latence mange une partie des 10 s. | 15 s, arrêt anticipé dès 3 synonymes valides. |
| 8 | Phonémique en 45 s | Durée non standard, résultats non comparables. | 60 s, format des épreuves de type FAS. |
| 9 | Catégorie « Modèles mentaux en management » | Produit surtout des noms propres (Pareto, Eisenhower) et des expressions longues : difficile à valider, contradictoire avec la règle « pas de noms propres ». | Catégories qui produisent des noms communs et des verbes. |
| 10 | « Génère le code complet » + « n'ajoute rien » | Un bloc de code livré d'un seul coup n'est pas testable. La robustesse demandée exige un mode debug, un mode simulation et des tests automatisés. | Livraison par phases avec critères d'acceptation. |

---

## 2. Intention

- Stimuler le cerveau pendant la marche en forêt, seul, à voix haute.
- Gagner en aisance et en confiance avant les prises de parole en public et les formations.
- Faire passer le temps de marche.

Conséquences sur la conception :
- **Ton encourageant.** Le feedback annonce d'abord ce qui a été trouvé. Un mot raté n'est pas commenté : la réponse est donnée, le mot reviendra plus tard.
- **Records annoncés seulement quand ils sont battus.**
- **Durée de session calée sur la marche** (15, 30 ou 45 min), sans item répété dans une même session.

---

## 3. Critères de succès (fixés avant de coder)

| Niveau | Critère | Seuil |
|---|---|---|
| Technique | Une session complète se déroule sans toucher l'écran | 3 marches consécutives sans intervention |
| Usage | L'app fait partie de la marche | Lancée à chaque marche sans effort |
| Contenu | Pas de lassitude | Aucun item répété dans une session ; banques assez larges pour 4 semaines sans redite fréquente |

Validation : Sacha, 3 semaines après la mise en service.

---

## 4. Contraintes techniques iPhone

| Point | Statut | Conséquence |
|---|---|---|
| `webkitSpeechRecognition` présent dans Safari iOS | Fait | Base web possible |
| Mode continu et résultats intermédiaires instables sur iOS | Fait, signalé par des développeurs ([forum Apple](https://developer.apple.com/forums/thread/775699)) | Ne pas dépendre des résultats intermédiaires. Si le résultat final n'arrive pas, le dernier intermédiaire vaut final. |
| Reconnaissance iOS dépendante du réseau | Fait, même source (échec hors connexion) | **En forêt, la 4G est incertaine.** Bascule automatique en mode sans reconnaissance quand le réseau tombe, retour automatique quand il revient, annonce vocale dans les deux cas. |
| Batterie | Écran allumé + micro + 4G pendant 45 min : consommation notable, non chiffrée | Mesure au spike (T7) |
| Écran verrouillé ou Safari en arrière-plan : page suspendue, micro coupé | Très probable, à vérifier (T7) | Écran allumé pendant toute la session |
| Screen Wake Lock API | Supportée par Safari depuis iOS 16.4. Comportement en web app écran d'accueil à vérifier | Test T7. Repli : Réglages > Luminosité > Verrouillage auto : Jamais, pendant la marche |
| Mode Économie d'énergie | Force le verrouillage automatique à 30 s | Le désactiver pendant la marche |
| `navigator.vibrate` | Non supporté sur iOS | Aucun retour haptique : tout passe par le son |
| Relance du micro par programme, sans toucher l'écran | **Inconnu. Point bloquant.** | Test T4. Si iOS exige un toucher à chaque écoute, le mains libres web est impossible : plan B (section 9) |
| Voix de synthèse françaises | Voix système fr-FR disponibles. Exposition au web des voix « améliorées » à vérifier | Test T1 |
| Stockage local | Safari peut effacer le stockage d'un site non visité depuis 7 jours. Les web apps installées sur l'écran d'accueil sont traitées à part | Installation sur l'écran d'accueil + export/import JSON |
| Confidentialité | L'audio part vraisemblablement vers les serveurs Apple pour la transcription | Transcriptions et historique stockés en local uniquement |

---

## 5. Spécification fonctionnelle

### 5.1 Principes de conception

1. **L'app reste utile si la reconnaissance échoue ou si le réseau tombe en forêt.** La valeur d'entraînement vient de l'effort de production sous contrainte de temps et de l'écoute des réponses modèles. Le score vient en second. Chaque exercice fonctionne donc aussi en mode `asr: off` : fenêtre chronométrée, pas de score, réponses modèles lues.
2. **Micro fermé pendant que l'app parle.** Sinon la reconnaissance transcrit la synthèse vocale.
3. **Messages courts.** Consigne < 8 s, feedback < 6 s, fin de session en 3 phrases.
4. **Chaque changement d'état a un signal sonore propre.**
5. **Aucun réglage avant de partir.** Un toucher sur la durée choisie, la session démarre.

### 5.2 Déroulé d'une session

Écran d'accueil : trois gros boutons, **15 / 30 / 45 min**. La session enchaîne des cycles d'environ 10 min jusqu'à la durée choisie (le cycle en cours se termine). Le contenu tourne : aucun item répété dans une session.

Un cycle :

| Ordre | Bloc | Durée | Contenu |
|---|---|---|---|
| 1 | Accueil (premier cycle seulement) | 5 s | « Trente minutes. On commence. » |
| 2 | A. Fluence sémantique | 60 s | 1 catégorie |
| 3 | B. Fluence phonémique | 60 s | 1 lettre |
| 4 | C. Du sens au mot | ≈ 3 min 30 | 6 items, révisions en priorité |
| 5 | D. Synonymes | ≈ 2 min | 5 items (adjectifs, verbes, connecteurs) |
| 6 | E. Parler sans le mot | ≈ 1 min 30 | 2 items de 30 s |
| 7 | Fin de cycle | 5 s | Une phrase : « Cycle deux terminé. Vingt et un mots trouvés. » En fin de session : trois phrases maximum, records battus inclus. |

Composition et durées modifiables dans `config.js`.

### 5.3 Les cinq exercices

**A. Fluence sémantique (60 s)**
- Consigne : « Catégorie : les biais cognitifs. Soixante secondes. » Signal `go`.
- Signal `warn` à 10 s de la fin, signal `end`.
- Score : nombre d'entrées distinctes du lexique trouvées. Les variantes sont regroupées (« ancrage » et « effet d'ancrage » comptent pour 1).
- Les mots hors lexique (hors mots-outils) deviennent des **candidats**, affichés dans l'historique et validables d'un toucher : ils rejoignent alors le lexique personnel.
- Feedback : « Quatorze termes. » (+ « Nouveau record. » s'il est battu) puis deux termes du lexique non cités (exposition).
- Si les horodatages sont exploitables (test T3) : répartition par tranches de 15 s. Optionnel.

**B. Fluence phonémique (60 s)**
- Consigne : « Des mots qui commencent par P, comme pomme. Pas de noms propres, pas de mots de la même famille. » Le mot d'ancrage (nom commun, pour ne pas contredire la consigne) lève l'ambiguïté P/T dans le vent.
- Lettres : P, R, V, M, F, C, T, D, S, L, B. Pas deux fois la même lettre sur 5 sessions.
- Score : mots distincts dont la forme normalisée commence par la lettre. Exclus : mots-outils, doublons, même famille (préfixe commun ≥ 5 lettres avec un mot déjà compté, approximation assumée), noms propres probables (majuscule hors début de segment, si iOS capitalise : test T6).
- Validation par règle, pas de lexique.

**C. Du sens au mot (exercice central)**

Hiérarchie d'indices, du plus faible au plus fort :

1. Définition lue, écoute 12 s. Arrêt immédiat si le mot est détecté, signal `hit`.
2. Indice de contexte : phrase à trou lue, écoute 8 s.
3. Indice phonologique : « Ça commence par OB. Trois syllabes. », écoute 8 s.
4. Réponse : « Le mot était : obsolète. Répète-le. », écoute 4 s (répétition vérifiée, non notée).

- Score : 3 / 2 / 1 / 0 selon l'étape où le mot est trouvé.
- **Latence** enregistrée (ms entre la fin de la consigne et la première détection) : meilleure mesure de la fluidité de récupération.
- Correspondance tolérante : forme normalisée, variantes, clé phonétique, distance d'édition ≤ 1 pour les mots de 6 lettres et plus, test sur toutes les alternatives de transcription.
- Révision espacée, système de Leitner à 5 boîtes :
  - trouvé sans indice : boîte + 1 ; trouvé avec indice : même boîte ; échec : boîte 1 ;
  - boîte 1 : chaque session ; boîte 2 : une session sur deux ; boîte 3 : une sur quatre ; boîte 4 : une sur huit ; boîte 5 : acquis, sort de la rotation.
- Sélection des 6 items : d'abord les révisions dues (4 max), puis les mots du carnet jamais vus, puis des items neufs par niveau croissant.

**D. Synonymes (15 s par item)**
- Consigne avec la classe grammaticale : « L'adjectif : obsolète. »
- Arrêt anticipé dès 3 synonymes valides.
- Feedback : « Deux sur trois. Tu avais aussi : caduc, suranné. » (2 manquants cités au maximum).
- Trois familles d'items : adjectifs et noms du conseil ; verbes ; **connecteurs et tics de langage** (« du coup », « en fait », « cependant », « donc »). Les connecteurs sont les mots qui s'usent le plus à l'oral.

**E. Parler sans le mot (circonlocution, 30 s)**
- Consigne : « Explique : retour sur investissement. Sans dire : retour, investissement, argent, rentable, rapporter. » Chaque racine interdite correspond à un mot annoncé.
- Si les résultats intermédiaires fonctionnent (T3) : signal `buzz` immédiat dès qu'un mot interdit est détecté. Sinon, détection en fin de fenêtre.
- Mesures : mots interdits prononcés, nombre de mots (débit), durée de parole estimée.
- Feedback : « Aucun mot interdit. Soixante-dix mots. »
- C'est la situation réelle : le mot ne vient pas, tu continues sans lui.

### 5.4 Contrôle mains libres

| Action | Moyen |
|---|---|
| Démarrer | Toucher le grand bouton (obligatoire sur iOS : débloque l'audio et le micro) |
| Pause / reprise | Appui long 1 s n'importe où, ou dire « pause » / « reprends » |
| Arrêter | Appui long 3 s, ou dire « stop session » |
| Répéter la consigne | Dire « répète » |
| Passer l'item | Dire « passe » |
| Toucher court | Ignoré (protection poche) |

- Une commande vocale n'est reconnue que si le segment transcrit ne contient qu'elle (évite les faux positifs).
- En pause, écoute légère en boucle pour « reprends ». Si T4 montre que ce n'est pas tenable, reprise par appui long uniquement.
- Reprise après pause : l'item en cours recommence du début.
- Bouton de tige des AirPods via Media Session API : bonus non bloquant, selon T8.

### 5.5 Signaux sonores

Générés avec Web Audio API, aucun fichier audio. `AudioContext` créé et débloqué au premier toucher.

| Signal | Son | Moment |
|---|---|---|
| `go` | 2 notes montantes | Micro ouvert, à toi |
| `warn` | 3 tics courts | 10 s avant la fin (fenêtres ≥ 30 s) |
| `end` | 2 notes descendantes | Micro fermé |
| `hit` | 1 note aiguë brève | Mot cible ou synonyme valide détecté |
| `buzz` | 1 note grave | Mot interdit |
| `error` | 3 notes graves | Problème (réseau, micro), suivi d'une annonce vocale |

### 5.6 Écrans

- **Session** : fond noir (économie batterie sur écran OLED), un seul bloc plein écran qui fait bouton. Lisible d'un coup d'œil : nom de l'exercice, compte à rebours très gros, état par couleur (l'app parle / j'écoute / pause), dernière transcription en petit.
- **Accueil** : trois gros boutons de durée (15 / 30 / 45 min), accès discret à l'historique.
- **Historique** (consulté à la maison) : sessions, records, mots à revoir, candidats hors lexique à valider, carnet des mots perdus, export/import JSON.
- **Debug** (`?debug=1`) : panneau de logs horodatés (événements de reconnaissance, relances, erreurs, latences).

---

## 6. Architecture

### 6.1 Stack

- HTML, CSS, JavaScript vanilla en modules ES. Aucun framework, aucune étape de build, aucune dépendance à l'exécution.
- Dépendances de développement uniquement : Node ≥ 20 (`node --test` pour les tests unitaires), Playwright pour les tests de bout en bout en mode simulation.
- Hébergement : GitHub Pages (HTTPS obligatoire pour le micro).
- PWA : manifeste + service worker qui met en cache l'interface et les données. La reconnaissance reste dépendante du réseau.

### 6.2 Arborescence

```
/
├── PLAN.md
├── CLAUDE.md                  conventions du projet (créé en phase 1)
├── index.html
├── manifest.webmanifest
├── sw.js
├── css/
│   ├── client-first.css       classes utilitaires Client-First
│   └── app.css                composants (accueil_, session_, historique_, carnet_)
├── js/
│   ├── main.js                amorçage, liaison UI
│   ├── config.js              durées, composition de session, options
│   ├── audio/
│   │   ├── tts.js             synthèse vocale : file d'attente, voix fr-FR, minuterie de secours
│   │   ├── asr.js             reconnaissance : fenêtres d'écoute, relances, alternatives
│   │   ├── asr-sim.js         simulation : champ texte à la place du micro (tests, dev)
│   │   ├── asr-off.js         mode sans reconnaissance : fenêtre chronométrée seule
│   │   ├── earcons.js         signaux sonores Web Audio
│   │   └── device.js          wake lock, appui long, Media Session
│   ├── engine/
│   │   ├── session.js         orchestrateur de session
│   │   ├── commands.js        commandes vocales
│   │   └── exercises/
│   │       ├── semantic.js
│   │       ├── phonemic.js
│   │       ├── definition.js
│   │       ├── synonyms.js
│   │       └── circumlocution.js
│   ├── scoring/
│   │   ├── normalize.js       normalisation, mots-outils, singulier, élisions
│   │   ├── phonetic.js        clé phonétique française simplifiée
│   │   └── match.js           n-grammes, lexique, tolérance, familles de mots
│   └── store/
│       ├── history.js         historique des sessions, records
│       ├── leitner.js         révision espacée
│       └── backup.js          export / import JSON
├── data/
│   ├── categories.json
│   ├── letters.json
│   ├── definitions.json
│   ├── synonyms.json
│   ├── circumlocution.json
│   └── stopwords-fr.json
├── scripts/
│   └── check-data.mjs         validation des banques de données
├── tests/
│   ├── unit/                  node --test
│   └── e2e/                   Playwright, mode ?sim=1
└── spike/
    ├── spike.html
    └── spike-report.md
```

### 6.3 Contrats des modules

**`asr.js`**, le module le plus critique :

```js
// Ouvre une fenêtre d'écoute de durée fixe et renvoie tout ce qui a été entendu.
listen({ durationMs, earlyStop, onSegment, signal })
  → Promise<{ segments: [{ text, alternatives, tStart, tEnd, isFinal }], restarts, errors }>
```

Exigences :
- `const SR = window.SpeechRecognition || window.webkitSpeechRecognition` ; `lang = 'fr-FR'` ; `maxAlternatives = 5` ; `continuous` et `interimResults` pilotés par `config.js` selon les résultats du spike.
- **Relance automatique** sur `end` tant que la fenêtre n'est pas écoulée (délai 250 ms, plafond de relances), avec journalisation du trou entre deux écoutes.
- **Agrégation par cycle d'écoute** : à chaque cycle, conserver la dernière transcription complète (concaténation de `event.results[i][0].transcript`) ; la valider en fin de cycle. Fonctionne que le moteur renvoie des résultats incrémentaux ou une transcription cumulée.
- **Repli iOS** : si aucun `isFinal` n'arrive, le dernier résultat intermédiaire est traité comme final.
- **Fin de fenêtre** : `stop()` puis attente des derniers résultats 1 500 ms max, puis `abort()`.
- **`earlyStop(segments)`** : prédicat évalué à chaque résultat (mot trouvé, 3 synonymes valides).
- **Erreurs** : `no-speech` → relance silencieuse ; `network` → signal `error` + annonce + pause ; `not-allowed` / `service-not-allowed` → écran d'explication ; `audio-capture` → annonce ; `aborted` → ignoré si voulu.
- Même interface pour `asr-sim.js` et `asr-off.js`.

**`tts.js`** :
- File d'attente, découpage par phrases, débit réglable.
- Sélection de voix fr-FR après `voiceschanged` ; repli sur la voix par défaut avec `lang = 'fr-FR'`.
- `onend` + minuterie de secours (≈ 70 ms par caractère + 1,5 s), car `onend` ne se déclenche pas toujours.
- `speechSynthesis.cancel()` avant chaque ouverture du micro.

**`session.js`** :
- Orchestrateur en `async/await` avec `AbortController` pour pause et arrêt.
- État exposé à l'UI et aux logs : `idle | speaking | listening | feedback | paused | summary | error`.
- Pause = abandon de l'étape en cours + mémorisation de l'index d'item.

**Interface d'un exercice** :

```js
export default {
  id: 'definition',
  label: 'Du sens au mot',
  pickItems(bank, store, n),        // choisit les items (révisions, carnet, neufs)
  async run(item, ctx),             // ctx = { say, cue, listen, signal, log, asrMode }
  summarize(results)                // → { score, spoken, details }
}
```

### 6.4 Normalisation et correspondance

- Minuscules → `œ` → `oe`, `æ` → `ae` → suppression des accents (NFD) → élisions retirées (`l'`, `d'`, `qu'`, `j'`, `n'`, `s'`, `c'`, `m'`, `t'`) → ponctuation → espaces.
- Mots-outils français (liste JSON) exclus du comptage.
- Singulier approximatif : `-s` / `-x` final retiré si le mot fait plus de 3 lettres.
- Expressions du lexique : correspondance par n-grammes, la plus longue d'abord, jusqu'à 4 mots (« mettre en œuvre », « coûts irrécupérables »).
- Clé phonétique française simplifiée, appliquée après normalisation : `ph→f`, `qu→k`, `c+e/i/y→s`, `c→k`, `g+e/i→j`, `gu+e/i→g`, `eau/au→o`, `ai/ei→e`, `am/an/em/en→an`, `om/on→on`, `ain/ein/im/in→in`, `h` supprimé, lettres doublées réduites, consonnes finales muettes (`t d s x p`) et `e` final supprimés.
- Distance de Levenshtein ≤ 1 pour les mots de 6 lettres et plus.
- Famille de mots (exercice B) : préfixe commun ≥ 5 lettres.
- Racines interdites (exercice E) : un token est interdit s'il commence par une racine de la liste.

### 6.5 Conventions HTML/CSS : Client-First (Finsweet)

- Structure : `page-wrapper` > `main-wrapper` > `section_[nom]` > `padding-global` > `container-[taille]` > `padding-section-[taille]`.
- Classes de composant : `[composant]_[élément]` (`session_button`, `session_timer`, `historique_list`, `carnet_form`).
- Classes utilitaires Client-First (`text-size-*`, `text-weight-*`, `text-align-center`, `margin-*`, `padding-*`, `hide`).
- États en classes combo `is-*` : `is-speaking`, `is-listening`, `is-paused`, `is-error`.
- Unités en `rem`, couleurs en variables CSS. Pas d'ID pour le style ; accroches JS par attributs `data-*`.

```html
<div class="page-wrapper">
  <main class="main-wrapper">
    <section class="section_session">
      <div class="padding-global">
        <div class="container-small">
          <button class="session_button" data-session-button>
            <span class="session_exercise text-size-medium" data-exercise-label>Prêt</span>
            <span class="session_timer" data-timer>▶</span>
            <span class="session_transcript text-size-small" data-transcript></span>
          </button>
        </div>
      </div>
    </section>
  </main>
</div>
```

---

## 7. Données

Les items générés par Claude Code portent `"a_valider": true`. Sacha les relit (dans le JSON ou en demandant une relecture à Claude) avant de retirer le marqueur. `scripts/check-data.mjs` vérifie la structure, les doublons, l'absence de mot cible dans sa propre définition, et que chaque racine interdite (exercice E) correspond à un mot annoncé.

Volumes visés en V1 : 20 catégories de 40 entrées minimum, 11 lettres, 60 définitions, 50 items de synonymes, 30 items « parler sans le mot ».

### 7.1 `categories.json`

```json
[
  {
    "id": "biais-cognitifs",
    "consigne": "les biais cognitifs",
    "lexique": [["ancrage", "effet d'ancrage"], ["confirmation", "biais de confirmation"], ["statu quo"], ["aversion à la perte"], ["coûts irrécupérables"], ["excès de confiance"], ["effet de halo"], ["disponibilité"]],
    "a_valider": true
  }
]
```

Catégories de départ (compléter chaque lexique à 40 entrées minimum) :

| Consigne | Exemples |
|---|---|
| les biais cognitifs | ancrage, confirmation, statu quo, aversion à la perte |
| les indicateurs de performance d'une entreprise | chiffre d'affaires, marge, trésorerie, panier moyen, BFR |
| les verbes pour décrire un changement | transformer, basculer, pivoter, réformer, remanier |
| les qualités d'un bon orateur | clarté, aisance, éloquence, concision, conviction |
| les risques d'un projet numérique | retard, dérive budgétaire, dette technique, faille de sécurité |
| les composantes d'une identité de marque | logo, typographie, couleurs, ton, promesse, signature |
| les émotions d'un client mécontent | frustration, colère, déception, méfiance, lassitude |
| les outils de gestion de crise | cellule de crise, porte-parole, communiqué, plan de continuité |
| les figures de style utiles à l'oral | métaphore, anaphore, antithèse, hyperbole, litote |
| les verbes d'action du consultant | diagnostiquer, arbitrer, prioriser, structurer, piloter |
| les métiers de la communication | graphiste, rédacteur, directeur artistique, attaché de presse |
| le vocabulaire de l'immobilier | mandat, compromis, estimation, copropriété, plus-value |

### 7.2 `letters.json`

```json
[{ "lettre": "P", "ancrage": "pomme" }, { "lettre": "R", "ancrage": "rivière" }, { "lettre": "V", "ancrage": "vélo" }]
```

Lettres : P, R, V, M, F, C, T, D, S, L, B.

### 7.3 `definitions.json`

```json
[
  {
    "id": "obsolete",
    "mot": "obsolète",
    "variantes": [],
    "classe": "adjectif",
    "definition": "Qui n'est plus en usage parce que quelque chose de plus récent l'a remplacé.",
    "contexte": "Ce logiciel n'est plus mis à jour depuis dix ans : il est devenu…",
    "debut": "ob",
    "syllabes": 3,
    "niveau": 1
  },
  {
    "id": "corroborer",
    "mot": "corroborer",
    "variantes": [],
    "classe": "verbe",
    "definition": "Confirmer une idée ou un témoignage en apportant des éléments supplémentaires.",
    "contexte": "Les chiffres du trimestre viennent … notre intuition de départ.",
    "debut": "cor",
    "syllabes": 4,
    "niveau": 2
  },
  {
    "id": "serendipite",
    "mot": "sérendipité",
    "variantes": [],
    "classe": "nom",
    "definition": "Le fait de découvrir par hasard ce qu'on ne cherchait pas, et de savoir en tirer parti.",
    "contexte": "La pénicilline est née d'une boîte de culture oubliée : un cas d'école de …",
    "debut": "sé",
    "syllabes": 5,
    "niveau": 3
  }
]
```

Mots cibles à rédiger ensuite : pérenne, idoine, prépondérant, paradigme, itératif, dichotomie, péremptoire, éluder, exhaustif, entériner, inhérent, galvaniser, circonscrire, désuet, heuristique, catalyseur, fédérer, pragmatique, tangible, subsidiaire, ubiquité, antinomique, vernaculaire, ambivalent, empirique, sibyllin, contingent, fallacieux, prégnant, granularité, désintermédiation, obsolescence, arbitrer, levier, pertinence.

### 7.4 `synonyms.json`

```json
[
  { "id": "obsolete", "mot": "obsolète", "classe": "adjectif", "synonymes": ["dépassé", "périmé", "caduc", "désuet", "suranné", "révolu", "démodé"] },
  { "id": "implementer", "mot": "implémenter", "classe": "verbe", "synonymes": ["mettre en œuvre", "déployer", "appliquer", "instaurer", "intégrer", "mettre en place", "installer"] },
  { "id": "problematique", "mot": "problématique", "classe": "nom", "synonymes": ["enjeu", "difficulté", "question", "défi", "problème", "point de blocage"] },
  { "id": "important", "mot": "important", "classe": "adjectif", "synonymes": ["majeur", "crucial", "essentiel", "capital", "déterminant", "primordial", "décisif", "fondamental"] },
  { "id": "cependant", "mot": "cependant", "classe": "connecteur", "synonymes": ["toutefois", "néanmoins", "pourtant", "en revanche", "or", "cela dit"] },
  { "id": "du-coup", "mot": "du coup", "classe": "tic", "synonymes": ["donc", "alors", "par conséquent", "de ce fait", "ainsi", "dès lors"] }
]
```

Items à rédiger ensuite : améliorer, difficile, montrer, rapide, changer, utiliser, clair, augmenter, réduire, expliquer, efficace, flou, risqué, décider, convaincre, donc, de plus, par exemple, en fait, enfin.

### 7.5 `circumlocution.json`

```json
[
  { "id": "roi", "concept": "retour sur investissement", "annonce": ["retour", "investissement", "argent", "rentable", "rapporter"], "racines": ["retour", "invest", "argent", "rentab", "rapport"] },
  { "id": "automatisation", "concept": "automatisation", "annonce": ["automatique", "machine", "robot", "tâche", "répétitif"], "racines": ["automat", "machine", "robot", "tache", "repet"] },
  { "id": "delegation", "concept": "délégation", "annonce": ["déléguer", "confier", "tâche", "responsabilité"], "racines": ["deleg", "confi", "tache", "respons"] }
]
```

Concepts à rédiger ensuite : effet de levier, image de marque, résilience, marge, négociation, intelligence artificielle, prototype, fidélisation, bouche-à-oreille, trésorerie, recrutement, cahier des charges, facturation, veille concurrentielle, persona.

---

## 8. Phases

Chaque phase se termine par : tests passés, liste de vérifications à faire sur l'iPhone, arrêt. Ne pas enchaîner sur la phase suivante sans validation.

### Phase 0 : test de compatibilité iPhone (spike)

Livrables : `spike/spike.html` (autonome, gros boutons, logs à l'écran, bouton « Copier le rapport » qui copie les résultats en JSON), `spike/spike-report.md` (grille à remplir), instructions de déploiement sur GitHub Pages.

| Test | Contenu | Relevé |
|---|---|---|
| T1 Voix | Lister les voix fr ; lire une phrase ; `onend` reçu ? | Voix disponibles, `onend` sur 10 lectures |
| T2 Reconnaissance au calme | Lire les 20 mots de test | Taux de mots corrects |
| T3 Fenêtre de 60 s | `continuous=true` puis relances successives | `end` spontanés, intermédiaires reçus ?, `isFinal` reçu ?, trou entre relances (ms) |
| **T4 Chaîne sans les mains** | 10 cycles synthèse → signal → écoute 8 s → synthèse, sans toucher l'écran | Cycles réussis sur 10 |
| T5 Alternatives | `maxAlternatives = 5` | Nombre d'alternatives renvoyées |
| T6 Majuscules | Dire « Paris, pomme, Pierre, poire » | Noms propres capitalisés ? |
| T7 Écran et batterie | Wake lock 30 min, en onglet Safari puis en web app installée | L'écran reste allumé ? % de batterie consommé |
| T8 AirPods | Micro des écouteurs, volume de la synthèse après une écoute, bouton de tige | Observations |
| **T9 Terrain** | En forêt, sur ton parcours, en marchant, écouteurs : les 20 mots de test | Taux de mots corrects |
| T10 Hors réseau | Mode avion, puis zone blanche réelle de ton parcours | Erreur renvoyée, endroits où la 4G décroche |

Mots de test : obsolète, paradigme, itératif, trésorerie, arbitrage, heuristique, prépondérant, dichotomie, sérendipité, catalyseur, corroborer, galvaniser, circonscrire, entériner, pérenne, idoine, ubiquité, péremptoire, subsidiaire, vernaculaire.

**Décision** : GO web si T4 = 10/10 et T9 ≥ 80 %. Sinon, plan B (section 9).

### Phase 1 : socle audio et moteur

- `tts.js`, `earcons.js`, `asr.js`, `asr-sim.js`, `asr-off.js`, `session.js`, `device.js` (wake lock seulement).
- Écran de session Client-First.
- Exercice C (du sens au mot) de bout en bout, avec 10 items de test.
- Mode debug `?debug=1`, mode simulation `?sim=1`, mode sans reconnaissance `?asr=off`.
- `CLAUDE.md` : conventions (Client-First, commentaires en français, pas de dépendance à l'exécution, commandes de test).

Acceptation : en simulation, un test Playwright exécute une session de 3 items et vérifie les états et scores ; sur l'iPhone, 10 items enchaînés sans toucher l'écran.

### Phase 2 : normalisation et scoring

- `normalize.js`, `phonetic.js`, `match.js`.
- Tests unitaires : 40 cas minimum couvrant élisions, accents, ligatures, pluriels, n-grammes, clé phonétique, tolérance d'édition, familles de mots, racines interdites, noms propres.

Acceptation : `node --test` au vert ; aucun faux positif sur une liste de 20 paires « proches mais différentes » (ex. « obsolète » / « absolu »).

### Phase 3 : les quatre autres exercices et les données

- Exercices A, B, D, E.
- Banques de données aux volumes de la section 7, marquées `a_valider`.
- `scripts/check-data.mjs`.

Acceptation : session complète d'environ 10 min en simulation ; `check-data` sans erreur.

### Phase 4 : mains libres et robustesse

- Appui long (1 s pause, 3 s arrêt), toucher court ignoré, commandes vocales.
- Gestion complète des erreurs de reconnaissance et du réseau, avec annonces vocales.
- Reprise du wake lock sur `visibilitychange`.
- Détection de perte réseau (`offline` + erreur `network`) : bascule en mode sans reconnaissance, annonce vocale, retour automatique au mode normal quand le réseau revient.
- Sessions de 15 / 30 / 45 min en cycles, rotation du contenu sans répétition.
- Media Session si T8 positif.

Acceptation : sur l'iPhone, une session de 30 min en forêt sans toucher l'écran ; passage en zone blanche annoncé, exercices poursuivis, retour au mode normal au retour du réseau.

### Phase 5 : mémoire et suivi

- Historique des sessions et records.
- Leitner (exercice C).
- **Carnet des mots perdus** : saisie d'un mot qui t'a échappé en vrai (mot + définition obligatoires, phrase à trou facultative ; début et syllabes calculés, modifiables). Il entre dans la rotation de l'exercice C en priorité.
- Écran historique : sessions, records, candidats hors lexique à valider.
- Export / import JSON.

Acceptation : un mot saisi au carnet revient à la session suivante ; export puis import restaure l'état complet.

### Phase 6 : PWA, déploiement, terrain

- Manifeste, icône, service worker, installation sur l'écran d'accueil.
- Déploiement GitHub Pages.
- Protocole terrain : 3 sorties, relevé des incidents dans `terrain.md`.

Acceptation : les critères techniques de la section 3.

### Hors périmètre V1

Évaluation par Claude des réponses hors lexique, génération automatique d'items, transcription serveur, statistiques avancées.

---

## 9. Risques et plan B

Inversion : ce qui ferait échouer le projet, par probabilité décroissante.

1. **Tu arrêtes de t'en servir après deux semaines.** Réponse : un toucher pour démarrer, contenu qui tourne sans redite, records, des mots issus de ta vraie vie (carnet).
2. **iOS ne relance pas le micro sans toucher (T4 en échec).** Réponse : plan B.
3. **Zones blanches en forêt.** La reconnaissance iOS a besoin du réseau. Réponse : bascule automatique en mode sans reconnaissance ; les exercices continuent, seul le score disparaît.
4. **Vent et bruit dégradent la transcription.** Réponse : écouteurs avec micro, test T9, mode `asr: off` disponible à tout moment.
5. **Lexique trop pauvre, scores injustes, démotivation.** Réponse : validation des candidats hors lexique en un toucher.

**Plan B**, si la reconnaissance web iOS n'est pas exploitable :

| Option | Principe | Coût | Limite |
|---|---|---|---|
| B1 | Mode `asr: off` : fenêtres chronométrées, réponses modèles lues, pas de score | 0 € | Pas de mesure objective |
| B2 | Micro ouvert une seule fois (un toucher), enregistrement de chaque fenêtre, transcription par un service de reconnaissance en fin d'exercice | Quelques centimes par session + un petit serveur | Latence du score, clé API à protéger derrière un proxy |
| B3 | App iOS native (Capacitor + reconnaissance Apple sur l'appareil) | Mac, Xcode, compte développeur Apple payant | Chantier nettement plus lourd |

B1 est intégré dès la phase 1 : l'app est utilisable même en cas de NO-GO.

---

## 10. Prompts pour Claude Code

### 10.1 Lancement (phase 0)

```
Lis PLAN.md en entier. Projet : Fluence, application web vocale d'entraînement lexical pour iPhone.
Exécute uniquement la phase 0 (test de compatibilité iPhone).
Livre spike/spike.html, spike/spike-report.md, et les étapes pour le publier sur GitHub Pages afin de l'ouvrir en HTTPS sur mon iPhone.
N'écris aucun code des phases suivantes. Arrête-toi et attends mon rapport.
Conventions : HTML/CSS Client-First (Finsweet), JS commenté en français pour les parties complexes, aucun framework, aucune dépendance à l'exécution.
```

### 10.2 Phases suivantes

```
Lis PLAN.md, CLAUDE.md et spike/spike-report.md. Exécute la phase N.
Règle config.js selon les résultats du spike (continuous, interimResults, maxAlternatives).
Respecte les critères d'acceptation de la phase, lance les tests, puis donne-moi la liste des vérifications à faire sur mon iPhone.
Ne commence pas la phase N+1.
```

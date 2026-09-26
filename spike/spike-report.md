# Fluence : rapport du spike iPhone (phase 0)

Page de test : https://millotsacha-sudo.github.io/FLUENCE/spike/spike.html

## Publication sur GitHub Pages

1. Fusionner la branche de travail dans `main` (la page n'est servie que depuis `main`).
2. Sur GitHub : Settings > Pages > Source : « Deploy from a branch », branche `main`, dossier `/ (root)`, Save.
3. Attendre 1 à 2 min (onglet Actions : « pages build and deployment » au vert), puis ouvrir l'URL ci-dessus dans Safari sur l'iPhone.
4. Autoriser le micro et la reconnaissance vocale quand Safari le demande.

## Avant de commencer

- iOS à jour. Noter la version : Réglages > Général > Informations.
- Mode Économie d'énergie désactivé.
- Réglages > Safari > Avancé : ne rien modifier.
- Ordre conseillé : T1, T2 (mode manuel : c'est lui qui déclenche la demande d'autorisation du micro), T5, T6, T3, T4, T8, T7, T10, puis T9 en forêt.
- **Stockage séparé** : les résultats enregistrés dans l'onglet Safari et ceux de la web app installée (écran d'accueil) ne sont pas partagés. Copier le rapport depuis chacun des deux, et coller les deux JSON ci-dessous.
- Installation sur l'écran d'accueil (pour T7) : Safari > Partager > « Sur l'écran d'accueil ». La page ouverte depuis l'icône affiche « webapp-ecran-accueil » en haut.
- Le bouton « Arrêter le test en cours » interrompt la voix et l'écoute.

## Informations générales

| Élément | Valeur |
|---|---|
| Date | |
| Modèle d'iPhone | |
| Version d'iOS | |
| Écouteurs | |
| Voix retenue (T1) | |

## Grille de résultats

| Test | Relevé | Résultat | Commentaire |
|---|---|---|---|
| T1 Voix | Voix fr disponibles ; `onend` sur 10 lectures | … / 10 | |
| T2 Reconnaissance au calme | Mots corrects sur 20 (manuel) | … / 20 (… %) | |
| T3-A Fenêtre 60 s, continu | `end` spontanés ; intermédiaires ; `isFinal` | | |
| T3-B Fenêtre 60 s, relances | Relances ; trou moyen entre deux écoutes (ms) | | |
| **T4 Chaîne sans les mains** | Cycles où le micro s'ouvre sans toucher | … / 10 | |
| T5 Alternatives | Nombre d'alternatives renvoyées | | |
| T6 Majuscules | Noms propres capitalisés ? | oui / non | |
| T7 Onglet Safari | Écran resté allumé 30 min ? Batterie consommée | oui / non · … % | |
| T7 Web app | Écran resté allumé 30 min ? Batterie consommée | oui / non · … % | |
| T8 Écouteurs | Micro des écouteurs utilisé ? Volume de la voix après écoute ? Tige reçue ? | | |
| **T9 Terrain** | Mots corrects sur 20, en marchant, en forêt | … / 20 (… %) | Conditions : |
| T10 Hors réseau | Erreur renvoyée en mode avion ; en zone blanche | | Zones sans 4G : |

## Lecture des résultats

- **T3** : si la variante A donne des `end` spontanés avant 60 s, le mode continu n'est pas fiable. Si des segments sont « promus », iOS n'envoie pas toujours `isFinal`. Le trou moyen de B indique ce qui est perdu à chaque relance.
- **T4** : un cycle est réussi quand le micro s'ouvre sans erreur `not-allowed` / `service-not-allowed`, même si rien n'a été dit. Le nombre de cycles avec transcription est relevé à part.
- **T7** : « plus gros trou » supérieur à 5 000 ms = la page a été suspendue (écran éteint ou application en arrière-plan) pendant le test.
- Le taux T2 / T9 est calculé automatiquement (mot attendu présent dans l'une des alternatives, accents ignorés). Un toucher sur ✓ / ✗ corrige un verdict erroné ; la correction est notée dans le JSON.

## Décision

Règle (PLAN.md, phase 0) : **GO web si T4 = 10/10 et T9 ≥ 80 %**. Sinon, plan B (PLAN.md, section 9).

| Critère | Seuil | Mesuré | Atteint |
|---|---|---|---|
| T4 | 10 / 10 | | |
| T9 | ≥ 80 % | | |

Décision : GO / NO-GO

Réglages à reporter dans `config.js` en phase 1 :

| Paramètre | Valeur retenue | Justification (test) |
|---|---|---|
| `continuous` | | T3 |
| `interimResults` | | T3 |
| `maxAlternatives` | | T5 |
| Détection des noms propres par majuscule | | T6 |
| Media Session (tige) | | T8 |

## Observations libres

## JSON : onglet Safari

```json

```

## JSON : web app écran d'accueil

```json

```

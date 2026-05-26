# Fiche — Command Injection

## Définition

La Command Injection permet à un attaquant d'exécuter des commandes système
arbitraires sur le serveur, en injectant des commandes dans un paramètre
transmis à une fonction d'exécution système.

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | [À compléter — DVWA / exercice Jedha / autre] |
| Niveau | Débutant / intermédiaire |
| Objectif | Identifier un point d'injection de commande, comprendre l'impact, proposer la remédiation |
| Environnement autorisé | Oui |

## Mécanisme

L'application utilise une fonction d'exécution système (`shell_exec`, `system`, `os.system`)
en concaténant directement une entrée utilisateur.
L'attaquant peut enchaîner des commandes supplémentaires via des séparateurs système
(`;`, `&&`, `||`, `|`).

## Impact potentiel

| Impact | Description |
|---|---|
| Exécution de commandes | Commandes arbitraires sur le serveur |
| Lecture de fichiers | Accès à des fichiers sensibles |
| Backdoor | Création d'un accès persistant |
| Exfiltration | Envoi de données vers un serveur tiers |
| Compromission | Prise de contrôle complète du système |

## Méthode de détection

- Identifier les paramètres transmis à des fonctions système
- Tester des séparateurs de commandes : `;`, `&&`, `||`, `|`
- Observer si la sortie contient des résultats de commande système

## Résultat observé en lab

[À compléter après exercice]

## Remédiation

| Mesure | Description |
|---|---|
| Éviter les appels shell | Ne jamais appeler le shell avec des entrées utilisateur |
| APIs natives | Utiliser des bibliothèques sans appel shell |
| Liste blanche | Valider strictement l'entrée si l'appel shell est inévitable |
| Échappement | `escapeshellarg()` (PHP) ou `shlex.quote()` (Python) |
| Moindre privilège | Le processus serveur ne doit pas être root |
| Isolation | Container / sandbox pour limiter l'impact |

## Ce que j'ai appris

[À compléter après exercice]

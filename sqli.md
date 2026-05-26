# Fiche — SQL Injection (SQLi)

## Définition

La SQL Injection permet à un attaquant d'injecter des instructions SQL malveillantes
dans une requête construite à partir d'entrées utilisateur non filtrées,
modifiant ainsi la logique de la base de données.

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | [À compléter — DVWA / exercice Jedha / autre] |
| Niveau | Débutant / intermédiaire |
| Objectif | Identifier une injection SQL, comprendre son impact, proposer la remédiation |
| Environnement autorisé | Oui |

## Mécanisme

L'application construit une requête SQL en concaténant directement une entrée
utilisateur sans validation ni paramétrage. L'attaquant peut modifier la logique
de la requête pour extraire des données, contourner l'authentification
ou exécuter des commandes destructives.

## Impact potentiel

| Impact | Description |
|---|---|
| Extraction de données | Identifiants, mots de passe, données clients |
| Contournement d'auth | Connexion sans mot de passe valide |
| Modification de données | UPDATE / DELETE non autorisés |
| Exécution de commandes | Via xp_cmdshell (MSSQL) ou LOAD_FILE (MySQL) |
| Compromission de la BDD | Destruction ou exfiltration complète |

## Méthode de détection

- Tester les paramètres avec des caractères spéciaux : `'`, `"`, `;`
- Observer les messages d'erreur (peuvent révéler le SGBD)
- Tester des conditions booléennes
- Utiliser Burp Suite ou observer les différences de réponse

## Résultat observé en lab

[À compléter après exercice]

## Remédiation

| Mesure | Description |
|---|---|
| Requêtes préparées | Séparer le code SQL des données (`SELECT * FROM t WHERE id = ?`) |
| ORM | Utiliser un ORM qui paramétrise automatiquement |
| Validation des entrées | Vérifier type et format attendu |
| Moindre privilège BDD | Le compte applicatif ne doit pas avoir tous les droits |
| Messages d'erreur génériques | Ne pas exposer les détails du SGBD en production |
| WAF | Filtrage des patterns SQL connus |

## Ce que j'ai appris

[À compléter après exercice]

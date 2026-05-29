# Fiche — SQL Injection (SQLi)

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | DVWA (Damn Vulnerable Web Application) — exercice Jedha |
| Environnement | localhost — VM lab autorisée |
| Outil d'analyse | Burp Suite (interception requêtes, inspection paramètres) |
| Rencontré aussi | Lab EvilCorp (pentest encadré, mai 2026) — interface d'administration |
| Environnement autorisé | Oui |

---

## Ce qu'est une injection SQL

Une application construit des requêtes SQL en concaténant directement ce que l'utilisateur a saisi.
Si ce contenu n'est pas filtré, l'attaquant peut modifier la logique de la requête
— non plus "cherche l'utilisateur n°1" mais "renvoie-moi toute la table des mots de passe".

Le problème central : le code SQL et les données utilisateur sont mélangés.
Séparer les deux (requêtes préparées) suffit à éliminer la vulnérabilité dans la majorité des cas.

---

## Comment ça s'est passé concrètement

La requête vulnérable de DVWA c'est du classique : `SELECT first_name, last_name FROM users WHERE user_id = '$id'`.
Une apostrophe dans le champ, et j'ai eu une erreur MySQL brute dans la réponse — c'est tout ce qu'il faut pour confirmer que l'entrée est collée directement dans la requête sans filtrage.

Dans Burp le paramètre `id` était visible en clair dans la requête GET. J'ai intercepté, modifié, renvoyé.
Trouver le bon nombre de colonnes avec `ORDER BY`, puis construire un `UNION SELECT` qui matche la structure.
Burp Repeater rend ça rapide — un clic pour renvoyer au lieu de recharger le formulaire à la main à chaque essai.

Résultat : noms et hashs MD5 de la table users. MD5 sans sel — cassables en quelques minutes avec hashcat.

![Burp Suite — DVWA SQL Injection, interception et inspection du paramètre id](assets/sqli-burp-dvwa.png)

---

## Dans le contexte du pentest lab (EvilCorp, mai 2026)

La vulnérabilité VULN-04 du rapport de pentest correspond à une injection SQL sur une interface d'administration.
Sévérité CVSS 3.1 : **9.1 Critical** (CWE-89).

La chaîne d'attaque documentée confirme que cette injection constituait l'un des vecteurs d'accès initiaux,
en parallèle de l'injection de commande et de la XSS stockée.

*(Voir VULN-04 dans le rapport EvilCorp pour le contexte complet — CVSS 9.1 Critical.)*

---

## Remédiation

| Mesure | Pourquoi |
|---|---|
| Requêtes préparées | `SELECT * FROM users WHERE id = ?` — le paramètre n'est jamais interprété comme du SQL |
| ORM | Paramétrise automatiquement, réduit la surface de risque |
| Validation des entrées | Si le champ attend un entier, rejeter tout ce qui n'est pas un entier |
| Moindre privilège sur le compte BDD | Le compte applicatif ne doit pas avoir `DROP`, `xp_cmdshell` ni les droits admin |
| Erreurs génériques en production | Ne pas exposer le message d'erreur du SGBD — il facilite la reconnaissance |
| WAF | Couche de détection complémentaire, mais pas suffisante seule |

---

## Ce que j'ai appris

- La syntaxe d'une injection UNION et pourquoi le nombre de colonnes doit correspondre à la requête d'origine
- Pourquoi les messages d'erreur du SGBD sont précieux pour l'attaquant — et doivent être masqués en prod
- La différence entre injection aveugle (boolean-based, time-based) et injection avec retour visible
- Que la même vulnérabilité CWE-89 peut se noter 9.1 sur une interface admin (impact élevé) ou moins selon le contexte

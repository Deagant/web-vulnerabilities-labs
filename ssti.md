# Fiche — Server-Side Template Injection (SSTI)

## Définition

La Server-Side Template Injection survient quand une application insère
des données contrôlées par l'utilisateur directement dans un moteur de templates
(Jinja2, Twig, Freemarker…), permettant l'exécution de code côté serveur.

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | Lab web autorisé — exercice Jedha |
| Niveau | Intermédiaire |
| Objectif | Identifier un point d'injection de template, comprendre le mécanisme, proposer la remédiation |
| Environnement autorisé | Oui |

## Mécanisme

Un moteur de template interprète des expressions spéciales (ex : `{{ }}` pour Jinja2).
Si une entrée utilisateur est intégrée dans un template sans sanitisation préalable,
l'attaquant peut injecter des expressions évaluées par le serveur.

**Différence clé avec XSS :** la SSTI s'exécute côté *serveur*, pas côté client.
Elle peut mener à la lecture de fichiers, à l'exécution de commandes système
ou à la compromission complète du serveur.

## Impact potentiel

| Impact | Description |
|---|---|
| Lecture de fichiers | Accès à des fichiers sensibles du système |
| Exécution de commandes (RCE) | Exécution de commandes système arbitraires |
| Exfiltration de secrets | Variables d'environnement, clés API, mots de passe |
| Compromission du serveur | Prise de contrôle complète |
| Mouvement latéral | Accès à d'autres systèmes du réseau interne |

## Méthode de détection

- Identifier les paramètres reflétés dans la réponse
- Tester des expressions mathématiques dans la syntaxe du moteur (`{{7*7}}`)
- Observer si le résultat est calculé par le serveur
- Identifier le moteur de template via les messages d'erreur ou le comportement

## Résultat observé en lab

L'analyse a confirmé que l'application évaluait des expressions injectées
dans un paramètre de rendu, permettant l'exécution de code côté serveur.

Le contexte était un moteur de template Python.
Le résultat a validé l'interprétation des expressions et l'accès à des objets internes du framework.

*(Aucun payload d'exploitation publié — mécanisme documenté à titre pédagogique.)*

## Remédiation

| Mesure | Description |
|---|---|
| Séparer code et données | Ne jamais concaténer d'entrée utilisateur dans un template |
| Variables séparées | `render_template("page.html", name=user_input)` — pas `render_template_string(user_input)` |
| Validation des entrées | Rejeter les caractères spéciaux inutiles au contexte |
| Sandbox moteur | Activer `SandboxedEnvironment` (Jinja2) si disponible |
| Moindre privilège | Le processus serveur ne doit pas tourner en root |
| WAF | Filtrage des expressions de template au niveau applicatif |

## Ce que j'ai appris

- La différence fondamentale entre SSTI (serveur) et XSS (client)
- L'importance de ne jamais rendre un template depuis une entrée utilisateur
- Comment identifier le moteur de template cible
- Pourquoi la SSTI peut mener à une compromission complète du serveur

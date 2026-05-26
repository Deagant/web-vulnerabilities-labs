# Fiche — Cross-Site Scripting réfléchi (XSS Reflected)

## Définition

Le Cross-Site Scripting réfléchi (XSS non-persistant) est une vulnérabilité
qui permet d'injecter du code JavaScript dans une page web via un paramètre
non filtré, renvoyé directement dans la réponse HTTP sans validation ni échappement.

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | Lab web autorisé — exercice Jedha |
| Niveau | Débutant / intermédiaire |
| Objectif | Identifier un point d'injection, comprendre le mécanisme, proposer la remédiation |
| Environnement autorisé | Oui |

## Mécanisme

L'application accepte une valeur en paramètre (URL, formulaire) et la réinsère
directement dans la page HTML sans filtrage ni échappement.
Si un attaquant soumet du code JavaScript à la place d'une valeur légitime,
le navigateur de la victime l'exécutera comme du code valide.

**Vecteur typique :** un lien piégé envoyé à une victime déclenche l'exécution
du script dans son propre navigateur, à son insu.

## Impact potentiel

| Impact | Description |
|---|---|
| Vol de session | Récupération du cookie de session si `HttpOnly` absent |
| Redirection | Envoi vers un site malveillant |
| Actions forcées | Exécution d'opérations à la place de l'utilisateur |
| Phishing | Affichage d'un faux formulaire de connexion |
| Keylogging | Capture de frappes clavier |

## Méthode de détection

- Identifier les paramètres reflétés dans la réponse HTTP
- Tester l'injection de caractères spéciaux : `<`, `>`, `"`, `'`
- Observer si la réponse contient les caractères non échappés
- Vérifier avec Burp Suite (onglet Repeater) ou l'inspecteur du navigateur

## Résultat observé en lab

L'analyse a confirmé qu'un paramètre de la page était inséré dans le DOM
sans échappement, permettant l'injection de code arbitraire interprété
par le navigateur lors du chargement de la page.

*(Aucun payload offensif publié — mécanisme documenté à titre pédagogique.)*

## Remédiation

| Mesure | Description |
|---|---|
| Échappement contextuel | Encoder `<` → `&lt;`, `>` → `&gt;` avant insertion dans le HTML |
| Validation des entrées | Rejeter les entrées ne correspondant pas au format attendu |
| Content Security Policy | Header HTTP limitant les scripts autorisés à s'exécuter |
| HttpOnly sur les cookies | Empêche l'accès aux cookies de session via JavaScript |
| Frameworks sécurisés | Moteurs de template qui échappent automatiquement |

## Ce que j'ai appris

- La différence entre XSS réfléchi, stocké et DOM-based
- L'importance du contexte d'injection (HTML, JavaScript, attribut, URL)
- Pourquoi l'échappement doit être adapté au contexte
- Le rôle de CSP comme couche de défense complémentaire

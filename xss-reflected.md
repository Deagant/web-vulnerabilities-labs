# Fiche — Cross-Site Scripting réfléchi (XSS Reflected)

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | DVWA (Damn Vulnerable Web Application) — exercice Jedha |
| Environnement | localhost — VM lab autorisée |
| Niveau de sécurité DVWA | Medium |
| Outil d'analyse | Burp Suite (interception + modification de requêtes) |
| Environnement autorisé | Oui |

---

## Ce qu'est le XSS réfléchi

Une application affiche directement dans la page ce que l'utilisateur a saisi — un prénom, un terme de recherche, n'importe quel champ de formulaire.
Si cette valeur n'est pas nettoyée avant insertion dans le HTML, un attaquant peut soumettre du JavaScript à la place d'un texte normal.
Le serveur renvoie ce code dans la réponse, et le navigateur de la victime l'exécute.

"Réfléchi" signifie que le payload n'est pas stocké : il voyage dans l'URL ou dans un champ POST, et ne touche que le navigateur de la personne qui clique sur le lien piégé.

---

## Mécanisme technique

Le paramètre `name` de DVWA est inséré dans la réponse HTML sous la forme :

```
Hello [valeur saisie]
```

Sans échappement, une valeur comme `<script>alert(1)</script>` devient du HTML valide.
Le navigateur la parse, identifie une balise script, et l'exécute.

---

## Ce que j'ai fait en lab

### Étape 1 — Identifier le filtre

Le niveau Medium de DVWA applique un filtre sur la chaîne `<script>` (en minuscules uniquement).
En soumettant `<script>test</script>` dans le champ, rien ne se passe.
En regardant la source de la réponse dans Burp, la balise a été retirée.

### Étape 2 — Contourner le filtre par variation de casse

Le filtre est sensible à la casse : il bloque `<script>` mais pas `<Script>`.

Premier payload testé via Burp Suite (onglet Repeater, paramètre `name`) :

```
<Script>document.body.style.background='red'</Script>
```

Résultat : la page s'affiche avec un fond rouge. L'injection est exécutée.

### Étape 3 — Démontrer l'impact : vol de cookie de session

Second payload, directement dans l'URL :

```
localhost/vulnerabilities/xss_r/?name=<Script>alert(document.cookie)</Script>
```

Résultat : une boîte de dialogue apparaît avec le contenu du cookie de session.
La valeur inclut `security=medium` et l'identifiant de session PHP — ce qui confirme qu'un attaquant pourrait récupérer ce token et usurper la session.

*(Le cookie de session du lab n'est pas reproduit ici — il n'a aucune valeur hors de la VM.)*

---

## Pourquoi ça marche au niveau Medium

DVWA Medium utilise `str_replace('<script>', '', $name)` — un remplacement simple, sensible à la casse.
`<Script>` passe donc sans problème. C'est un exemple classique de filtre insuffisant : une liste noire partielle est contournable par variation syntaxique.

---

## Impact potentiel

| Impact | Description |
|---|---|
| Vol de session | Récupération du cookie `PHPSESSID` si `HttpOnly` absent — l'attaquant peut rejouer la session |
| Redirection silencieuse | `window.location` vers un faux site de login |
| Keylogging | Capture des frappes clavier de la victime |
| Actions forcées | Soumettre des formulaires, modifier des paramètres à la place de l'utilisateur |
| Phishing in-page | Injecter un faux formulaire dans la page légitime |

---

## Remédiation

| Mesure | Pourquoi |
|---|---|
| Échappement contextuel (HTML) | `<` → `&lt;`, `>` → `&gt;` — le navigateur affiche le texte, n'exécute pas le code |
| Validation côté serveur | Rejeter les caractères `<`, `>`, `"`, `'` si le champ n'en a pas besoin |
| Content Security Policy (CSP) | Header HTTP qui liste les sources de script autorisées — bloque l'exécution de scripts inline |
| `HttpOnly` sur les cookies de session | Empêche JavaScript d'accéder à `document.cookie` |
| Frameworks modernes | React, Vue et Angular échappent automatiquement les variables dans les templates |

---

## Ce que j'ai appris

- Un filtre par liste noire partielle est insuffisant — la casse, l'encodage et les variantes syntaxiques permettent de le contourner
- L'absence du flag `HttpOnly` sur un cookie de session transforme un XSS en vol de session direct
- Burp Suite Repeater permet de tester des variantes de payload rapidement sans recharger manuellement le navigateur
- La différence concrète entre XSS réfléchi (payload dans l'URL), stocké (payload en base) et DOM-based (payload traité par le JS côté client)

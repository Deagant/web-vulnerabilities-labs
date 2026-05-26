# Web Vulnerabilities Labs

Notes techniques issues de labs web autorisés.  
Objectif : comprendre les vulnérabilités applicatives courantes, leur impact et les remédiations associées.

---

## Contexte

| Élément | Détail |
|---|---|
| Type | Exercices de formation |
| Environnement | DVWA (localhost) et application Jedha dédiée |
| Périmètre | Autorisé et contrôlé |
| Outils | Burp Suite, navigateur |
| Données sensibles | Supprimées ou anonymisées |

---

## Vulnérabilités documentées

| Fiche | Vulnérabilité | Environnement | Statut |
|---|---|---|---|
| `xss-reflected.md` | Cross-Site Scripting réfléchi | DVWA, niveau Medium | ✅ Documenté |
| `ssti.md` | Server-Side Template Injection | App Flask/Jinja2 Jedha | ✅ Documenté |
| `sqli.md` | SQL Injection | DVWA + pentest EvilCorp | ✅ Documenté |
| `command-injection.md` | Command Injection | Pentest EvilCorp (CVSS 6.5) | ✅ Documenté |
| `mitigations.md` | Synthèse des remédiations | — | ✅ Documenté |

---

## Approche

Chaque fiche contient :

1. **Définition** — explication courte de la vulnérabilité
2. **Mécanisme** — pourquoi la faille existe
3. **Impact potentiel** — conséquences possibles
4. **Méthode de détection** — comment l'identifier
5. **Résultat observé en lab** — sans données sensibles
6. **Remédiation** — comment corriger
7. **Ce que j'ai appris** — retour réflexif

---

## Compétences démontrées

- Compréhension des vulnérabilités web courantes (OWASP Top 10)
- Analyse du mécanisme d'exploitation
- Évaluation de l'impact
- Documentation orientée remédiation
- Capacité à expliquer techniquement une faille sans publier de payload offensif

---

## Avertissement

Exercices réalisés dans un cadre légal, éducatif et autorisé.  
Aucune donnée réelle ou sensible n'est publiée dans ce dépôt.  
Les fiches décrivent les mécanismes — elles ne constituent pas un guide d'attaque.

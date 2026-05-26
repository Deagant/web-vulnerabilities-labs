# Synthèse des remédiations — Web Vulnerabilities

---

## Principes généraux

| # | Principe | Description |
|---|---|---|
| 1 | Valider toutes les entrées | Côté serveur, pas seulement côté client |
| 2 | Échapper toutes les sorties | Selon le contexte : HTML, JS, SQL, shell |
| 3 | Moindre privilège | Compte applicatif avec droits minimaux |
| 4 | Défense en profondeur | Combiner plusieurs couches de protection |
| 5 | Erreurs génériques | Ne pas exposer les détails techniques en production |

---

## Par vulnérabilité

| Vulnérabilité | Défense principale | Défenses complémentaires |
|---|---|---|
| XSS réfléchi | Échappement contextuel | CSP, HttpOnly, SameSite cookies |
| SSTI | Séparer code et données du template | Sandbox, moindre privilège |
| SQLi | Requêtes préparées / ORM | Moindre privilège BDD, WAF |
| Command Injection | Éviter les appels shell | Liste blanche, shlex.quote, isolation |

---

## Headers de sécurité HTTP recommandés

```http
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=31536000; includeSubDomains
Permissions-Policy: geolocation=(), camera=()
```

---

## Références

- OWASP Top 10 : https://owasp.org/Top10/
- OWASP Testing Guide : https://owasp.org/www-project-web-security-testing-guide/
- CWE Top 25 : https://cwe.mitre.org/top25/
- PortSwigger Web Security Academy : https://portswigger.net/web-security

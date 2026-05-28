# Fiche — Command Injection

## Contexte du lab

| Élément | Détail |
|---|---|
| Plateforme | Lab EvilCorp — pentest encadré, mai 2026 |
| Équipe | 4 participants |
| Phase | Exploitation web (avant mouvement latéral) |
| Sévérité (CVSS 3.1) | 6.5 Medium — CWE-78 |
| Environnement autorisé | Oui |

---

## Ce qu'est une injection de commande

Une application passe une valeur fournie par l'utilisateur à une fonction d'exécution système
(`shell_exec`, `system`, `os.system`, `subprocess`…) sans la valider au préalable.

L'attaquant peut alors enchaîner ses propres commandes en utilisant les séparateurs shell
standard (`;`, `&&`, `||`, `|`).

La différence avec la SSTI ou la SQLi : l'exécution ne passe pas par un interpréteur applicatif
(template engine, SGBD) mais directement par le shell système. L'impact immédiat
est l'exécution de commandes OS avec les droits du processus serveur.

---

## Mécanisme technique

L'application exécutait en interne une commande construite à partir d'une valeur saisie dans un champ
censé recevoir une valeur réseau simple — typiquement une adresse IP pour un diagnostic (ping, traceroute).

Côté serveur, la logique ressemblait à :

```python
# Vulnérable
os.system("ping -c 1 " + user_input)
```

En soumettant une valeur légitime suivie d'un séparateur de commande shell :

```
127.0.0.1; id
```

Le serveur exécutait d'abord le ping, puis la commande `id` — et renvoyait les deux résultats.

---

## Ce que j'ai observé en lab

Le champ acceptait une valeur réseau simple.
En ajoutant un séparateur shell suivi d'une commande de validation (`id`, `whoami`),
la réponse renvoyait l'identité du processus serveur.

Ce retour a confirmé :

1. **L'injection était possible** — la valeur n'était pas filtrée avant exécution
2. **L'exécution était côté serveur** — pas du JavaScript côté navigateur
3. **Le compte applicatif** — son identité était visible, permettant d'évaluer le niveau de privilèges disponibles

*(Détails de l'environnement anonymisés — voir le [rapport de pentest](https://github.com/Remy-Miquel/pentest-lab-report-portfolio) pour le contexte complet.)*

---

## Impact potentiel

| Impact | Description |
|---|---|
| Énumération système | `id`, `whoami`, `uname -a`, liste des fichiers accessibles |
| Lecture de fichiers | Accès aux fichiers lisibles par le compte applicatif (`/etc/passwd`, configs, clés) |
| Reverse shell | Ouverture d'une connexion sortante vers un point d'écoute attaquant |
| Mouvement latéral | Accès aux ressources réseau internes accessibles depuis le serveur |
| Persistance | Création d'une backdoor, modification de fichiers de démarrage |

Dans le contexte du pentest EvilCorp, cette vulnérabilité constituait un point d'entrée
qui, combiné à des droits de groupe excessifs, permettait un mouvement latéral vers d'autres comptes.

---

## Remédiation

| Mesure | Pourquoi |
|---|---|
| Supprimer les appels shell sur entrée utilisateur | La solution la plus sûre — utiliser les APIs natives du langage à la place |
| Validation stricte par liste blanche | Si l'entrée doit être une IP, rejeter tout ce qui ne correspond pas à `^\d{1,3}(\.\d{1,3}){3}$` |
| `shlex.quote()` (Python) ou `escapeshellarg()` (PHP) | Échapper les métacaractères si l'appel shell est inévitable |
| Moindre privilège processus | Le serveur web ne doit pas tourner en root — limite l'impact d'une injection réussie |
| Isolation | Conteneur ou sandbox — restreint la portée des commandes exécutables |
| WAF | Filtrage des séparateurs shell (`;`, `&&`, pipe) — couche complémentaire, pas suffisante seule |

---

## Ce que j'ai appris

- Un champ de diagnostic réseau (ping, traceroute) est un vecteur classique de command injection — l'usage légitime appelle le shell, l'usage malveillant en profite
- La validation par liste noire (bloquer `;`, `&`…) est insuffisante : les encodages URL, les espaces remplacés par `$IFS`, et d'autres techniques permettent de la contourner
- La différence pratique entre injection de commande et SSTI : même résultat (RCE), chemin différent (shell OS vs moteur de template)
- Pourquoi le principe du moindre privilège sur le compte applicatif réduit concrètement l'impact — un compte non-root ne peut pas lire `/etc/shadow` ni modifier les fichiers système

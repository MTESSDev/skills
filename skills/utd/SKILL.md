---
name: utd
description: Système de design UTD (utd-webcomponents, MESS, basé sur Québec.ca). À utiliser pour écrire ou modifier une page, un gabarit, un formulaire ou un composant d'interface qui utilise les balises utd-* (utd-champ-form, utd-liste-deroulante, utd-dialog, utd-piv-entete…), les classes UTD (utd-btn, utd-table, utd-row, utd-col-*, utd-icone-ms…) ou l'API JS window.utd (message, dialogue, notification, traitementEnCours, filAriane), en HTML, Vue, Svelte ou .NET/Razor. Aussi pour relire ou auditer une page qui les utilise. Prime sur toute skill CSS ou UI générique (Bootstrap, Tailwind, bibliothèques de composants) dès que la page est destinée au gouvernement du Québec. Ne couvre ni le backend, ni le routage, ni les essais automatisés, ni les conventions de code d'une équipe.
---

# UTD — composants web du MESS (plusieurs versions)

Ce skill documente chaque version mineure d'UTD dans son propre dossier. Balises, attributs, slots et classes changent d'une version à l'autre : n'employer que le dossier qui correspond au projet.

## 1. Trouver la version d'UTD du projet

Chercher dans cet ordre et retenir la première trouvée :

1. `node_modules/utd-webcomponents/package.json` ou `package-lock.json` (version installée exacte), sinon la dépendance `utd-webcomponents` de `package.json`. Pour une plage (`^5.1.0`, `~5.1.0`), à défaut de version installée, prendre la borne basse.
2. Adresse CDN `utd-webcomponents@x.y.z` dans les gabarits (`.html`, `.cshtml`, `.vue`, `.svelte`…).
3. Paramètre `?v=x.y.z` sur le `<link>` du CSS ou le `<script>` du JS UTD.
4. `libman.json` (projets .NET) : bibliothèque `utd-webcomponents@x.y.z`.

## 2. Choisir le dossier

| Dossier | Version documentée |
|---|---|
| [`versions/5.0/`](versions/5.0/index.md) | 5.0.0 |

- Même majeure et même mineure (`5.1.3` → `5.1`) : ce dossier.
- Mineure absente : la mineure la plus proche **en dessous**, dans la même majeure (`5.3.0` sans dossier `5.3` → `5.2`). Jamais une mineure au-dessus : elle peut décrire des balises ou des attributs que le projet n'a pas.
- Version plus ancienne que tous les dossiers de sa majeure, ou majeure absente : prévenir l'utilisateur qu'aucune documentation ne correspond et lui demander quoi faire avant d'écrire du code UTD.
- Version introuvable : la demander à l'utilisateur. Projet neuf qui n'emploie pas encore UTD : version la plus récente, 5.0.0.

Annoncer la version retenue en une ligne (ex. « UTD 5.1.3, lue dans package-lock.json → versions/5.1 »).

## 3. Lire la documentation de cette version

Lire `versions/<dossier>/index.md` et suivre ses règles ; ses liens sont relatifs à ce dossier. Dans une même tâche, ne jamais lire ni citer un autre dossier de `versions/`.

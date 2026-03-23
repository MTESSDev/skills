---
name: utd-webcomponents
description: >
  Générer du HTML valide utilisant les composants web UTD (utd-webcomponents) du MESS/MTESSDev.
  Déclencher dès que la conversation implique des balises utd-*, la librairie utd-webcomponents,
  les gabarits PIV gouvernementaux du Québec, ou toute interface web MESS utilisant ce système de
  design. Inclut les règles d'interfaces gouvernementales, la structure de page requise,
  tous les composants disponibles et leur API complète.
compatibility: "Tout projet web — HTML vanilla, Vue 3, Blazor, Razor Pages, Angular"
---

# UTD Web Components — SKILL

## Objectif

Produire du HTML correct, accessible et conforme au système de design gouvernemental du Québec
en utilisant les composants web UTD du MESS.

**Lire les fichiers de référence** avant de générer du code pour un composant spécifique :

| Besoin | Fichier de référence |
|--------|----------------------|
| Gabarits de page (1 col, 2 col, pleine largeur), PIV entête/pied | `references/gabarits.md` |
| Formulaires : champs, listes déroulantes, cases à cocher, boutons radio, commutateur | `references/composants-formulaire.md` |
| Navigation : menus vertical/horizontal, onglets, accordéon, section, fil d'Ariane | `references/composants-navigation.md` |
| Feedback : avis, message, notification, dialogue modal, alerte générale | `references/composants-feedback.md` |
| API JavaScript : utd.message, utd.notification, utd.dialogue, utd.ajusterAccessibiliteLiens | `references/api-javascript.md` |

---

## Règles universelles

### 1 — Intégration des fichiers (à charger dans chaque page)

**Option A — CDN (utilisateurs avec accès Internet)**
```html
<!-- Dans <head> — précharger les fonts -->
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/open-sans-v18-latin-regular.woff2" type="font/woff2" crossorigin="anonymous">
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/open-sans-v18-latin-600.woff2" type="font/woff2" crossorigin="anonymous">
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/open-sans-v18-latin-700.woff2" type="font/woff2" crossorigin="anonymous">
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/roboto-v20-latin-regular.woff2" type="font/woff2" crossorigin="anonymous">
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/roboto-v20-latin-500.woff2" type="font/woff2" crossorigin="anonymous">
<link rel="preload" as="font" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/fonts/roboto-v20-latin-700.woff2" type="font/woff2" crossorigin="anonymous">

<!-- CSS UTD — AVANT votre propre CSS -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/css/utd-webcomponents.min.css">
<!-- Votre CSS après -->
<link rel="stylesheet" href="/css/monSite.css">

<!-- JS — FIN du <body> -->
<script src="https://cdn.jsdelivr.net/gh/MTESSDev/utd-webcomponents@4.1.0/dist/js/utd-webcomponents.min.js"></script>
```

**Option B — Copie locale (environnements sans accès Internet, recommandé pour intranet/IIS)**
```html
<!-- Même structure mais en chemin relatif, avec query string de version pour cache busting -->
<link rel="preload" as="font" href="/fonts/open-sans-v18-latin-regular.woff2" type="font/woff2" crossorigin="anonymous">
<!-- ... autres fonts ... -->
<link rel="stylesheet" href="/css/utd-webcomponents.min.css?v=4.1.0">
<link rel="stylesheet" href="/css/monSite.css">
<!-- fin du body -->
<script src="/js/utd-webcomponents.min.js?v=4.1.0"></script>
```

> **Note déploiement IIS** : La structure du dossier `dist` doit être respectée. Télécharger
> depuis [les releases GitHub](https://github.com/MTESSDev/utd-webcomponents/releases).

### 2 — Structure HTML obligatoire

- Toujours `lang="fr"` sur `<html>`.
- `<div class="utd-conteneur-principal">` encadre tout le contenu (header + main) — garantit le pied de page en bas.
- `<main id="main">` est requis — le lien "Passer au contenu" du PIV pointe vers `#main`.
- `utd-hautpage` se place juste avant le `<footer>`.
- `<footer class="utd">` contient `utd-pied-page-site` et `utd-piv-pied-page`.

### 3 — Classes CSS utilitaires courantes

| Classe | Effet |
|--------|-------|
| `utd-container` | Conteneur centré avec marges responsives |
| `utd-bandeau-principal` | Bandeau de navigation horizontal (wrap du menu horizontal) |
| `utd-zone-raccourcis-connexion` | Zone icônes + bouton connexion à droite du menu |
| `utd-zone-raccourcis` | Sous-zone icônes |
| `utd-zone-connexion` | Sous-zone bouton connexion |
| `utd-sr-only` | Visible lecteur écran seulement |
| `utd-d-none` | `display: none` |
| `utd-emphase-gris` | Surlignage code inline gris |
| `utd-icone-svg [nom]` | Icône SVG sprite (ex. `imprimante`, `poubelle`, `crayon`, `pdf`, `enregistrer`, `crochet`, `utilisateur-blanc`, `enveloppe-blanc`) |
| `utd-largeur-max-zone-texte` | Limite la largeur du texte pour lisibilité |
| `mb-N` / `mt-N` / `mr-N` | Marges en px (ex. `mb-32`, `mt-48`) |
| `utd-text-sm` | Texte petit |

### 4 — Grille (Bootstrap 5 préfixé `utd-`)

La grille fonctionne exactement comme Bootstrap 5 mais toutes les classes sont préfixées `utd-` :
- `utd-row`, `utd-col`, `utd-col-auto`
- `utd-row-cols-1`, `utd-row-cols-md-2`, `utd-row-cols-lg-3`
- `utd-groupe-champs-combines` — pour aligner des champs sur une même ligne (ex. téléphone + poste, date début + date fin)
- `utd-groupe-champs` — groupe de champs avec titre (utiliser `role="group"` + `aria-labelledby`)

> ⚠️ Ne pas importer Bootstrap — les composants UTD l'incluent déjà.

### 5 — Liens externes

Appeler `utd.ajusterAccessibiliteLiens()` après le chargement de la page (et après chaque injection de contenu dynamique) pour ajouter automatiquement l'icône lien externe et les attributs d'accessibilité sur tous les `<a target="_blank">`.

### 6 — Règles d'interfaces gouvernementales (obligatoires)

Tout site MESS doit respecter :
- **PIV** (Programme d'identification visuelle) : bandeau d'entête + pied de page gouvernemental via `utd-piv-entete` et `utd-piv-pied-page`. Voir `references/gabarits.md`.
- **Système de design gouvernemental** : utiliser obligatoirement les composants UTD. Toute nouvelle composante non disponible doit être développée avec l'équipe UX/DCNG.
- **Accessibilité SGQRI 008 2.0** (basé sur WCAG 2.0) : tous les composants UTD respectent ces standards. Ne pas contourner les patterns d'accessibilité intégrés.

### 7 — Utilisation dans Vue 3 (SPA)

Dans un contexte SPA, utiliser les **slots** pour les liens (router-link) au lieu des attributs `href` simples. Voir `references/gabarits.md` pour les exemples Vue complets.

```html
<!-- Menu horizontal avec Vue router -->
<utd-menu-horizontal :path-courant="$route.path">
  <router-link to="/">Accueil</router-link>
  <utd-menu-horizontal-item libelle="Composants" href="/composants">
    <router-link to="/composants">Composants</router-link>
  </utd-menu-horizontal-item>
</utd-menu-horizontal>
```

### 8 — Validation et gestion des erreurs de formulaire

Le pattern standard pour tous les champs UTD :
```javascript
// Sur blur/change du contrôle natif, mettre à jour les attributs du composant parent utd-champ-form
controleUtd.setAttribute('invalide', estValide ? 'false' : 'true');
// Le message-erreur est défini en attribut HTML ou via setAttribute
```

Pour les avis d'erreur de formulaire (résumé d'erreurs) :
```html
<utd-avis type="erreur" titre="Des erreurs sont présentes dans le formulaire." focus="true">
  <ul>
    <li><a href="#champId">Le champ « Nom » est obligatoire.</a></li>
  </ul>
</utd-avis>
```

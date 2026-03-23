# Gabarits de page UTD — Référence

## Gabarit 1 colonne

Structure HTML complète minimale :

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Mon application – MESS</title>

  <!-- Fonts UTD -->
  <link rel="preload" as="font" href="/fonts/open-sans-v18-latin-regular.woff2" type="font/woff2" crossorigin="anonymous">
  <link rel="preload" as="font" href="/fonts/open-sans-v18-latin-600.woff2" type="font/woff2" crossorigin="anonymous">
  <link rel="preload" as="font" href="/fonts/open-sans-v18-latin-700.woff2" type="font/woff2" crossorigin="anonymous">
  <link rel="preload" as="font" href="/fonts/roboto-v20-latin-regular.woff2" type="font/woff2" crossorigin="anonymous">
  <link rel="preload" as="font" href="/fonts/roboto-v20-latin-500.woff2" type="font/woff2" crossorigin="anonymous">
  <link rel="preload" as="font" href="/fonts/roboto-v20-latin-700.woff2" type="font/woff2" crossorigin="anonymous">

  <!-- CSS UTD avant votre CSS -->
  <link rel="stylesheet" href="/css/utd-webcomponents.min.css?v=4.1.0">
  <link rel="stylesheet" href="/css/monSite.css">
</head>
<body>
  <noscript>Message si JavaScript désactivé.</noscript>

  <div class="utd-conteneur-principal">
    <header>
      <!-- PIV Entête — voir section ci-dessous pour tous les attributs -->
      <utd-piv-entete
        id="pivEntete"
        titre-site1="Nom de mon application"
        alt-logo="Signature du gouvernement du Québec. Accédez à Nom de mon application."
        url-nous-joindre="/nous-joindre"
        url-langue-alternative="/en">
      </utd-piv-entete>

      <!-- Bandeau principal avec menu horizontal -->
      <div class="utd-bandeau-principal" id="bandeauPrincipal">
        <div class="utd-container">
          <utd-menu-horizontal id="menuHorizontal" afficher-icone-accueil="true">
            <utd-menu-horizontal-item libelle="Accueil" href="/"></utd-menu-horizontal-item>
            <utd-menu-horizontal-item libelle="Formulaires" href="/formulaires"></utd-menu-horizontal-item>
            <!-- Sous-menu -->
            <utd-menu-horizontal-item libelle="Administration">
              <utd-menu-horizontal-item libelle="Utilisateurs" href="/admin/utilisateurs"></utd-menu-horizontal-item>
              <utd-menu-horizontal-item libelle="Paramètres" href="/admin/parametres"></utd-menu-horizontal-item>
            </utd-menu-horizontal-item>
          </utd-menu-horizontal>

          <!-- Zone raccourcis + connexion — retirer si non requis -->
          <div class="utd-zone-raccourcis-connexion">
            <div class="utd-zone-raccourcis">
              <a href="/communications">
                <span id="envelopeCommunications" aria-hidden="true" class="utd-icone-svg enveloppe-blanc"></span>
                <span class="utd-sr-only">Accéder à vos communications</span>
              </a>
              <a href="/profil">
                <span aria-hidden="true" class="utd-icone-svg utilisateur-blanc"></span>
                <span class="utd-sr-only">Accéder à votre profil</span>
              </a>
            </div>
            <div class="utd-zone-connexion">
              <button type="button" class="utd-btn secondaire option-1 compact" id="btnDeconnexion">Déconnexion</button>
            </div>
          </div>
        </div>
      </div>
    </header>

    <div class="utd-container">
      <main id="main">
        <h1>Titre de la page</h1>
        <!-- Votre contenu -->
      </main>
    </div>
  </div>

  <!-- Bouton retour haut de page — juste avant le footer -->
  <utd-hautpage id="hautPage"></utd-hautpage>

  <footer class="utd">
    <utd-pied-page-site id="piedPageSite">
      <div slot="contenu">
        <!-- Votre contenu de pied de page de site -->
      </div>
    </utd-pied-page-site>

    <utd-piv-pied-page id="pivPiedPage">
      <div slot="liens">
        <ul>
          <li><a href="/accessibilite">Accessibilité</a></li>
          <li><a href="/conditions">Politique et conditions d'utilisation</a></li>
        </ul>
      </div>
    </utd-piv-pied-page>
  </footer>

  <script src="/js/utd-webcomponents.min.js?v=4.1.0"></script>
  <script>
    utd.ajusterAccessibiliteLiens();
  </script>
</body>
</html>
```

---

## Gabarit 2 colonnes (menu vertical gauche + contenu droit)

```html
<!-- Même head/header que gabarit 1 colonne, puis : -->
<div class="utd-container">
  <div id="conteneur2Colonnes" class="utd contenu-principal-droite">

    <!-- Colonne gauche : menu vertical -->
    <div id="colonneGauche" class="utd">
      <utd-menu-vertical id="menuVertical" titre-visible="false" titre="Navigation" sr-titre="Menu de navigation">
        <utd-menu-vertical-item href="/" libelle="Accueil"></utd-menu-vertical-item>
        <utd-menu-vertical-item libelle="Section 1">
          <utd-menu-vertical-item libelle="Page A" href="/section1/page-a"></utd-menu-vertical-item>
          <utd-menu-vertical-item libelle="Page B" href="/section1/page-b"></utd-menu-vertical-item>
        </utd-menu-vertical-item>
      </utd-menu-vertical>
    </div>

    <!-- Colonne droite : contenu -->
    <div id="colonneDroite" class="utd">
      <main id="main">
        <h1>Titre de la page</h1>
        <!-- Votre contenu -->
      </main>
    </div>
  </div>
</div>
```

---

## utd-piv-entete — Tous les attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `titre-site1` | String **(Obligatoire)** | Texte ligne 1 du nom du site |
| `titre-site2` | String (Optionnel) | Texte ligne 2 du nom du site |
| `url-titre-site` | String (Optionnel) | URL du clic sur le titre. Défaut `/`. Mettre `""` pour désactiver le lien. |
| `alt-logo` | String (Optionnel) | Texte alt du logo. Défaut : "Signature du gouvernement du Québec." |
| `url-logo` | String (Optionnel) | URL du logo. Défaut `/`. Pour sites publics : `Quebec.ca`. |
| `logo-nouvel-onglet` | Boolean (Optionnel) | Ouvrir logo dans nouvel onglet. Défaut `false`. |
| `url-langue-alternative` | String (Optionnel) | URL lien langue alternative. Si absent : lien non affiché. |
| `texte-langue-alternative` | String (Optionnel) | Texte du lien. Défaut "Français"/"English". |
| `url-nous-joindre` | String (Optionnel) | URL lien "Nous joindre". Si absent : lien non affiché. |
| `texte-nous-joindre` | String (Optionnel) | Texte du lien. Défaut "Nous joindre". |
| `passer-contenu` | Boolean (Optionnel) | Inclure lien "Passer au contenu". Défaut `true`. |
| `url-passer-contenu` | String (Optionnel) | Défaut `#main`. |
| `afficher-recherche` | Boolean (Optionnel) | Afficher barre de recherche. Défaut `false`. |
| `placeholder-recherche` | String (Optionnel) | Placeholder du champ de recherche. |
| `type-recherche` | String (Optionnel) | `"instantane"` (résultats live) ou `"simple"` (redirection). Défaut `"instantane"`. |
| `url-contenu-recherche` | String (Optionnel) | URL JSON pour le contenu de recherche (mode instantané). |

**Slots (contexte SPA — Vue, Angular, Blazor) :**

| Slot | Description |
|------|-------------|
| `lienLogo` | Lien sur le logo (ex. `<router-link to="/">`) |
| `lienTitreSite` | Lien sur le titre du site |
| `liens` | Liens droite du PIV dans `<ul><li>` (langue alt + nous joindre) |

```html
<!-- Exemple Vue 3 SPA -->
<utd-piv-entete titre-site1="Mon app" alt-logo="Signature du gouvernement du Québec.">
  <div slot="liens">
    <ul>
      <li><router-link to="/en">English</router-link></li>
      <li><router-link to="/nous-joindre">Nous joindre</router-link></li>
    </ul>
  </div>
  <span slot="lienLogo">
    <router-link to="/">Accueil</router-link>
  </span>
  <span slot="lienTitreSite">
    <router-link to="/">Accueil</router-link>
  </span>
</utd-piv-entete>
```

---

## utd-menu-horizontal — Attributs et slots

| Attribut | Description |
|----------|-------------|
| `afficher-icone-accueil` | Afficher icône maison. Défaut `false`. |
| `path-courant` | (SPA) Chemin actuel pour surligner l'élément actif. Ex. `:path-courant="$route.path"` |

**utd-menu-horizontal-item** :
- `libelle` — Texte de l'élément
- `href` — URL (si lien direct)
- Imbrication : éléments enfants = sous-menu déroulant

**Slots (Vue/Angular)** : insérer `<router-link>` ou `<a routerLink>` à l'intérieur de l'item.

---

## utd-menu-vertical — Attributs

| Attribut | Description |
|----------|-------------|
| `titre` | Titre du menu (obligatoire pour accessibilité) |
| `titre-visible` | Afficher le titre. Défaut `true`. Mettre `false` pour SR uniquement. |
| `sr-titre` | Texte SR alternatif au titre |
| `path-courant` | (SPA) Chemin actuel pour état actif |

**utd-menu-vertical-item** :
- `libelle` — Texte de l'élément
- `href` — URL
- Imbrication : sous-menu

---

## utd-hautpage — Attributs

| Attribut | Description |
|----------|-------------|
| `title` | Texte survol. Défaut "Retour en haut de page." |
| `hauteur-minimale-scroll` | Hauteur défilement avant apparition. Défaut 555px. |

---

## utd-pied-page-site — Slot

| Slot | Description |
|------|-------------|
| `contenu` | Contenu de votre pied de page de site |

---

## utd-piv-pied-page — Slot

| Slot | Description |
|------|-------------|
| `liens` | Liens réglementaires dans `<ul><li>` (Accessibilité, Conditions, etc.) |

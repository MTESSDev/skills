# Composants de navigation UTD — Référence

## utd-accordeon

Panneau extensible/réductible. Défaut : réduit.

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `titre` | String | Titre de l'accordéon |
| `tag-titre` | String (Optionnel) | Balise HTML du titre. Défaut `h2`. |
| `reduit` | Boolean (Optionnel) | État initial. Défaut `true` (réduit). Peut être modifié par code. |
| `type` | String (Optionnel) | Icône/couleur : `information`, `avertissement`, `succes`, `erreur`, `general` (défaut) |
| `contenu` | String (Optionnel) | Texte simple (sinon slot défaut) |
| `conserver-etat-affichage` | Boolean (Optionnel) | Persister l'état en session. Nécessite un `id` unique sur le composant. Défaut `false`. |

### Slots

| Slot | Description |
|------|-------------|
| défaut | Contenu HTML de l'accordéon |
| `titre` | HTML dans l'entête (ex. ajout d'une icône) |

### Événements

- `changementEtat` : déclenché à chaque ouverture/fermeture. `e.detail.reduit` (boolean).

### Exemples

```html
<!-- Accordéon de base -->
<utd-accordeon titre="Informations importantes" tag-titre="h2">
  <p>Contenu de l'accordéon...</p>
</utd-accordeon>

<!-- Accordéon développé par défaut, avec conservation d'état -->
<utd-accordeon id="accordeonFaq" titre="Foire aux questions" reduit="false"
  conserver-etat-affichage="true">
  <p>Réponse à la question...</p>
</utd-accordeon>

<!-- Accordéon de type avertissement -->
<utd-accordeon titre="Programme d'identification visuelle (PIV)" tag-titre="h2"
  type="avertissement">
  <p>Tous les sites gouvernementaux doivent...</p>
</utd-accordeon>
```

Contrôle par code :
```javascript
const accordeon = document.getElementById('accordeonFaq');
accordeon.setAttribute('reduit', 'false'); // Développer
accordeon.setAttribute('reduit', 'true');  // Réduire

// Événement
accordeon.addEventListener('changementEtat', e => {
  console.log('Réduit ?', e.detail.reduit);
});
```

---

## utd-section

Similaire à l'accordéon mais avec bordure et styles différents. Utilisé pour
regrouper des contenus dans une page (ex. section de filtres de recherche).

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `titre` | String | Titre de la section |
| `tag-titre` | String (Optionnel) | Balise HTML du titre. Défaut `h2`. |
| `extensible` | Boolean (Optionnel) | Peut-on réduire la section ? Défaut `true`. |
| `reduit` | Boolean (Optionnel) | État initial. Défaut `true`. |
| `bordure` | Boolean (Optionnel) | Afficher la bordure. Défaut `true`. |
| `padding` | Boolean (Optionnel) | Padding intérieur. Défaut `true`. Si `false`, la bordure est aussi retirée. |
| `conserver-etat-affichage` | Boolean (Optionnel) | Persister l'état. Nécessite un `id`. |

### Slots

| Slot | Description |
|------|-------------|
| défaut | Contenu de la section |
| `titre` | HTML dans l'entête |

```html
<!-- Section non extensible (toujours développée) -->
<utd-section titre="Résultats de recherche" extensible="false">
  <!-- Tableau de résultats -->
</utd-section>

<!-- Section extensible avec formulaire -->
<utd-section id="sectionFiltres" titre="Filtres de recherche" reduit="false">
  <utd-champ-form libelle="Nom" format="xl">
    <input type="text">
  </utd-champ-form>
</utd-section>
```

---

## utd-section-filtres-recherche

Variante de section dédiée aux filtres de recherche. Inclut un slot pour les boutons de recherche.

```html
<utd-section-filtres-recherche titre="Critères de recherche" extensible="false">
  <form id="formRecherche">
    <div class="utd-row-cols-lg-2">
      <div class="utd-col-auto">
        <utd-champ-form libelle="Numéro de dossier" format="lg">
          <input type="text">
        </utd-champ-form>
      </div>
      <div class="utd-col-auto">
        <utd-champ-form libelle="Date de transmission">
          <input type="date">
        </utd-champ-form>
      </div>
    </div>
    <div slot="boutons">
      <button type="button" class="utd-btn compact primaire">Rechercher</button>
      <button type="button" class="utd-btn compact tertiaire comme-lien">Réinitialiser</button>
    </div>
  </form>
</utd-section-filtres-recherche>
```

---

## utd-onglets / utd-onglet

### Attributs de utd-onglets

| Attribut | Type | Description |
|----------|------|-------------|
| `titre` | String **(Obligatoire)** | Titre du groupe d'onglets (accessibilité) |
| `titre-visible` | String (Optionnel) | Afficher le titre. Défaut invisible (SR seulement). |
| `tag-titre` | String (Optionnel) | Balise HTML du titre. Défaut `h2`. |
| `id-onglet-actif` | String (Optionnel) | Id de l'onglet actif. Lecture/écriture pour contrôle par code. |
| `conserver-etat-affichage` | String (Optionnel) | `"aucun"` (défaut), `"session"`, `"persistant"`. Nécessite un `id`. |

### Attributs de utd-onglet

- `titre` — Titre affiché dans l'onglet
- `id` — Id unique (requis pour `id-onglet-actif` et conservation d'état)

### Événements

- `affichageOnglet` : déclenché lors d'un changement d'onglet.
  `e.detail` = `{ indexe, id, titre }`

```html
<utd-onglets id="mesOnglets" titre="Mon groupe d'onglets" conserver-etat-affichage="session">
  <utd-onglet titre="Informations générales" id="onglet1">
    <div>
      <p>Contenu de l'onglet 1.</p>
    </div>
  </utd-onglet>
  <utd-onglet titre="Historique" id="onglet2">
    <div>
      <p>Contenu de l'onglet 2.</p>
    </div>
  </utd-onglet>
</utd-onglets>
```

Contrôle par code :
```javascript
// Afficher l'onglet 2
document.getElementById('mesOnglets').setAttribute('id-onglet-actif', 'onglet2');

// Écouter les changements
document.getElementById('mesOnglets').addEventListener('affichageOnglet', e => {
  console.log('Onglet affiché :', e.detail.titre, '(index', e.detail.indexe, ')');
});
```

---

## utd-menu-ancres

Génère automatiquement un menu de navigation par ancres en scannant les titres d'une page.

```html
<!-- Placer en haut du contenu principal, avant les sections -->
<utd-menu-ancres selecteur="#main h2">
</utd-menu-ancres>
```

- `selecteur` : sélecteur CSS des titres à inclure dans le menu.

---

## utd-fil-ariane

```html
<utd-fil-ariane>
  <a href="/">Accueil</a>
  <a href="/formulaires">Formulaires</a>
  <span>Demande de prestation</span>
</utd-fil-ariane>
```

---

## utd-lien-retour

Bouton de retour en arrière standardisé :

```html
<utd-lien-retour href="/formulaires" libelle="Retour à la liste des formulaires">
</utd-lien-retour>
```

---

## utd-tuile-conteneur / utd-tuile

Pour des blocs de navigation sous forme de tuiles cliquables.

### Attributs de utd-tuile-conteneur

| Attribut | Type | Description |
|----------|------|-------------|
| `sr-titre` | String (Optionnel/recommandé) | Titre SR du bloc de tuiles |
| `tag-titre` | String (Optionnel) | Balise HTML du titre des tuiles. Défaut `h2`. |
| `couleur-fond` | String (Optionnel) | `bleu-pale` (défaut), `gris-pale`, `transparent` |
| `fond-pleine-largeur` | Boolean (Optionnel) | Fond pleine largeur. Défaut `false`. |

### Attributs de utd-tuile

| Attribut | Description |
|----------|-------------|
| `titre` | Titre de la tuile **(Obligatoire)** |
| `href` | URL du lien **(Obligatoire)** |
| `icone-utd` | Classe CSS de l'icône (ex. `imprimante`, `pdf`) |
| `description` à `description5` | Lignes de description |

```html
<utd-tuile-conteneur sr-titre="Services disponibles" tag-titre="h2">
  <utd-tuile
    titre="Formulaires en ligne"
    href="/formulaires"
    icone-utd="formulaire"
    description="Accédez à tous vos formulaires">
  </utd-tuile>
  <utd-tuile
    titre="Mes demandes"
    href="/demandes"
    icone-utd="dossier"
    description="Suivez vos demandes en cours">
  </utd-tuile>
</utd-tuile-conteneur>
```

---

## utd-liste-liens-blocs

Navigation en liste de blocs (alternative aux tuiles) :

```html
<utd-liste-liens-blocs>
  <a href="/service-1">Service 1 — Description courte</a>
  <a href="/service-2">Service 2 — Description courte</a>
</utd-liste-liens-blocs>
```

---

## utd-consulter-aussi

Bloc "À consulter aussi" / liens connexes :

```html
<utd-consulter-aussi titre="À consulter aussi">
  <a href="/aide">Centre d'aide</a>
  <a href="/faq">Foire aux questions</a>
</utd-consulter-aussi>
```

---

## Liens externes — utd.ajusterAccessibiliteLiens()

Appeler cette méthode après le chargement de la page pour gérer automatiquement tous les
liens externes (`target="_blank"`) : ajout de l'icône, du texte SR et association du dernier
mot au symbole externe.

```javascript
// Au chargement de la page
utd.ajusterAccessibiliteLiens();

// Après injection de contenu dynamique
function chargerResultats() {
  // ... injection DOM ...
  utd.ajusterAccessibiliteLiens();
}
```

Pour les liens internes normaux : aucun traitement requis.

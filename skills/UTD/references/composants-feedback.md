# Composants de feedback UTD — Référence

## utd-avis

Messages contextuels affichés dans la page. Types : information, avertissement, succes, erreur, complementaire.

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `type` | String (Optionnel) | `information` (défaut), `avertissement`, `succes`, `erreur`, `complementaire astuce`, `complementaire rappel`, `complementaire communication`, `complementaire notification`, ou `complementaire ico-[nom]` |
| `titre` | String (Optionnel) | Titre de l'avis |
| `contenu` | String (Optionnel) | Texte simple (sinon slot défaut) |
| `focus` | String (Optionnel) | `"true"` pour donner le focus à l'avis (ex. après soumission avec erreurs) |

### Slots

| Slot | Description |
|------|-------------|
| défaut | Contenu HTML riche |

### Exemples

```html
<!-- Avis information avec attribut contenu -->
<utd-avis titre="Nouvelle règlementation"
  contenu="Les nouveaux règlements seront en vigueur à partir du 12 mai.">
</utd-avis>

<!-- Avis information avec contenu HTML riche -->
<utd-avis titre="Aide financière — Situation familiale">
  <p>Est considérée comme conjoint :</p>
  <ul>
    <li>la personne mariée ou unie civilement qui habite avec vous;</li>
    <li>l'autre parent d'au moins un de vos enfants qui habite avec vous.</li>
  </ul>
</utd-avis>

<!-- Avertissement -->
<utd-avis type="avertissement" titre="Attention">
  <p>Cette action est irréversible.</p>
</utd-avis>

<!-- Succès -->
<utd-avis type="succes" titre="Votre demande a été soumise avec succès."
  contenu="Un accusé de réception vous sera envoyé par courriel.">
</utd-avis>

<!-- Erreur — résumé des erreurs de formulaire avec focus automatique -->
<utd-avis id="avisErreurs" type="erreur" titre="Des erreurs sont présentes dans le formulaire.">
  <ul>
    <li><a href="#champNom">Le champ « Nom de famille » est obligatoire.</a></li>
    <li><a href="#champDate">Le champ « Date de naissance » est obligatoire.</a></li>
  </ul>
</utd-avis>

<!-- Complémentaire — sous-types -->
<utd-avis type="complementaire astuce" titre="Saviez-vous que..."
  contenu="Vous pouvez sauvegarder votre formulaire à tout moment.">
</utd-avis>
<utd-avis type="complementaire rappel" titre="À ne pas oublier">
  <ul>
    <li>Joindre une copie de votre carte d'assurance maladie;</li>
    <li>Signer le formulaire avant de le soumettre.</li>
  </ul>
</utd-avis>
<utd-avis type="complementaire communication" titre="Communication importante"
  contenu="Une mise à jour du système est prévue le 15 juin.">
</utd-avis>

<!-- Complémentaire avec icône personnalisée -->
<utd-avis type="complementaire ico-cadenas" titre="Information sécurisée"
  contenu="Vos données sont chiffrées et protégées.">
</utd-avis>
```

Donner le focus par code (après soumission avec erreurs) :
```javascript
document.getElementById('avisErreurs').setAttribute('focus', 'true');
```

---

## utd-alerte-generale

Alerte affichée dans le bandeau de la page (sous le menu horizontal). Utilisée pour des
messages importants de portée globale (ex. maintenance prévue, changement majeur).

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `type` | String (Optionnel) | `information` (défaut), `avertissement`, `succes`, `erreur` |
| `permanente` | Boolean (Optionnel) | `true` = pas de bouton de fermeture |

```html
<!-- À insérer entre le bandeau principal et le contenu, dans le <header> -->
<utd-alerte-generale type="avertissement">
  <p>Une maintenance est prévue le <strong>samedi 15 juin de 2h à 6h</strong>. Le service sera temporairement indisponible.</p>
</utd-alerte-generale>
```

Injection par JavaScript :
```javascript
const bandeauPrincipal = document.querySelector('.utd-bandeau-principal');
const alerte = document.createElement('utd-alerte-generale');
alerte.innerHTML = '<p>Message d\'alerte important.</p>';
bandeauPrincipal.after(alerte);
```

---

## utd-dialog (Dialogue modal)

Fenêtre modale accessible. Gère le piège de focus, la fermeture par Echap, le retour du focus.

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `titre` | String (Optionnel) | Titre de la modale |
| `afficher` | Boolean (Optionnel) | Affichage initial. Défaut `false`. Utiliser `utd.dialogue.afficher()` plutôt. |
| `type` | String (Optionnel) | Icône/couleur : `information`, `avertissement`, `succes`, `erreur` |
| `sr-bouton-fermer` | String (Optionnel) | Texte SR du bouton fermer (X) |
| `id-focus-ouverture` | String (Optionnel) | Id du contrôle à focaliser à l'ouverture (remplace comportement par défaut) |
| `id-focus-fermeture` | String (Optionnel) | Id du contrôle à focaliser à la fermeture (défaut : élément déclencheur) |
| `boutons-texte-long` | Boolean (Optionnel) | Boutons en colonne à partir de 525px (au lieu de 425px) |
| `forcer-boutons-inline` | Boolean (Optionnel) | Forcer les boutons en ligne (pas de colonne en mobile) |
| `affichage-lateral` | Boolean (Optionnel) | Animation d'ouverture latérale (droite → gauche) |

### Slots

| Slot | Description |
|------|-------------|
| défaut | Contenu de la modale |
| `pied` | Boutons dans le pied de la modale |

### Exemple complet

```html
<button type="button" id="btnOuvrirModale" class="utd-btn primaire">Ouvrir</button>

<utd-dialog id="maModale" titre="Confirmation de suppression" type="avertissement">
  <p>Êtes-vous certain de vouloir supprimer cet enregistrement ? Cette action est irréversible.</p>

  <div slot="pied">
    <button id="btnAnnuler" type="button" class="utd-btn secondaire compact">Annuler</button>
    <button id="btnConfirmer" type="button" class="utd-btn avertissement compact">Supprimer</button>
  </div>
</utd-dialog>

<script>
  document.getElementById('btnOuvrirModale').addEventListener('click', () => {
    utd.dialogue.afficher('maModale');
  });

  document.getElementById('btnAnnuler').addEventListener('click', () => {
    utd.dialogue.masquer('maModale');
  });

  document.getElementById('btnConfirmer').addEventListener('click', () => {
    // Traitement...
    utd.dialogue.masquer('maModale');
  });

  // Écouter l'événement de fermeture
  document.getElementById('maModale').addEventListener('fermeture', e => {
    console.log('Raison fermeture :', e.detail.raisonFermeture);
    // Valeurs possibles : 'boutonFermer', 'escape', ou selon votre code
  });
</script>
```

### Dialogue latéral (panneau latéral)

```html
<utd-dialog id="dialogueFeedback" titre="Votre avis" affichage-lateral="true"
  forcer-boutons-inline="true" id-focus-ouverture="texteFeedback">
  <form>
    <p class="utd-text-sm">Votre avis est anonyme.</p>
    <textarea id="texteFeedback" rows="3" placeholder="Décrivez votre expérience..."></textarea>
  </form>
  <div slot="pied">
    <button id="btnAnnulerFeedback" type="button" class="utd-btn secondaire compact">Annuler</button>
    <button id="btnEnvoyerFeedback" type="button" class="utd-btn primaire compact">Envoyer</button>
  </div>
</utd-dialog>
```

---

## Pastilles (badges de statut)

Classes CSS à appliquer à un élément `<span>` pour afficher un statut coloré :

```html
<span class="utd-pastille vert">Actif</span>
<span class="utd-pastille rouge">Inactif</span>
<span class="utd-pastille jaune">En attente</span>
<span class="utd-pastille bleu">En traitement</span>
<span class="utd-pastille gris">Archivé</span>
```

Couleurs disponibles : `vert`, `rouge`, `jaune`, `bleu`, `gris`.

---

## Étiquettes (tags)

```html
<span class="utd-etiquette">Formulaire</span>
<span class="utd-etiquette secondaire">Brouillon</span>
```

---

## Points de suspension / Traitement en cours (`utd-traitement-en-cours`)

Indicateur de chargement/traitement :

```html
<!-- Avec message -->
<utd-traitement-en-cours message="Chargement en cours, veuillez patienter...">
</utd-traitement-en-cours>

<!-- Sans message -->
<utd-traitement-en-cours></utd-traitement-en-cours>
```

Contrôle par code :
```javascript
document.getElementById('monTraitement').setAttribute('actif', 'true');
// ... traitement ...
document.getElementById('monTraitement').setAttribute('actif', 'false');
```

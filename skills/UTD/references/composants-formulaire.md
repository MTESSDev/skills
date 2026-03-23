# Composants de formulaire UTD — Référence

## utd-champ-form

Wrapper universel pour tous les types de champs. Gère automatiquement : label lié (`for`),
`aria-required`, `aria-invalid`, `aria-describedby` (précision + message erreur).

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `format` | String (Optionnel) | Largeur du champ : `sm`, `md`, `lg`, `xl` (défaut), `xxl`. Pour cases à cocher/radios : `compact`. Pour commutateur : `commutateur`. |
| `libelle` | String (Optionnel) | Texte du label |
| `obligatoire` | String (Optionnel) | `"true"` = affiche étoile rouge + aria-required |
| `precision` | String (Optionnel) | Texte de précision sous le label |
| `invalide` | Boolean (Optionnel) | `"true"` = affiche message d'erreur + état visuel erreur |
| `message-erreur` | String (Optionnel) | Message d'erreur (visible si `invalide="true"`) |
| `bloquer-coller` | Boolean (Optionnel) | Empêche le collage (utile pour mot de passe, confirmation courriel) |
| `max-caracteres` | Integer (Optionnel) | Compteur de caractères (textarea uniquement). Ne bloque pas la saisie. |
| `hauteur-automatique` | Boolean (Optionnel) | Hauteur auto selon contenu (textarea uniquement) |

### Slots

| Slot | Description |
|------|-------------|
| défaut | Le contrôle natif (`<input>`, `<textarea>`, `<select>`, `utd-liste-deroulante`, etc.) + optionnellement un `<span class="utd-erreur-champ">` pour message d'erreur manuel |

### Exemples

```html
<!-- Champ texte standard -->
<utd-champ-form
  id="champNom"
  obligatoire="true"
  libelle="Nom de famille"
  format="xl"
  invalide="false"
  precision="Tel qu'il apparaît sur vos documents officiels."
  message-erreur="Le champ « Nom de famille » est obligatoire.">
  <input type="text" maxlength="50"/>
</utd-champ-form>

<!-- Textarea avec compteur et hauteur auto -->
<utd-champ-form
  id="champDescription"
  libelle="Description"
  format="xxl"
  max-caracteres="500"
  hauteur-automatique="true"
  invalide="false"
  message-erreur="La description est obligatoire.">
  <textarea rows="3"></textarea>
</utd-champ-form>

<!-- Mot de passe — coller bloqué -->
<utd-champ-form libelle="Mot de passe" format="md" obligatoire="true"
  invalide="false" message-erreur="Le mot de passe est obligatoire." bloquer-coller="true">
  <input type="password"/>
</utd-champ-form>

<!-- Date -->
<utd-champ-form libelle="Date de naissance" obligatoire="true"
  invalide="false" message-erreur="La date de naissance est obligatoire.">
  <input type="date"/>
</utd-champ-form>

<!-- Message d'erreur géré manuellement (Razor Pages / Blazor) -->
<utd-champ-form obligatoire="true" libelle="Date de réception">
  <input type="date" asp-for="@Model.DateReception">
  <span asp-validation-for="@Model.DateReception" class="utd-erreur-champ"></span>
</utd-champ-form>
```

### Pattern de validation JavaScript

```javascript
const controleUtd = document.getElementById('champNom');
controleUtd.querySelector('input').addEventListener('blur', (event) => {
  controleUtd.setAttribute('invalide', event.target.value === '' ? 'true' : 'false');
});
```

### Blazor — Extensions helper recommandées

```csharp
// Attributs dans le razor
<utd-champ-form libelle="Nom" format="xl"
  invalide="@(editContext.EstChampInvalide("Nom"))"
  message-erreur="@(editContext.MessageErreurChamp("Nom"))">
  <InputText maxlength="50" id="Nom" @bind-Value="model.Nom"/>
</utd-champ-form>

// Extensions C#
public static class EditContextExtensions
{
    public static string EstChampInvalide(this EditContext? editContext, string NomChamp)
        => editContext?.GetValidationMessages(editContext.Field(NomChamp)).Any() == true ? "true" : "false";

    public static string MessageErreurChamp(this EditContext? editContext, string NomChamp)
    {
        if (editContext?.GetValidationMessages(editContext.Field(NomChamp)).Any() != true)
            return string.Empty;
        return string.Join("</br>", editContext.GetValidationMessages(editContext.Field(NomChamp)));
    }
}
```

---

## utd-liste-deroulante

Remplace le `<select>` natif par un composant accessible avec recherche optionnelle.
À placer à l'intérieur d'un `utd-champ-form`.

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `recherchable` | Boolean (Optionnel) | Activer la recherche dans la liste. Défaut `false`. |
| `recherche-floue` | Boolean (Optionnel) | Tolérance aux fautes de frappe. Défaut `true`. |
| `largeur` | String (Optionnel) | `lg` (528px), `md` (342px, défaut), `sm` (156px) |
| `refresh` | Boolean (Optionnel) | Mettre à `true` après modification programmatique du select natif |
| `focus` | Boolean (Optionnel) | Donner le focus au contrôle |
| `regex-separateur-recherche` | String (Optionnel) | Regex des séparateurs de recherche. Défaut : espaces. |
| `mots-exclus-recherche` | String (Optionnel) | Mots exclus de la recherche, séparés par virgule |
| `chargement-en-cours` | Boolean (Optionnel) | Affiche indicateur de chargement dans la liste |

### Exemples

```html
<!-- Liste simple -->
<utd-champ-form libelle="Province" obligatoire="true"
  invalide="false" message-erreur="Veuillez sélectionner votre province.">
  <utd-liste-deroulante>
    <label>Province</label>
    <select id="selectProvince">
      <option value="QC">Québec</option>
      <option value="ON">Ontario</option>
      <option value="AB">Alberta</option>
    </select>
  </utd-liste-deroulante>
</utd-champ-form>

<!-- Liste avec recherche (boîte combinée) -->
<utd-champ-form id="champMinistere" libelle="Ministère" obligatoire="true"
  invalide="false" message-erreur="Veuillez sélectionner un ministère.">
  <utd-liste-deroulante id="listeMinistere" recherchable="true">
    <select id="selectMinistere">
      <option value="MESS">Ministère de l'Emploi et de la Solidarité sociale</option>
      <option value="MSP">Ministère de la Sécurité publique</option>
      <!-- ... -->
    </select>
  </utd-liste-deroulante>
</utd-champ-form>

<!-- Liste multiple avec recherche -->
<utd-champ-form libelle="Langues maîtrisées">
  <utd-liste-deroulante recherchable="true">
    <select id="selectLangues" multiple>
      <option value="fr">Français</option>
      <option value="en">Anglais</option>
      <option value="es">Espagnol</option>
    </select>
  </utd-liste-deroulante>
</utd-champ-form>
```

### Mise à jour par code (JavaScript)

```javascript
// Modifier la valeur sélectionnée
const select = document.getElementById('selectProvince');
select.value = 'QC';
// IMPORTANT : rafraîchir la liste UTD après modification par code
document.getElementById('listeProvince').setAttribute('refresh', 'true');

// Chargement asynchrone des options
const listeUtd = document.querySelector('utd-liste-deroulante');
listeUtd.setAttribute('chargement-en-cours', 'true');
fetch('/api/provinces')
  .then(r => r.json())
  .then(data => {
    data.forEach(p => {
      select.add(new Option(p.nom, p.code));
    });
    listeUtd.removeAttribute('chargement-en-cours');
  });
```

---

## Boutons (`utd-btn`)

Classe CSS appliquée à un `<button>` ou `<a>` natif. Pas de web component.

### Types

```html
<!-- Primaire -->
<button type="button" class="utd-btn primaire">Enregistrer</button>
<button type="button" class="utd-btn primaire compact">Compact</button>
<button type="button" class="utd-btn primaire arrondi">Arrondi</button>
<button type="button" class="utd-btn primaire" disabled>Inactif</button>

<!-- Secondaire -->
<button type="button" class="utd-btn secondaire">Annuler</button>

<!-- Tertiaire -->
<button type="button" class="utd-btn tertiaire">Tertaire</button>
<button type="button" class="utd-btn tertiaire comme-lien">Affiché comme lien</button>

<!-- Avertissement -->
<button type="button" class="utd-btn avertissement">Supprimer</button>
```

### Navigation (précédent/suivant/retour)

```html
<button type="button" class="utd-btn primaire navigation precedent">
  <span class="utd-icone-svg" aria-hidden="true"></span>
  <span>Précédent</span>
</button>
<button type="button" class="utd-btn primaire navigation suivant">
  <span>Suivant</span>
  <span class="utd-icone-svg" aria-hidden="true"></span>
</button>
<button type="button" class="utd-btn secondaire navigation retour">
  <span class="utd-icone-svg" aria-hidden="true"></span>
  <span>Retour</span>
</button>
```

### Avec icône

```html
<button type="button" class="utd-btn primaire avec-icone-gauche">
  <span class="utd-icone-svg imprimante" aria-hidden="true"></span>
  <span>Imprimer</span>
</button>
<button type="button" class="utd-btn secondaire avec-icone-gauche compact">
  <span class="utd-icone-svg pdf" aria-hidden="true"></span>
  <span>Exporter PDF</span>
</button>
<!-- Icône seulement (sites internes uniquement — non autorisé en public) -->
<button type="button" class="utd-btn secondaire compact avec-icone" title="Modifier">
  <span class="utd-icone-svg crayon" aria-hidden="true"></span>
</button>
```

### Zone de boutons (mise en forme responsive)

```html
<!-- 2 boutons standard -->
<div class="utd-zone-boutons">
  <button type="button" class="utd-btn secondaire">Annuler</button>
  <button type="button" class="utd-btn primaire">Enregistrer</button>
</div>

<!-- 3 boutons ou texte long — affichage colonne à partir de 576px -->
<div class="utd-zone-boutons" affichage-colonne="sm">
  <button type="button" class="utd-btn tertiaire">Annuler</button>
  <button type="button" class="utd-btn secondaire">Supprimer</button>
  <button type="button" class="utd-btn primaire">Confirmer le changement</button>
</div>
```

### Bouton de téléversement de fichier

```html
<input type="file" aria-label="Fichier à transmettre" id="fichier1" class="utd">
<input type="file" aria-label="Fichiers à transmettre" multiple id="fichiers2" class="utd">
<input type="file" aria-label="Fichier" disabled class="utd">
```

---

## Case à cocher (`utd-champ-form` format compact)

```html
<utd-champ-form id="utdChampConsentement" obligatoire="true" format="compact"
  invalide="false" message-erreur="Vous devez accepter les conditions.">
  <label>
    <input type="checkbox" name="consentement" value="oui">
    <span>J'accepte les conditions d'utilisation</span>
  </label>
  <label>
    <input type="checkbox" name="consentement2" value="non">
    <span>Je refuse</span>
  </label>
</utd-champ-form>
```

Validation JavaScript (même pattern) :
```javascript
const controleUtd = document.getElementById('utdChampConsentement');
controleUtd.querySelectorAll('input').forEach((controle) => {
  ['blur', 'change'].forEach((event) => {
    controle.addEventListener(event, () => {
      controleUtd.setAttribute('invalide', !controleUtd.querySelector('input:checked'));
    });
  });
});
```

---

## Bouton radio (`utd-champ-form` format compact)

```html
<utd-champ-form id="utdChampStatut" obligatoire="true"
  libelle="Quel est votre statut ?"
  precision="Choisissez l'option qui correspond le mieux à votre situation."
  message-erreur="Vous devez sélectionner un statut.">
  <label>
    <input type="radio" name="statut" value="employe">
    <span>Employé</span>
  </label>
  <label>
    <input type="radio" name="statut" value="chomeur">
    <span>Sans emploi</span>
  </label>
  <label>
    <input type="radio" name="statut" value="retraite">
    <span>Retraité</span>
  </label>
</utd-champ-form>
```

---

## Commutateur (`utd-champ-form` format commutateur)

```html
<utd-champ-form format="commutateur" libelle="Recevoir les notifications par courriel">
  <label>
    <input type="checkbox" name="notifications">
    <span>Activer</span>
  </label>
</utd-champ-form>
```

---

## Champ avec bouton (`.utd-champ-bouton`)

```html
<utd-champ-form id="champRecherche" libelle="Numéro de dossier" format="md">
  <div class="utd-champ-bouton">
    <input type="text">
    <button type="button" class="utd-btn secondaire avec-icone" title="Rechercher">
      <span class="utd-icone-svg loupe" aria-hidden="true"></span>
    </button>
  </div>
</utd-champ-form>
```

---

## Champs combinés (`utd-groupe-champs-combines`)

Aligner plusieurs champs sur une même ligne (ex. téléphone + poste, dates) :

```html
<div class="utd-row utd-groupe-champs-combines">
  <div class="utd-col">
    <utd-champ-form libelle="Téléphone" obligatoire="true" format="md">
      <input type="tel">
    </utd-champ-form>
    <utd-champ-form libelle="Poste" format="sm">
      <input type="text">
    </utd-champ-form>
  </div>
</div>

<!-- Dates début/fin -->
<div class="utd-row utd-groupe-champs-combines" role="group" aria-labelledby="titrePeriode">
  <h3 id="titrePeriode">Période visée</h3>
  <div class="utd-col">
    <utd-champ-form libelle="Date de début" obligatoire="true">
      <input type="date">
    </utd-champ-form>
    <utd-champ-form libelle="Date de fin" obligatoire="true">
      <input type="date">
    </utd-champ-form>
  </div>
</div>
```

---

## Infobulle (`utd-infobulle`)

À placer à l'intérieur d'un `utd-champ-form`, juste avant le contrôle.

### Attributs

| Attribut | Type | Description |
|----------|------|-------------|
| `titre` | String (Optionnel) | Titre de l'infobulle |
| `sr-titre` | String (Optionnel) | Texte SR en préfixe. Défaut "Aide concernant ". |
| `contenu` | String (Optionnel) | Texte simple (sinon utiliser slot `contenu`) |
| `mode-affichage` | String (Optionnel) | `"feuille"` (défaut) ou `"bulle"`. Mobile force toujours "feuille". |

### Slots

| Slot | Description |
|------|-------------|
| `contenu` | Contenu HTML riche de l'infobulle |
| `texte-lie` | Texte lié (affiché en bleu pâle avant le bouton d'infobulle) |

```html
<utd-champ-form id="champDateNaissance" libelle="Date de naissance">
  <utd-infobulle titre="Format de la date">
    <div slot="contenu">
      <p>Inscrivez votre date de naissance au format <strong>AAAA-MM-JJ</strong>.</p>
    </div>
  </utd-infobulle>
  <input type="date">
</utd-champ-form>
```

---

## Groupe de champs (`utd-groupe-champs`)

Regrouper plusieurs champs avec un titre de groupe (accessibilité : `role="group"` + `aria-labelledby`) :

```html
<div class="utd-row-cols-lg-3 utd-row-cols-md-2 utd-row-cols-1 utd-groupe-champs"
     role="group" aria-labelledby="titreIdentite">
  <h3 id="titreIdentite">Identité</h3>
  <div class="utd-col">
    <utd-champ-form libelle="Nom" format="xl" obligatoire="true">
      <input type="text">
    </utd-champ-form>
  </div>
  <div class="utd-col">
    <utd-champ-form libelle="Prénom" format="xl" obligatoire="true">
      <input type="text">
    </utd-champ-form>
  </div>
  <div class="utd-col">
    <utd-champ-form libelle="Date de naissance">
      <input type="date">
    </utd-champ-form>
  </div>
</div>
```

---

## Bouton menu (`utd-btn-menu`)

```html
<utd-btn-menu libelle="Actions" id="btnActions" type="secondaire">
  <utd-btn-menu-item libelle="Modifier"></utd-btn-menu-item>
  <utd-btn-menu-item libelle="Supprimer"></utd-btn-menu-item>
  <utd-btn-menu-item libelle="Exporter"></utd-btn-menu-item>
</utd-btn-menu>
```

---

## Boutons de sélection (`utd-boutons-selection`)

Pour des choix mutuellement exclusifs affichés en boutons visuels :

```html
<utd-champ-form libelle="Type de demande">
  <utd-boutons-selection>
    <label>
      <input type="radio" name="typeDemande" value="nouveau">
      <span>Nouveau</span>
    </label>
    <label>
      <input type="radio" name="typeDemande" value="renouvellement">
      <span>Renouvellement</span>
    </label>
  </utd-boutons-selection>
</utd-champ-form>
```

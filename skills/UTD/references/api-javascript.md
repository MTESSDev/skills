# API JavaScript UTD — Référence

L'objet global `utd` expose des méthodes utilitaires et des composants contrôlables par code.
Il est disponible après le chargement du script `utd-webcomponents.min.js`.

---

## utd.message.afficher(params)

Affiche un **dialogue d'alerte** (fenêtre modale simple avec boutons) sans HTML préalable.
Retourne une **Promise** avec la raison de fermeture.

### Paramètres

| Paramètre | Type | Description |
|-----------|------|-------------|
| `type` | String (Optionnel) | `information`, `avertissement`, `succes`, `erreur` — ajoute une icône |
| `titre` | String | Titre du message |
| `corps` | String | Corps HTML du message |
| `texteBoutonPrimaire` | String | Texte du bouton primaire. Absent = bouton non affiché. |
| `texteBoutonSecondaire` | String (Optionnel) | Texte du bouton secondaire. Absent = bouton non affiché. |
| `texteBoutonFermer` | String (Optionnel) | Texte du bouton X. Défaut "Fermer". |
| `estBoutonsTexteLong` | Boolean (Optionnel) | Boutons en colonne à partir de 525px. Défaut `false`. |
| `idControleFocusFermeture` | String (Optionnel) | Id du contrôle à focaliser à la fermeture. Défaut : élément déclencheur. Mettre `"null"` pour gestion manuelle. |

### Valeurs de retour (Promise)

| Valeur | Signification |
|--------|---------------|
| `'primaire'` | Bouton primaire cliqué |
| `'secondaire'` | Bouton secondaire cliqué |
| `'boutonFermer'` | Bouton X cliqué |
| `'escape'` | Touche Echap pressée |

### Exemples

```javascript
// Message de confirmation (avertissement)
const retour = await utd.message.afficher({
  type: 'avertissement',
  titre: 'Confirmation de suppression',
  corps: '<p>Êtes-vous certain de vouloir supprimer cet enregistrement ?</p><p>Cette action est irréversible.</p>',
  texteBoutonPrimaire: 'Supprimer',
  texteBoutonSecondaire: 'Annuler'
});

if (retour === 'primaire') {
  // L'utilisateur a confirmé la suppression
  await supprimerEnregistrement();
  utd.notification.emettre({ type: 'positif', message: 'Enregistrement supprimé.' });
}

// Message informatif simple
utd.message.afficher({
  type: 'information',
  titre: 'Session expirée',
  corps: '<p>Votre session a expiré. Vous allez être redirigé vers la page de connexion.</p>',
  texteBoutonPrimaire: 'Reconnexion'
}).then(() => {
  window.location.href = '/connexion';
});

// Message d'erreur avec redirection du focus
utd.message.afficher({
  type: 'erreur',
  titre: 'Erreur de traitement',
  corps: '<p>Une erreur s\'est produite lors du traitement de votre demande.</p>',
  texteBoutonPrimaire: 'Réessayer',
  idControleFocusFermeture: 'btnSoumettre'
}).then((retour) => {
  if (retour === 'primaire') {
    document.getElementById('formPrincipal').dispatchEvent(new Event('submit'));
  }
});
```

---

## utd.notification.emettre(params)

Affiche une **notification contextuelle** temporaire (toast) en bas de page.
Ne retourne rien.

### Paramètres

| Paramètre | Type | Description |
|-----------|------|-------------|
| `type` | String (Optionnel) | `positif` (défaut), `negatif`, `neutre` |
| `titre` | String (Optionnel) | Titre de la notification |
| `message` | String | Texte de la notification (HTML autorisé) |
| `delaiFermeture` | Number (Optionnel) | Délai de fermeture en ms. Défaut 5000. |
| `texteBoutonFermer` | String (Optionnel) | Texte du bouton fermer. Défaut "Fermer". |

### Exemples

```javascript
// Succès après enregistrement
utd.notification.emettre({
  titre: 'Enregistrement réussi',
  message: 'Vos modifications ont été sauvegardées.'
});

// Notification sans titre
utd.notification.emettre({
  message: 'Données chargées avec succès.'
});

// Erreur (type negatif)
utd.notification.emettre({
  type: 'negatif',
  titre: 'Erreur',
  message: 'Impossible de se connecter au serveur. Veuillez réessayer.'
});

// Notification neutre persistante (10 secondes)
utd.notification.emettre({
  type: 'neutre',
  titre: 'Information',
  message: 'Une mise à jour est disponible. Rechargez la page.',
  delaiFermeture: 10000
});

// Avec HTML dans le message
utd.notification.emettre({
  type: 'positif',
  message: '<p>Formulaire soumis.</p><p>Numéro de référence : <strong>2024-001234</strong></p>'
});
```

---

## utd.dialogue.afficher(id) / utd.dialogue.masquer(id)

Ouvrir ou fermer un composant `utd-dialog` existant dans le DOM.

```javascript
// Ouvrir
utd.dialogue.afficher('idDeMaModale');

// Fermer
utd.dialogue.masquer('idDeMaModale');
```

**Voir `references/composants-feedback.md`** pour l'utilisation complète de `utd-dialog`.

---

## utd.ajusterAccessibiliteLiens()

Traite automatiquement tous les `<a target="_blank">` dans la page :
- Ajoute l'icône de lien externe
- Lie l'icône au dernier mot du texte du lien (évite l'icône seule sur une ligne)
- Ajoute le texte SR en français ou en anglais selon la langue de la page

```javascript
// Appeler au chargement de la page
utd.ajusterAccessibiliteLiens();

// Et après chaque injection de contenu dynamique
async function chargerResultats() {
  const data = await fetch('/api/resultats').then(r => r.json());
  document.getElementById('conteneurResultats').innerHTML = genererHtml(data);
  utd.ajusterAccessibiliteLiens(); // Re-traiter les nouveaux liens
}
```

---

## utd.scrollIntoViewSiRequis(element)

Fait défiler vers l'élément si celui-ci n'est pas visible dans la fenêtre.

```javascript
const element = document.getElementById('avisErreurs');
element.focus();
utd.scrollIntoViewSiRequis(element);
```

---

## Combinaison message + notification (pattern complet)

Pattern recommandé pour les actions avec confirmation puis feedback :

```javascript
async function supprimerDossier(idDossier) {
  // 1. Demander confirmation
  const retour = await utd.message.afficher({
    type: 'avertissement',
    titre: 'Confirmation',
    corps: `<p>Supprimer le dossier #${idDossier} ?</p>`,
    texteBoutonPrimaire: 'Supprimer',
    texteBoutonSecondaire: 'Annuler'
  });

  if (retour !== 'primaire') return;

  try {
    // 2. Effectuer l'action
    await fetch(`/api/dossiers/${idDossier}`, { method: 'DELETE' });

    // 3. Notifier le succès
    utd.notification.emettre({
      titre: 'Suppression réussie',
      message: `Le dossier #${idDossier} a été supprimé.`
    });

    // 4. Mettre à jour l'interface
    rafraichirListe();
  } catch (e) {
    utd.notification.emettre({
      type: 'negatif',
      titre: 'Erreur',
      message: 'La suppression a échoué. Veuillez réessayer.'
    });
  }
}
```

---

## utd.datatables (intégration DataTables)

Si DataTables est utilisé, UTD fournit une intégration pour appliquer les styles gouvernementaux.

```javascript
// Définir les paramètres par défaut UTD avant tout new DataTable()
utd.datatables.definirParametresDefaut();

// Puis initialiser DataTables normalement
const tableau = new DataTable('#monTableau', {
  columns: [
    { data: 'nom', title: 'Nom' },
    { data: 'statut', title: 'Statut',
      render: (data, type, row) => {
        const couleur = data === 'Actif' ? 'vert' : 'rouge';
        return `<span class="utd-pastille ${couleur}">${data}</span>`;
      }
    }
  ]
});
```

> DataTables doit être chargé séparément via CDN ou bundle.
> `<script src="https://cdn.datatables.net/v/dt/...datatables.min.js"></script>`

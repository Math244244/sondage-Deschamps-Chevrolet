# 🚀 GUIDE DE MIGRATION & STANDARDISATION (Sondage V2)
**Objectif :** Transformer un projet de sondage existant pour qu'il adopte l'architecture "St-Georges Chevrolet" (Optimisée Mobile & Google Avis).

---

## 1. STRUCTURE DES FICHIERS
Le projet doit être nettoyé pour ne contenir que ces fichiers essentiels. Supprimez tout le reste.

```text
MON-PROJET/
├── index.html                   <-- L'application complète (Code + Design)
├── _redirects                   <-- Règle de redirection Netlify (Optionnel mais recommandé)
├── Instructions.txt             <-- Guide pour l'utilisateur final
├── Template-Email-Sondage.html  <-- Modèle HTML avec bouton
├── Template-Email-Texte.txt     <-- Modèle texte simple
└── Template-SMS.txt             <-- Modèle pour texto
```

---

## 2. MODIFICATION DU CODE (index.html)

### A. Configuration (Haut du fichier)
Remplacez la section configuration pour utiliser le lien Google Avis **STABLE** (format `writereview`).
*Évitez les liens "lrd" ou "maps" génériques qui causent des erreurs sur mobile.*

```javascript
const CONFIG = {
  // ... autres configs ...
  
  // LIEN CRITIQUE : Utilisez ce format exact avec le PlaceID du client
  GOOGLE_REVIEW_URL: 'https://search.google.com/local/writereview?placeid=VOTRE_PLACE_ID_ICI',
  GOOGLE_REVIEW_URL_DIRECT: 'https://search.google.com/local/writereview?placeid=VOTRE_PLACE_ID_ICI',
};
```

### B. Correction du Bug Mobile (Suppression du Mailto)
Cherchez la fonction `sendResult(data)`. Elle contient probablement une ligne `window.location.href = mailto:...`.
**IL FAUT LA SUPPRIMER.** C'est elle qui cause le menu "Ouvrir avec" sur Android/iOS.

**Remplacez la fonction `sendResult` par celle-ci :**

```javascript
async function sendResult(data) {
    try {
        // Envoi silencieux vers serveur (si configuré)
        if (navigator.sendBeacon && CONFIG.API_ENDPOINT) {
            if (navigator.sendBeacon(CONFIG.API_ENDPOINT, new Blob([JSON.stringify(data)], { type: 'application/json' }))) return;
        }
        if (CONFIG.API_ENDPOINT) {
            await fetch(CONFIG.API_ENDPOINT, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data), keepalive: true });
            return;
        }
        
        // CRITIQUE : Ne JAMAIS utiliser window.location.href = 'mailto:...' ici.
        // Cela bloque la redirection vers Google sur mobile.
        console.log('Sondage complété. Redirection Google imminente.', data);

    } catch (e) {
        console.error('Erreur envoi:', e);
    }
}
```

### C. Logique de Redirection
Assurez-vous que la redirection se fait **après** la tentative d'envoi, dans `submitSurvey`.

```javascript
if (isPositiveExperience()) {
    // Petit délai pour laisser l'UI respirer
    setTimeout(() => { window.location.href = CONFIG.GOOGLE_REVIEW_URL; }, 100);
} else {
    // Afficher remerciement interne
    state.step = 4;
    render();
}
```

---

## 3. STANDARDISATION DES TEMPLATES

### A. Template Email HTML (`Template-Email-Sondage.html`)
*   Assurez-vous que le lien du bouton est mis à jour : `<a href="https://NOM-DU-PROJET.pages.dev">`
*   Utilisez des styles "inline" (CSS dans les balises) pour la compatibilité Gmail/Outlook.
*   **Pas d'images externes** (logos) sauf si absolument nécessaire (encodé en Base64 ou hébergé sur un CDN fiable).

### B. Templates Texte (`.txt`)
Vérifiez simplement que l'URL est la bonne.
*   Template SMS : "Bonjour... 30 secondes... 👉 https://NOM-DU-PROJET.pages.dev"

---

## 4. DÉPLOIEMENT (NETLIFY)

1.  **Créer le site :** Glisser-déposer le dossier sur Netlify.
2.  **Renommer IMMÉDIATEMENT :**
    *   Aller dans `Site settings` > `Change site name`.
    *   Choisir un nom propre : `sondage-client` (ex: `sondage-stgeorgesgm`).
3.  **Mise à jour des liens :**
    *   Une fois le nom validé, revenez dans vos fichiers locaux.
    *   Faites un "Chercher/Remplacer" de l'ancien nom (ou placeholder) vers le nouveau nom `https://sondage-client.netlify.app`.
    *   Sauvegardez tout.

---

## 5. CHECKLIST DE VALIDATION

Avant de livrer au client, testez ceci sur un **TÉLÉPHONE CELLULAIRE** :
1.  [ ] Ouvrir le lien du sondage.
2.  [ ] Répondre : Excellent / Excellent / Oui.
3.  [ ] **VÉRIFICATION CRITIQUE :** Êtes-vous redirigé vers Google Avis **SANS** voir apparaître un menu "Ouvrir avec Gmail/Outlook" ?
4.  [ ] Si oui, le projet est validé.
5.  [ ] Si non, vérifiez que vous avez bien supprimé le `mailto` dans le JavaScript.

---
*Ce guide est basé sur l'architecture validée du projet St-Georges Chevrolet (Dec 2025).*




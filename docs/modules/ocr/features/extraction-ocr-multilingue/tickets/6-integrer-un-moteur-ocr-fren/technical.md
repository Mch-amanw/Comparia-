## Ticket : Intégrer un moteur OCR FR/EN

### 1. Architecture d’intégration

Le moteur OCR sera intégré dans le module OCR existant selon l’architecture définie :

Pipeline :
1. Upload
2. Détection PDF scanné
3. Appel moteur OCR (ce ticket)
4. Normalisation
5. Comparaison

Deux modes d’intégration possibles selon environnement :
- SaaS : microservice OCR dédié (recommandé), scalable horizontalement.
- On-premise : service interne embarqué dans le backend.

L’intégration doit être abstraite via une interface interne (ex : `OcrService`) afin de permettre un éventuel remplacement/versionnement futur du moteur.

---

### 2. Interface technique

#### 2.1 Entrées

- `documentId`
- Flux binaire du PDF
- Paramètres :
  - `languageMode` : `auto` | `forced`
  - `forcedLanguage` : `fr` | `en` | null
  - `tenantId`

#### 2.2 Sorties

Objet structuré :

- `pages[]`
  - `pageNumber`
  - `blocks[]`
    - `blockIndex`
    - `text`
- `detectedLanguage`
- `confidenceScore` (0–1 ou % selon moteur)
- `processingTimeMs`
- `engineVersion`
- `status` : success | failure

Encodage obligatoire : UTF-8.

---

### 3. Gestion linguistique

- Si `languageMode = forced` → charger explicitement le modèle FR ou EN.
- Si `languageMode = auto` → utiliser le mécanisme de détection interne du moteur ou un pré-traitement de détection.

La langue réellement utilisée doit être persistée dans les métadonnées de traitement.

---

### 4. Traitement technique

Étapes minimales :
1. Conversion PDF → images par page.
2. Prétraitement léger (redressement, amélioration contraste si supporté).
3. Appel OCR avec modèle linguistique approprié.
4. Reconstruction en blocs ordonnés.
5. Normalisation encodage UTF-8.

Le traitement doit être parallélisable par page si l’infrastructure le permet.

---

### 5. Gestion des erreurs

Cas à gérer :
- Exception moteur.
- Timeout configurable.
- Dépassement mémoire.

Comportement :
- Capture et journalisation structurée.
- Retour `status = failure`.
- Aucun envoi de texte partiel au module de normalisation.

Les logs techniques doivent inclure :
- `documentId`
- `tenantId`
- nombre de pages
- durée
- erreur technique

---

### 6. Performance et optimisation

Contraintes :
- Support jusqu’à 50 Mo / 300 pages.
- Optimisation mémoire (streaming recommandé pour conversion page par page).
- Possibilité d’exécution via workers OCR dédiés.

Des métriques doivent être exposées :
- Temps total par document.
- Temps moyen par page.
- Taux d’échec.

---

### 7. Sécurité

- Traitement en environnement isolé (conteneur recommandé en SaaS).
- Suppression des fichiers temporaires après traitement.
- Chiffrement en transit.
- Isolation stricte des données par tenant.

---

### 8. Versionnement

Le moteur OCR intégré doit exposer un identifiant de version (`engineVersion`).

Toute évolution majeure du moteur nécessitera :
- Mise à jour de la version exposée.
- Traçabilité dans les métadonnées.

---

### 9. Dépendances techniques

- Module de stockage (lecture PDF).
- Infrastructure file de tâches.
- Module de normalisation.
- Module d’audit.
- Configuration tenant.

Aucune dépendance externe ne doit empêcher un déploiement on-premise conforme aux exigences du projet.
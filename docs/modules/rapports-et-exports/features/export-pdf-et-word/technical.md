# Spécification technique

## 1. Composants impactés

- Frontend (écran de comparaison – bouton export + paramètres).
- API backend : endpoint POST /comparisons/{id}/exports.
- reportBuilder.
- templateEngine.
- formatRenderer (PDF + DOCX).
- exportStorageHandler.
- exportAuditLogger.

---

## 2. Flux technique

1. Le frontend envoie une requête POST /comparisons/{id}/exports avec :
   - format (pdf | docx)
   - parameters (includeComments, includeValidatedOnly, detailLevel, language)
2. Vérification des droits utilisateur.
3. Récupération du comparisonReportModel v1.
4. Application des filtres selon paramètres.
5. Construction du modèle intermédiaire via reportBuilder.
6. Application du template (standard ou marque blanche).
7. Génération via formatRenderer :
   - PDF : moteur HTML → PDF ou bibliothèque dédiée.
   - DOCX : génération OpenXML.
8. Stockage ou mise à disposition directe selon configuration.
9. Écriture d’un log via exportAuditLogger.

---

## 3. Spécificités techniques PDF

- Support pagination automatique.
- Styles injectés dynamiquement selon branding.
- Encodage UTF-8.
- Gestion des tableaux et sections longues.

Contraintes :
- Support jusqu’à 300 pages.
- Mode asynchrone si seuil dépassé (> 100 pages ou seuil configurable de différences).

---

## 4. Spécificités techniques DOCX

- Génération via bibliothèque OpenXML.
- Utilisation des styles :
  - Heading 1, Heading 2.
  - Paragraph styles dédiés aux différences.
- Surlignage natif Word pour ajouts/suppressions.
- Gestion correcte des tableaux.

---

## 5. Performance

- Mode synchrone par défaut si document ≤ 100 pages.
- Mode asynchrone via jobQueue sinon.
- Statuts : pending → processing → completed → failed.
- Objectif : < 10 secondes en mode synchrone.

---

## 6. Sécurité

- Vérification d’autorisation avant génération.
- Transmission via HTTPS.
- Chiffrement au repos si archivage activé.
- URL d’accès signée et temporaire pour téléchargement.

---

## 7. Journalisation

Chaque export génère un enregistrement contenant :
- exportId
- comparisonId
- format
- paramètres utilisés
- userId
- date/heure
- scoringModelVersion
- hash SHA-256 du fichier

Les logs sont transmis au module d’audit central.

---

## 8. Contraintes de déterminisme

À données identiques et paramètres identiques :
- Le contenu généré (hors métadonnées temporelles) doit être strictement identique.
- Le hash calculé doit être reproductible.

---
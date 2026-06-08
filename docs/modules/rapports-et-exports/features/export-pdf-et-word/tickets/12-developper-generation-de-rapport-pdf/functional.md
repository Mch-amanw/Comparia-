## Ticket : Développer génération de rapport PDF

### 1. Objectif
Implémenter la génération d’un rapport de comparaison au format **PDF**, conforme aux spécifications du module "Rapports et exports" et de la fonctionnalité "Export PDF et Word".

Ce rapport doit permettre à un utilisateur autorisé de télécharger un document figé contenant :
- Le score global de similarité (obligatoire),
- La synthèse des différences détectées,
- Le détail des différences selon les paramètres sélectionnés.

---

### 2. Périmètre fonctionnel

Ce ticket couvre exclusivement :
- La construction du contenu du rapport PDF,
- La mise en forme structurée du document,
- L’intégration des données issues du comparisonReportModel v1,
- La prise en charge des paramètres d’export transmis (filtres et niveau de détail).

Ne sont pas inclus dans ce ticket :
- L’export Word (.docx),
- Les exports JSON ou Excel,
- La logique de scoring (déjà produite en amont),
- La gestion complète de la file asynchrone (déclenchement géré au niveau supérieur).

---

### 3. Contenu du rapport PDF

Le PDF généré doit inclure les sections suivantes :

#### 3.1 Page de synthèse
- Nom du document A
- Nom du document B
- Date et heure de la comparaison
- Auteur de la comparaison
- Version du modèle de scoring (ex : v1)
- Score global de similarité (mis en évidence)
- Sous-scores (ST, SH, SB, SS) si disponibles

#### 3.2 Synthèse des différences
- Nombre total d’ajouts
- Nombre total de suppressions
- Nombre total de modifications
- Nombre total de sections déplacées

Les différences identifiées comme « déplacées » doivent être explicitement mentionnées.

#### 3.3 Détail des différences
Selon les paramètres transmis :
- Liste structurée des différences
- Type (ajout, suppression, modification, déplacement)
- Section concernée
- Extrait avant / après (si applicable)
- Statut (validé, refusé, en attente)
- Commentaires associés (si includeComments = true)

Si le niveau de détail est "résumé", seules les informations synthétiques sont incluses.

---

### 4. Contraintes de présentation

- Document non modifiable (format figé).
- Mise en page structurée avec titres hiérarchiques.
- Pagination automatique.
- Intégration du branding tenant si configuré (logo, couleurs principales).
- Encodage UTF-8.

Le score global doit être visuellement mis en avant (taille supérieure ou encadré distinct).

---

### 5. Règles de gestion

1. Le score affiché doit correspondre strictement au scoringModelVersion fourni.
2. Les filtres appliqués (includeValidatedOnly, includeComments, detailLevel) doivent être respectés dans le contenu généré.
3. Le contenu du PDF doit être cohérent avec les données visibles côté interface au moment de la génération.
4. Deux exports générés avec les mêmes données et paramètres doivent produire un contenu identique (hors métadonnées temporelles).
5. Les différences marquées comme déplacées doivent être clairement identifiées comme telles.

---

### 6. Dépendances

- comparisonReportModel v1 (fourni par le service de comparaison).
- templateEngine (pour application du template standard ou marque blanche).
- exportStorageHandler (pour stockage temporaire ou archivage).
- exportAuditLogger (journalisation de la génération).
- Configuration tenant (branding, langue, mode de stockage).

---

### 7. Résultat attendu

À l’issue de l’implémentation, le backend doit être capable de générer un fichier PDF conforme aux règles ci-dessus à partir des données structurées d’une comparaison et des paramètres d’export fournis.
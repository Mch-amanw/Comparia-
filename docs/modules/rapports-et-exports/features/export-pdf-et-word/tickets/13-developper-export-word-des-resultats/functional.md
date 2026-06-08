## Ticket : Développer export Word des résultats

### 1. Objectif
Implémenter la génération d’un rapport de comparaison au format **Word (.docx)**, structuré, modifiable et conforme aux spécifications du module « Rapports et exports » et de la fonctionnalité « Export PDF et Word ».

Cet export doit permettre aux utilisateurs autorisés de télécharger un document Word contenant :
- Le score global de similarité,
- Les sous-scores si disponibles,
- Une synthèse des différences,
- Le détail structuré des différences détectées,
- Les métadonnées de comparaison.

---

### 2. Déclenchement

Depuis l’écran de consultation d’une comparaison finalisée :
- L’utilisateur sélectionne le format **Word (.docx)**.
- Il peut configurer les paramètres disponibles :
  - includeComments
  - includeValidatedOnly
  - detailLevel (summary | standard | exhaustive)
  - language (fr | en)

La génération est soumise aux règles de droits d’accès définies au niveau de la comparaison.

---

### 3. Structure du document Word généré

Le document Word doit respecter une structure hiérarchique claire et cohérente.

#### 3.1 Page de synthèse
Contient :
- Nom du document A
- Nom du document B
- Date et heure de la comparaison
- Auteur
- Version du modèle de scoring
- Score global de similarité (mis en évidence)
- Sous-scores (ST, SH, SB, SS) si disponibles

#### 3.2 Synthèse des différences
Section dédiée présentant :
- Nombre total d’ajouts
- Nombre total de suppressions
- Nombre total de modifications
- Nombre de sections déplacées

#### 3.3 Détail des différences
Selon le niveau de détail :
- Liste structurée des différences
- Pour chaque différence :
  - Type (ajout, suppression, modification, déplacement)
  - Section concernée
  - Extrait avant / après (si applicable)
  - Statut de validation
  - Commentaires associés (si includeComments = true)

Les différences marquées comme « déplacées » doivent être explicitement identifiées.

---

### 4. Mise en forme et éditabilité

Le document Word doit :
- Être entièrement modifiable.
- Utiliser des niveaux de titres structurés (Heading 1, Heading 2, etc.).
- Mettre en évidence les différences :
  - Ajouts : surlignage dédié.
  - Suppressions : indication visuelle claire.
  - Modifications : distinction avant / après.
- Conserver les tableaux sous forme éditable.

La hiérarchie des sections doit refléter la structure logique du rapport.

---

### 5. Respect des paramètres

Le contenu généré doit respecter :
- Les filtres appliqués (ex : uniquement différences validées).
- Le niveau de détail sélectionné.
- La langue choisie (FR/EN).
- La configuration de marque blanche du tenant (logo, couleurs, nom organisation si activé).

---

### 6. Stockage et disponibilité

Selon la configuration du tenant :

- Mode suppression automatique :
  - Le fichier est disponible au téléchargement puis supprimé automatiquement après la durée configurée.

- Mode archivage :
  - Le fichier est conservé et rattaché à la comparaison.
  - Accessible ultérieurement aux utilisateurs autorisés.

---

### 7. Traçabilité

Chaque génération d’export Word doit :
- Être associée à un identifiant unique d’export.
- Inclure la version du modèle de scoring.
- Être journalisée via le module d’audit.

Le contenu généré doit être cohérent avec les données de comparaison au moment de la génération.
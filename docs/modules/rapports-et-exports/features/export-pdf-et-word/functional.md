# Fonctionnalité : Export PDF et Word

## 1. Objectif métier

Permettre aux utilisateurs autorisés de générer et télécharger un rapport de comparaison au format **PDF** ou **Word (.docx)**, contenant une synthèse claire des différences détectées ainsi que le **score global de similarité**.

Cette fonctionnalité vise à :
- Faciliter le partage des résultats avec des tiers (clients, juristes, direction).
- Permettre l’archivage formel d’une comparaison.
- Fournir un livrable exploitable, cohérent avec l’affichage de l’interface.
- Garantir la traçabilité (version du scoring, date, auteur).

---

## 2. Périmètre fonctionnel

### 2.1 Déclenchement de l’export

Depuis l’écran de consultation d’une comparaison finalisée :
- L’utilisateur peut choisir le format : **PDF** ou **Word (.docx)**.
- L’utilisateur peut configurer certains paramètres d’export (selon droits) :
  - Inclure ou non les commentaires.
  - Inclure uniquement les différences validées.
  - Niveau de détail : résumé ou détaillé.

Règle :
- Seuls les utilisateurs ayant accès à la comparaison peuvent générer un export.

---

### 2.2 Contenu du rapport

Le rapport généré doit inclure au minimum :

#### A. Page de synthèse
- Nom des documents comparés.
- Date et heure de la comparaison.
- Auteur de la comparaison.
- Version du modèle de scoring (ex : v1).
- **Score global de similarité (obligatoire)**.
- Sous-scores (ST, SH, SB, SS) si disponibles.

#### B. Synthèse des différences
- Nombre total d’ajouts.
- Nombre total de suppressions.
- Nombre total de modifications.
- Nombre de sections déplacées.

#### C. Détail des différences
Selon le niveau de détail choisi :
- Liste structurée des différences.
- Type : ajout, suppression, modification, déplacement.
- Section concernée.
- Extrait avant / après (si applicable).
- Statut : validé, refusé, en attente.
- Commentaires associés (si inclus).

Les différences marquées comme « déplacées » doivent être explicitement identifiées.

---

### 2.3 Spécificités par format

#### 2.3.1 Export PDF
- Document figé, non modifiable.
- Mise en forme structurée et professionnelle.
- Intégration du branding tenant (logo, couleurs) si activé.
- Compatible archivage documentaire.

#### 2.3.2 Export Word (.docx)
- Document modifiable.
- Structuration avec niveaux de titres (Heading 1, Heading 2, etc.).
- Mise en évidence des différences via surlignage natif.
- Contenu éditable (textes, tableaux).

---

### 2.4 Gestion des droits

- L’utilisateur doit avoir un droit de consultation sur la comparaison.
- L’export respecte les droits d’accès au document.
- Toute génération ou téléchargement est journalisé.

---

### 2.5 Mode de stockage

Selon la configuration du tenant :

- **Mode suppression automatique** :
  - Le fichier est disponible au téléchargement puis supprimé automatiquement après une durée configurable.

- **Mode archivage** :
  - Le rapport est conservé et rattaché à la comparaison.
  - Il reste accessible aux utilisateurs autorisés.

---

## 3. Règles de gestion

1. Le score affiché doit correspondre strictement au modèle de scoring versionné utilisé lors de la comparaison.
2. Le contenu du rapport doit être cohérent avec la vue affichée au moment de la génération (filtres appliqués inclus).
3. Toute génération d’export déclenche une entrée dans le journal d’audit.
4. Les rapports archivés restent associés à leur version de scoring et de template.
5. Deux exports avec les mêmes données et paramètres doivent produire un contenu identique (hors timestamp si applicable).

---

## 4. Critères d’acceptation

- ✅ Un utilisateur autorisé peut générer un PDF contenant le score global et la synthèse des différences.
- ✅ Un utilisateur autorisé peut générer un Word structuré et modifiable.
- ✅ Les différences déplacées sont clairement identifiées.
- ✅ Les paramètres (commentaires, filtres) sont respectés dans le rapport.
- ✅ Une entrée d’audit est créée à chaque génération.
- ✅ Le rapport inclut la version du modèle de scoring.
- ✅ Le comportement diffère correctement selon le mode suppression ou archivage.

---
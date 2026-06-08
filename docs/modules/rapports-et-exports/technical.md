# Module : Rapports et exports – Spécification technique

## 1. Objectif technique du module

Le module "Rapports et exports" est responsable de la génération, de la mise en forme et de l’export des résultats d’une comparaison de documents sous différents formats exploitables (PDF, Word, évolution vers Excel et JSON).

Il doit :
- Produire des rapports fidèles, déterministes et traçables.
- Être compatible SaaS et on-premise.
- Supporter des traitements synchrones et asynchrones.
- Garantir la sécurité et la confidentialité des données exportées.

---

## 2. Architecture du module

### 2.1 Positionnement dans l’architecture globale

Le module est un service backend dédié :

- Input :
  - Résultats structurés de comparaison (diff, scores, métadonnées).
  - Configuration tenant (branding, langue, pondérations).
  - Paramètres d’export (format, niveau de détail, inclusion commentaires).

- Output :
  - Fichier généré (PDF, DOCX, JSON, futur XLSX).
  - Métadonnées d’export (version scoring, date, utilisateur).

### 2.2 Composants internes

1. reportBuilder
   - Transforme les données de comparaison en modèle de rapport intermédiaire (format structuré interne JSON).

2. templateEngine
   - Applique un template (standard ou marque blanche).
   - Gère la localisation (FR/EN).

3. formatRenderer
   - Rend le rapport dans le format cible (PDF, DOCX, etc.).

4. exportStorageHandler
   - Gère le stockage temporaire ou persistant.
   - Applique la politique de conservation (suppression ou archivage).

5. exportAuditLogger
   - Journalise chaque génération/export.

---

## 3. Modèle de données d’entrée

Le module consomme un objet structuré :

comparisonReportModel v1 :
- comparisonId : string
- tenantId : string
- scoringModelVersion : string
- globalScore : number (0–100)
- subScores :
  - textScore
  - headingScore
  - tableScore
  - structureScore
- differences :
  - type (addition, deletion, modification, moved)
  - location (sectionId, page, offset)
  - originalText
  - modifiedText
  - validatedStatus
  - comments[]
- sections[]
- metadata :
  - documentAName
  - documentBName
  - comparedBy
  - comparedAt
  - language

Ce modèle garantit la séparation entre logique métier et rendu.

---

## 4. Formats supportés

### 4.1 PDF

- Génération via moteur HTML → PDF ou librairie dédiée.
- Doit être :
  - Non modifiable par défaut.
  - Compatible archivage.
  - Support des en-têtes/pieds personnalisés.

Contraintes :
- Support documents jusqu’à 300 pages.
- Gestion pagination automatique.

### 4.2 Word (DOCX)

- Génération via librairie OpenXML.
- Structure :
  - Page de garde
  - Résumé exécutif
  - Détail des différences
  - Annexes éventuelles

Contraintes :
- Conservation des styles structurés (Heading 1, Heading 2, etc.).
- Surlignage natif Word pour différences.

### 4.3 JSON (évolution prioritaire API)

- Export brut structuré.
- Conforme au comparisonReportModel.
- Utilisable via API.
- Encodage UTF-8.

### 4.4 Excel (évolution)

- Format XLSX.
- Onglets :
  - Résumé
  - Liste des différences
  - Détail par section

---

## 5. Paramétrage des exports

### 5.1 Paramètres utilisateur

- includeComments : boolean
- includeValidatedOnly : boolean
- includeUnresolvedOnly : boolean
- detailLevel : summary | standard | exhaustive
- language : fr | en

### 5.2 Paramètres tenant

- logo
- couleurs principales
- nom affiché
- mentions légales
- pondérations affichées

---

## 6. Gestion des performances

### 6.1 Mode synchrone

Autorisé si :
- Document ≤ 100 pages
- Nombre de différences raisonnable

Temps cible : < 10 secondes.

### 6.2 Mode asynchrone

Déclenché si :
- > 100 pages
- > seuil configurable de différences

Mécanisme :
- Mise en file (jobQueue).
- Statut : pending → processing → completed → failed.
- Notification utilisateur à la fin.

---

## 7. Stockage et cycle de vie

### 7.1 Mode suppression automatique

- Fichier stocké temporairement (durée configurable, ex : 24h).
- Suppression automatique via tâche planifiée.

### 7.2 Mode archivage

- Stockage chiffré.
- Indexation par comparisonId.
- Lien avec journal d’audit.

### 7.3 Sécurité

- Chiffrement au repos.
- URL d’accès signée et temporaire.
- Contrôle d’accès basé sur rôles.

---

## 8. Journalisation et audit

Chaque export génère un log contenant :
- exportId
- comparisonId
- format
- paramètres utilisés
- utilisateur
- date/heure
- scoringModelVersion
- hash du fichier généré (intégrité)

Les logs sont immuables.

---

## 9. Versioning et traçabilité

Le rapport doit inclure :
- Version du modèle de scoring (ex : v1).
- Date de génération.
- Identifiant unique du rapport.
- Hash SHA-256 du contenu.

Toute modification du modèle de scoring ou du template entraîne :
- Incrément de version.
- Conservation des anciens rapports intacts.

---

## 10. Internationalisation

- Support natif FR/EN.
- Fichiers de traduction externes.
- Formatage localisé des dates et nombres.

---

## 11. Contraintes non fonctionnelles

- Déterminisme : mêmes données + mêmes paramètres = fichier identique (hors timestamp).
- Scalabilité horizontale en SaaS.
- Compatible déploiement on-premise sans dépendance cloud obligatoire.
- Aucune fuite de données entre tenants.

---

## 12. API associée

### 12.1 Endpoint génération

POST /comparisons/{id}/exports

Body :
- format
- parameters

Response :
- exportId
- status

### 12.2 Endpoint récupération

GET /exports/{exportId}

Contrôle d’accès obligatoire.

---

## 13. Tests techniques requis

- Tests unitaires : génération modèle intermédiaire.
- Tests d’intégration : rendu multi-format.
- Tests de charge : génération simultanée (≥ 50 exports concurrents).
- Tests de non-régression sur scoringModelVersion.
- Vérification hash reproductible.

---

Fin de spécification technique – Module Rapports et exports.
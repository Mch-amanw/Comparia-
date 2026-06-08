# Module : Rapports et exports

## 1. Rôle du module
Le module **Rapports et exports** permet de générer, formater et exporter les résultats d’une comparaison de documents réalisée dans Comparia. 

Il transforme les données de comparaison (différences détectées, score de similarité, validations, commentaires, métadonnées) en livrables exploitables et partageables par les utilisateurs, dans des formats standards (PDF, Word).

Ce module contribue à :
- La diffusion des résultats auprès de tiers (clients, juristes, directions),
- L’archivage formel d’une comparaison,
- La traçabilité et la conformité (audit, validation),
- L’intégration avec d’autres outils via exports structurés (évolution future : Excel, JSON).

---

## 2. Périmètre fonctionnel

### 2.1 Génération de rapport de comparaison
Pour chaque comparaison finalisée, l’utilisateur autorisé peut générer un rapport contenant :

- Informations générales :
  - Nom des documents comparés
  - Date et heure de la comparaison
  - Auteur de la comparaison
  - Version du modèle de scoring (ex : scoring v1)
- Score global de similarité (obligatoire)
- Sous-scores (ST, SH, SB, SS) si disponibles
- Liste structurée des différences détectées :
  - Ajouts
  - Suppressions
  - Modifications
  - Sections déplacées
- Indication visuelle des différences (code couleur ou légende textuelle)
- Commentaires associés aux différences (si activés)
- Statut des différences (validé, refusé, en attente)

Règle :
- Toute comparaison doit pouvoir donner lieu à un rapport exportable.

---

### 2.2 Formats d’export supportés

#### 2.2.1 Export PDF
- Format figé, prêt à être partagé.
- Intègre le score global de similarité en première page.
- Mise en forme structurée (sections, sommaire si pertinent).
- Compatible avec l’archivage documentaire.

#### 2.2.2 Export Word (.docx)
- Rapport modifiable.
- Structuration en titres hiérarchiques.
- Conservation des tableaux et sections sous forme éditable.

#### 2.2.3 Évolutions prévues
- Export Excel (extraction tabulaire des différences).
- Export JSON (format structuré pour intégration API).

Ces formats doivent respecter les droits d’accès de l’utilisateur.

---

### 2.3 Personnalisation du rapport

#### 2.3.1 Sélection du contenu
L’utilisateur peut, selon ses droits :
- Inclure ou exclure les commentaires.
- Inclure uniquement les différences validées.
- Choisir un niveau de détail (résumé / détaillé).

#### 2.3.2 Marque blanche
Dans le cadre des clients disposant de la fonctionnalité marque blanche :
- Logo personnalisé.
- Nom de l’organisation.
- Couleurs principales.

Les paramètres de branding sont définis au niveau du tenant.

---

### 2.4 Gestion des droits
- Seuls les utilisateurs ayant accès à la comparaison peuvent générer ou télécharger un rapport.
- Les exports respectent les droits d’accès par document.
- Toute génération ou téléchargement de rapport est journalisé (module d’audit).

---

### 2.5 Mode de stockage
Selon la configuration client :
- Mode suppression automatique : le rapport généré n’est pas conservé après téléchargement.
- Mode archivage : le rapport est stocké avec la comparaison et consultable ultérieurement.

---

### 2.6 Performance
- La génération d’un rapport standard doit être quasi immédiate pour des documents jusqu’à 300 pages.
- Si nécessaire (rapport volumineux), la génération peut être asynchrone avec notification de disponibilité.

---

## 3. Règles de gestion

1. Le score affiché dans le rapport doit correspondre strictement au modèle de scoring utilisé (versionné).
2. Les différences marquées comme "déplacées" doivent être explicitement identifiées.
3. Le rapport doit être cohérent avec la vue affichée dans l’interface au moment de la génération.
4. Toute génération ou export déclenche une entrée dans le journal d’audit.
5. En cas de modification future du modèle de scoring, les anciens rapports restent associés à leur version de scoring.

---

# technicalSpec

## 1. Positionnement dans l’architecture

Le module Rapports et exports s’intègre comme un service applicatif distinct :

- Consomme les données issues du service de comparaison.
- Interagit avec le module d’authentification/autorisation pour contrôle d’accès.
- Interagit avec le module d’audit pour journalisation.
- Utilise le service de stockage selon la configuration client.

Il expose des endpoints via l’API REST du backend.

---

## 2. Flux technique de génération

### 2.1 Étapes
1. Requête utilisateur via frontend.
2. Vérification des droits d’accès.
3. Récupération des données de comparaison :
   - Métadonnées
   - Résultats détaillés
   - Scores (global + sous-scores)
   - Commentaires et statuts
4. Construction d’un modèle de données interne de rapport.
5. Génération du document dans le format demandé.
6. Stockage ou transmission directe au client.
7. Écriture d’un événement d’audit.

---

## 3. Génération de documents

### 3.1 Moteur de templating
- Utilisation de templates structurés pour PDF et Word.
- Templates paramétrables pour prise en charge de la marque blanche.
- Séparation claire entre :
  - Données (JSON interne)
  - Présentation (template).

### 3.2 Export PDF
- Génération via moteur de rendu HTML → PDF ou bibliothèque dédiée.
- Support des tableaux, sections, légendes, styles.

### 3.3 Export Word (.docx)
- Génération via bibliothèque compatible OpenXML.
- Structuration en niveaux de titres cohérents avec la hiérarchie documentaire.

---

## 4. Modèle de données du rapport

Le modèle interne inclut :
- comparisonId
- tenantId
- scoringVersion
- globalScore
- subScores (ST, SH, SB, SS)
- differences[] :
  - type (ajout, suppression, modification, deplacement)
  - sectionId
  - contenuAvant
  - contenuApres
  - statutValidation
  - commentaires[]
- metadata (date, auteur, documents concernés)

Ce modèle doit être stable et versionné pour garantir la cohérence des exports.

---

## 5. Performance et scalabilité

- Génération synchrone par défaut pour rapports standards.
- Génération asynchrone via file de tâches pour rapports volumineux.
- Scalabilité horizontale en environnement SaaS.
- Optimisation mémoire pour éviter le chargement complet de très gros documents en RAM.

---

## 6. Sécurité

- Vérification systématique des droits avant génération.
- Chiffrement des rapports stockés au repos (si archivage activé).
- Transmission sécurisée (HTTPS).
- Aucun accès direct aux fichiers sans contrôle d’autorisation.

---

## 7. Journalisation et audit

Pour chaque génération/export :
- userId
- comparisonId
- format exporté
- date/heure
- mode (téléchargement direct ou archivage)

Les logs sont transmis au module d’audit central.

---

## 8. Versioning

- Le rapport inclut explicitement la version du modèle de scoring.
- Toute évolution majeure du format de rapport entraîne une version interne du template.
- Les rapports archivés restent associés à leur version de génération.

---
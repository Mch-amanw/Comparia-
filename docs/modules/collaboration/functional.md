# Module : Collaboration

## 1. Rôle du module
Le module Collaboration permet aux utilisateurs de travailler à plusieurs sur une comparaison de documents. Il offre des fonctionnalités de commentaires, de validation ou de refus des différences détectées, ainsi que le partage sécurisé d’un lien de comparaison.

Il s’intègre au moteur de comparaison, au module de gestion des utilisateurs et rôles, ainsi qu’au module d’audit. Il contribue à faire de Comparia un outil d’analyse, de revue et de validation documentaire adapté aux environnements professionnels (juridique, administratif, entreprise).

---

## 2. Périmètre fonctionnel

### 2.1 Commentaires sur les différences

#### 2.1.1 Ajout de commentaire
Un utilisateur autorisé peut :
- Ajouter un commentaire sur une différence détectée (ajout, suppression, modification, déplacement).
- Associer le commentaire à :
  - Une différence précise,
  - Une section,
  - Ou la comparaison globale.

Chaque commentaire contient :
- Auteur (utilisateur authentifié),
- Date/heure de création,
- Contenu textuel (multilingue FR/EN),
- Statut (actif, résolu).

#### 2.1.2 Réponses et fil de discussion
- Possibilité de répondre à un commentaire existant.
- Affichage sous forme de fil (thread).
- Historique conservé (pas de suppression définitive visible côté utilisateur standard).

#### 2.1.3 Résolution
- Un commentaire peut être marqué comme « résolu ».
- La résolution est traçable (qui, quand).
- Les commentaires résolus restent consultables.

---

### 2.2 Validation des différences

#### 2.2.1 Statut des différences
Chaque différence détectée peut avoir un statut :
- À valider (par défaut),
- Validée,
- Refusée.

#### 2.2.2 Action de validation
Un utilisateur disposant des droits adéquats peut :
- Valider une différence (acceptation du changement),
- Refuser une différence (changement jugé non acceptable ou incorrect).

Chaque action enregistre :
- L’utilisateur,
- La date/heure,
- Le statut choisi.

#### 2.2.3 Vue synthétique
L’interface doit permettre :
- De filtrer les différences par statut,
- De visualiser un récapitulatif :
  - Nombre total de différences,
  - Nombre validées,
  - Nombre refusées,
  - Nombre restantes.

---

### 2.3 Partage sécurisé de comparaisons

#### 2.3.1 Génération de lien sécurisé
Un utilisateur autorisé peut générer un lien de partage permettant :
- La consultation d’une comparaison,
- Selon configuration : consultation seule ou avec commentaires.

Le lien doit être :
- Unique,
- Non prédictible,
- Associé à une comparaison précise.

#### 2.3.2 Paramètres de partage
Lors de la génération du lien :
- Définition d’une date d’expiration (optionnelle selon politique client),
- Définition d’un niveau d’accès (lecture seule / lecture + commentaire).

Les droits effectifs restent contraints par la configuration du tenant et les règles de sécurité globales.

#### 2.3.3 Révocation
- Le créateur ou un administrateur peut révoquer un lien.
- Toute tentative d’accès après expiration ou révocation est refusée.

---

## 3. Règles de gestion

1. Les droits d’accès conditionnent l’ajout de commentaires, la validation et le partage.
2. Toute action (commentaire, validation, partage, révocation) doit être journalisée dans le module d’audit.
3. Une comparaison supprimée (selon politique de stockage) rend inaccessibles ses commentaires et liens associés.
4. Les actions de validation n’altèrent pas le score de similarité, mais peuvent être utilisées dans les rapports exportés.
5. En mode archivage, l’historique des commentaires et validations est conservé.

---

## 4. Dépendances

- Module Authentification & Gestion des rôles (contrôle d’accès).
- Module Comparaison (référentiel des différences et sections).
- Module Audit (traçabilité des actions).
- Module Rapport (intégration des commentaires et statuts dans les exports).

---

## 5. Cas d’usage principaux

1. Un juriste compare deux versions d’un contrat, commente plusieurs modifications et valide les changements acceptés.
2. Un manager partage un lien de consultation à un client en lecture seule.
3. Une équipe traite les différences une à une et suit l’avancement via le tableau de synthèse.

---

# technicalSpec

## 1. Architecture du module

Le module Collaboration est implémenté côté backend via l’API REST et exposé au frontend pour interaction en temps réel ou quasi temps réel.

Composants principaux :
- Service de gestion des commentaires,
- Service de gestion des statuts de différences,
- Service de gestion des liens de partage,
- Intégration avec le module d’audit.

---

## 2. Modèle de données (conceptuel)

### 2.1 Comment
- id
- comparisonId
- differenceId (nullable)
- parentCommentId (nullable)
- authorId
- content
- status (active, resolved)
- createdAt
- resolvedAt (nullable)
- resolvedBy (nullable)

### 2.2 DifferenceReviewStatus
- id
- comparisonId
- differenceId
- status (pending, validated, rejected)
- updatedBy
- updatedAt

### 2.3 ShareLink
- id
- comparisonId
- token (aléatoire, non prédictible)
- accessLevel (read, comment)
- expiresAt (nullable)
- revoked (bool)
- createdBy
- createdAt

Les relations doivent garantir l’intégrité référentielle avec les entités Comparison et User.

---

## 3. API REST (principes)

### 3.1 Commentaires
- POST /comparisons/{id}/comments
- GET /comparisons/{id}/comments
- POST /comments/{id}/resolve
- POST /comments/{id}/reply

### 3.2 Validation des différences
- POST /comparisons/{id}/differences/{diffId}/status
- GET /comparisons/{id}/differences/status-summary

### 3.3 Partage
- POST /comparisons/{id}/share-links
- GET /share/{token}
- POST /share-links/{id}/revoke

Toutes les routes sont protégées par authentification, sauf l’accès via token sécurisé (contrôlé par validité et droits associés).

---

## 4. Sécurité

- Vérification systématique des permissions par document et par comparaison.
- Token de partage généré via un mécanisme cryptographiquement sûr.
- Chiffrement des données en transit (HTTPS obligatoire).
- Journalisation automatique des actions critiques.
- Respect de la politique de rétention définie par le mode de stockage (suppression ou archivage).

---

## 5. Performance et contraintes

- Les opérations de commentaires et validation doivent être légères et ne pas impacter le temps de calcul de la comparaison.
- Les requêtes doivent être paginées pour les comparaisons avec grand nombre de différences.
- Conçu pour supporter 100 à 500 comparaisons par jour avec usage collaboratif simultané.

---

## 6. Intégration aux rapports

Le module expose des données structurées permettant au module Rapport de :
- Inclure les commentaires (optionnellement),
- Inclure le statut de chaque différence,
- Générer un récapitulatif des validations.

---
# Fonctionnalité : Commentaires et validation des différences

## 1. Objectif métier

Cette fonctionnalité permet aux utilisateurs autorisés de :
- Ajouter des commentaires sur les différences détectées lors d’une comparaison,
- Échanger via des fils de discussion contextualisés,
- Valider ou refuser individuellement chaque différence,
- Suivre l’avancement de la revue d’une comparaison.

Elle transforme Comparia en véritable outil de revue collaborative, notamment pour les équipes juridiques, administratives et métiers, en facilitant la prise de décision sur les écarts détectés.

---

## 2. Périmètre fonctionnel

### 2.1 Commentaires sur les différences

#### 2.1.1 Ajout de commentaire
Un utilisateur authentifié et disposant des droits nécessaires peut :
- Ajouter un commentaire associé :
  - À une différence spécifique (ajout, suppression, modification, déplacement),
  - À une section,
  - Ou à la comparaison globale.

Chaque commentaire contient :
- Auteur (utilisateur identifié),
- Date et heure de création,
- Contenu textuel (FR/EN),
- Statut (actif, résolu).

Les commentaires sont visibles dans l’interface de comparaison (vue côte à côte), contextualisés par rapport à la différence concernée.

#### 2.1.2 Réponses et fil de discussion
- Un commentaire peut recevoir des réponses (thread).
- Les réponses sont hiérarchisées sous le commentaire parent.
- L’historique est conservé (pas de suppression définitive visible pour un utilisateur standard).

#### 2.1.3 Résolution de commentaire
- Un commentaire peut être marqué comme « résolu ».
- La résolution enregistre :
  - L’utilisateur ayant effectué l’action,
  - La date/heure.
- Les commentaires résolus restent consultables.

---

### 2.2 Validation des différences

#### 2.2.1 Statut des différences
Chaque différence détectée possède un statut de revue :
- pending (À valider) – statut par défaut,
- validated (Validée),
- rejected (Refusée),
- needsReview (si activé selon configuration module).

#### 2.2.2 Action de validation
Un utilisateur disposant des droits adéquats peut :
- Valider une différence (acceptation du changement),
- Refuser une différence.

Chaque action enregistre :
- L’utilisateur,
- La date/heure,
- Le nouveau statut.

La validation :
- N’altère pas le score de similarité,
- Peut être intégrée aux rapports exportés.

#### 2.2.3 Vue synthétique d’avancement
L’interface doit fournir :
- Un résumé du nombre total de différences,
- Le nombre validées,
- Le nombre refusées,
- Le nombre restantes (pending / needsReview),
- Des filtres par statut.

---

## 3. Règles de gestion

1. Les droits d’accès déterminent la possibilité d’ajouter des commentaires ou de modifier le statut d’une différence.
2. Les actions de commentaire et de validation sont journalisées dans le module d’audit.
3. Les commentaires et statuts sont liés à une version spécifique de la comparaison.
4. En cas de suppression d’une comparaison (selon politique de stockage), les commentaires et statuts associés deviennent inaccessibles.
5. En mode archivage, l’historique des commentaires et validations est conservé.
6. La validation d’une différence n’impacte ni le score global ni les sous-scores (ST, SH, SB, SS).

---

## 4. Critères d’acceptation

- Un utilisateur autorisé peut créer un commentaire sur une différence précise.
- Les réponses apparaissent correctement dans un fil hiérarchisé.
- Un commentaire peut être marqué comme résolu et reste consultable.
- Un utilisateur autorisé peut changer le statut d’une différence.
- Le résumé des statuts reflète fidèlement l’état réel des différences.
- Toute action (création, modification, résolution, validation) génère un événement d’audit.
- Les commentaires et statuts disparaissent de l’interface si la comparaison est supprimée.

---

## 5. Dépendances

- Module Comparaison (référentiel des différences).
- Module Authentification & Rôles (contrôle d’accès).
- Module Audit (traçabilité).
- Module Rapport (intégration des commentaires et statuts dans les exports).
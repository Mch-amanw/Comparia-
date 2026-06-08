## Ticket : Ajouter système de commentaires par différence

### 1. Objectif
Mettre en place le système permettant aux utilisateurs autorisés d’ajouter, modifier et supprimer (suppression logique) des commentaires associés à une différence spécifique détectée lors d’une comparaison.

Ce ticket couvre uniquement les commentaires liés à une différence (diffId non nul). Les commentaires globaux ou par section sont hors périmètre de ce ticket.

---

### 2. Périmètre fonctionnel

#### 2.1 Ajout de commentaire sur une différence

Un utilisateur authentifié et disposant des droits nécessaires peut :

- Ajouter un commentaire rattaché à :
  - Une comparaison donnée,
  - Une différence précise identifiée par son `diffId`.

Chaque commentaire doit contenir :
- Auteur (utilisateur connecté),
- Date et heure de création,
- Contenu textuel (FR/EN),
- Statut initial : `active`.

Le commentaire est affiché dans l’interface de comparaison, contextualisé visuellement sur la différence correspondante.

---

#### 2.2 Modification d’un commentaire

Un utilisateur peut modifier le contenu d’un commentaire existant si :
- Il en est l’auteur ou
- Il dispose d’un rôle autorisé (ex : administrateur selon règles RBAC globales).

La modification :
- Met à jour le contenu textuel,
- Met à jour le champ `updatedAt`,
- Ne supprime pas l’historique d’audit.

Le commentaire reste lié à la même différence.

---

#### 2.3 Suppression d’un commentaire (soft delete)

Un utilisateur autorisé peut supprimer un commentaire.

La suppression :
- Est logique (soft delete),
- Rend le commentaire non visible dans l’interface standard,
- Conserve la trace dans la base et dans le module d’audit.

Un commentaire supprimé :
- N’est plus affiché dans la liste des commentaires d’une différence,
- Reste traçable côté audit.

---

### 3. Règles de gestion

1. Un commentaire doit obligatoirement être rattaché à un `comparisonId` et un `diffId` valides.
2. Les droits d’accès sont vérifiés au niveau de la comparaison.
3. Les commentaires sont liés à une version spécifique de comparaison (via `comparisonId`).
4. La suppression d’une comparaison rend inaccessibles les commentaires associés.
5. Toute action (création, modification, suppression) doit générer un événement dans le module d’audit.
6. Les opérations sur les commentaires n’impactent pas :
   - Le score global de similarité,
   - Les sous-scores (ST, SH, SB, SS),
   - Le statut de validation des différences.

---

### 4. Dépendances

- Module Comparaison (référentiel des différences via `diffId`).
- Module Authentification & Rôles (contrôle RBAC).
- Module Audit (journalisation des actions).
- Frontend de comparaison (affichage contextualisé).
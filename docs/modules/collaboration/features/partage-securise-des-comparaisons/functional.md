## Fonctionnalité : Partage sécurisé des comparaisons

### 1. Objectif métier
Permettre à un utilisateur autorisé de partager une comparaison de documents via un lien sécurisé, tout en garantissant un contrôle strict des accès, la confidentialité des données et la traçabilité des actions.

Cette fonctionnalité répond au besoin courant de transmettre les résultats d’une comparaison (contrat, version révisée, document administratif) à un tiers interne ou externe sans exposer l’ensemble du système.

---

### 2. Périmètre fonctionnel

#### 2.1 Génération d’un lien de partage
Un utilisateur disposant des droits adéquats peut :
- Générer un lien unique associé à une comparaison spécifique.
- Définir les paramètres d’accès au moment de la création.

Chaque lien :
- Est rattaché à une seule comparaison.
- Est unique et non prédictible.
- Peut être multiple pour une même comparaison (plusieurs liens possibles).

---

#### 2.2 Paramétrage du lien
Lors de la création, l’utilisateur peut configurer :

- **Niveau d’accès** :
  - Lecture seule (consultation des documents et différences).
  - Lecture + commentaires.
  - Lecture + commentaires + validation (si autorisé par la politique du tenant).

- **Date d’expiration** (optionnelle) :
  - Date et heure précises.
  - Si atteinte, le lien devient automatiquement invalide.

- **Restrictions supplémentaires (si activées côté tenant)** :
  - Mot de passe d’accès.
  - Restriction à un domaine email spécifique.

Les options disponibles peuvent être restreintes par la configuration du tenant.

---

#### 2.3 Accès via lien
Un utilisateur accédant au lien :
- Peut consulter uniquement la comparaison associée.
- Ne peut pas naviguer vers d’autres comparaisons.
- Est limité au niveau d’accès défini par le lien.

En cas :
- D’expiration,
- De révocation,
- De comparaison supprimée (selon politique de stockage),

→ L’accès est refusé avec un message explicite.

---

#### 2.4 Révocation du lien
Un lien peut être révoqué :
- Par son créateur.
- Par un administrateur disposant des droits nécessaires.

La révocation :
- Est immédiate.
- Rend le lien définitivement invalide.
- Est journalisée.

---

#### 2.5 Gestion et visualisation des liens
Depuis l’interface de la comparaison, l’utilisateur autorisé peut :
- Voir la liste des liens actifs et expirés.
- Visualiser pour chaque lien :
  - Date de création,
  - Créateur,
  - Niveau d’accès,
  - Date d’expiration,
  - Statut (actif, expiré, révoqué).
- Révoquer un lien actif.

---

### 3. Règles de gestion

1. La création d’un lien est soumise aux droits d’accès sur la comparaison.
2. Un lien ne peut donner plus de droits que ceux du créateur.
3. Toute création, consultation, révocation ou tentative d’accès invalide est journalisée.
4. Si la comparaison est supprimée (mode suppression automatique), tous les liens associés deviennent inaccessibles.
5. Les actions réalisées via un lien (commentaire, validation) sont tracées avec un identifiant utilisateur (authentifié ou session liée au lien selon configuration).
6. En environnement multi-tenant, un lien est strictement isolé dans le tenant d’origine.

---

### 4. Critères d’acceptation

- Un lien généré permet d’accéder uniquement à la comparaison ciblée.
- Un lien expiré ou révoqué retourne une erreur d’accès claire.
- Le niveau d’accès défini est strictement respecté.
- La création et la révocation sont visibles dans le journal d’audit.
- Les liens sont impossibles à deviner (test de robustesse sur entropie minimale).
- La suppression d’une comparaison invalide automatiquement ses liens.
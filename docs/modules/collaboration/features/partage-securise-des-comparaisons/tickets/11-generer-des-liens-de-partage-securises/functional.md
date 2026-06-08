## Ticket : Générer des liens de partage sécurisés

### 1. Objectif
Mettre en place le mécanisme permettant à un utilisateur autorisé de générer un lien de partage sécurisé pour une comparaison donnée, avec contrôle strict du niveau d’accès et des paramètres de sécurité.

Ce ticket couvre :
- La création d’un lien unique associé à une comparaison.
- La configuration des paramètres d’accès lors de la génération.
- La restitution de l’URL sécurisée à l’utilisateur.

La gestion détaillée de l’accès via token et la révocation sont couvertes par des tickets complémentaires, mais doivent être compatibles avec cette implémentation.

---

### 2. Périmètre fonctionnel

#### 2.1 Conditions préalables
- L’utilisateur est authentifié.
- L’utilisateur dispose des droits suffisants sur la comparaison concernée.
- La comparaison existe et appartient au tenant de l’utilisateur.

---

### 3. Génération d’un lien de partage

#### 3.1 Action utilisateur
Depuis l’interface d’une comparaison, un utilisateur autorisé peut :
- Cliquer sur « Générer un lien de partage ».
- Configurer les paramètres du lien.
- Valider la création.

À l’issue de la création, le système affiche :
- L’URL complète contenant le token sécurisé.
- Les paramètres choisis (niveau d’accès, expiration, etc.).

---

### 4. Paramètres configurables

Lors de la génération, l’utilisateur peut définir :

#### 4.1 Niveau d’accès
Selon la configuration du tenant et les droits du créateur :
- Lecture seule.
- Lecture + commentaires.
- Lecture + commentaires + validation.

Le système ne doit jamais permettre de générer un lien avec un niveau d’accès supérieur aux droits du créateur.

#### 4.2 Date d’expiration (optionnelle)
- Date et heure précises.
- Si non définie, le lien est considéré sans expiration (sauf politique globale du tenant).

#### 4.3 Mot de passe (optionnel)
- Si activé, l’accès via le lien nécessitera la saisie d’un mot de passe.
- Le mot de passe n’est jamais affiché en clair après création.

#### 4.4 Restriction de domaine (optionnelle)
- Limitation de l’accès aux utilisateurs dont l’email correspond à un domaine spécifique.
- Applicable uniquement si la configuration du tenant l’autorise.

---

### 5. Règles de gestion

1. Un lien est toujours rattaché à une seule comparaison.
2. Plusieurs liens peuvent être générés pour une même comparaison.
3. Un lien généré est unique et non prédictible.
4. La création du lien est journalisée (créateur, date, paramètres principaux).
5. Le lien appartient strictement au tenant de la comparaison.
6. Si la comparaison est ultérieurement supprimée (mode suppression automatique), le lien devient inopérant.

---

### 6. Contraintes UX

- L’URL doit pouvoir être copiée facilement (bouton « Copier »).
- Un message de confirmation clair doit indiquer que le lien a été généré avec succès.
- Les paramètres sélectionnés doivent être visibles après création.
- Les options indisponibles (selon rôle ou politique tenant) doivent être masquées ou désactivées.

---

### 7. Intégration aux modules existants

- Contrôle des droits via module Authentification & RBAC.
- Persistance via module Collaboration.
- Journalisation via module Audit.
- Isolation stricte en environnement multi-tenant.

---

Ce ticket constitue la base du mécanisme de partage sécurisé et doit être conforme aux exigences globales de sécurité et de confidentialité du projet Comparia.
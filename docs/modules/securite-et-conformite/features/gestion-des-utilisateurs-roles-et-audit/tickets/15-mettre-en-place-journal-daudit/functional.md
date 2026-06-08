## Ticket : Mettre en place le journal d’audit

### 1. Objectif
Mettre en place un mécanisme centralisé de journal d’audit permettant d’enregistrer toutes les actions critiques effectuées sur la plateforme Comparia, avec traçabilité complète (qui, quoi, quand, sur quelle ressource, avec quel résultat).

Ce journal d’audit constitue un pilier de sécurité, de conformité (RGPD) et de gouvernance pour les clients SaaS et on-premise.

---

### 2. Périmètre fonctionnel

Le ticket couvre :
- L’enregistrement automatique des événements métier critiques.
- L’association systématique d’un utilisateur et d’un tenant à chaque événement.
- L’horodatage précis de chaque action.
- La consultation des logs par les rôles autorisés.
- L’interdiction de modification ou suppression des logs par les utilisateurs standards.

Il ne couvre pas :
- L’export avancé des logs (évolution ultérieure).
- L’intégration avec des outils SIEM externes (évolution ultérieure).

---

### 3. Événements à journaliser (minimum requis)

Les événements suivants doivent être enregistrés :

#### 3.1 Authentification
- Tentative de connexion (succès / échec)
- Déconnexion

#### 3.2 Gestion des utilisateurs
- Création de compte
- Modification de compte
- Désactivation
- Suppression
- Modification des rôles

#### 3.3 Documents et comparaisons
- Téléversement de document
- Lancement d’une comparaison
- Consultation d’une comparaison
- Suppression d’un document

#### 3.4 Collaboration
- Ajout de commentaire
- Validation ou refus de différences

#### 3.5 Rapports
- Export de rapport

---

### 4. Données enregistrées pour chaque événement

Chaque entrée du journal d’audit doit contenir au minimum :

- Identifiant unique de l’événement
- Identifiant utilisateur (si action authentifiée)
- Identifiant tenant
- Date et heure (horodatage serveur)
- Type d’action (code normalisé)
- Type de ressource concernée (Document, Comparison, User, Report, etc.)
- Identifiant de la ressource
- Résultat de l’action (succès / échec)

Les données doivent être suffisamment explicites pour permettre un audit ultérieur sans ambiguïté.

---

### 5. Règles de gestion

- Toute action critique doit déclencher automatiquement une écriture dans le journal d’audit.
- L’écriture dans le journal ne doit pas être optionnelle.
- Les utilisateurs standards ne peuvent ni modifier ni supprimer les logs.
- Seuls les rôles disposant de la permission dédiée peuvent consulter les logs.
- Les logs sont isolés par tenant en mode SaaS.
- La durée de conservation des logs dépend de la configuration client.

---

### 6. Consultation des logs

Un mécanisme de consultation doit permettre :
- Filtrage par période (date de début / date de fin)
- Filtrage par utilisateur
- Filtrage par type d’action

La consultation doit respecter strictement les règles RBAC et l’isolation multi-tenant.

---

### 7. Contraintes

- Le journal d’audit ne doit pas dégrader significativement les performances globales.
- Les comparaisons doivent rester dans l’objectif < 1 minute.
- Le mécanisme doit fonctionner en SaaS et en on-premise.

---

### 8. Dépendances

- Middleware d’authentification et d’autorisation (pour récupérer l’utilisateur et le tenant).
- Modules Comparaison, OCR, Rapports et Collaboration (pour émettre les événements).
- Module Stockage (pour politique de rétention).

Ce ticket pose les fondations opérationnelles de la traçabilité complète de la plateforme Comparia.
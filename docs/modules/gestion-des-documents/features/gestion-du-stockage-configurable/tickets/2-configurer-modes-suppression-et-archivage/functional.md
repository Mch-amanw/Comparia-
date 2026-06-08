## Ticket : Configurer modes suppression et archivage

### 1. Objectif

Permettre la configuration, au niveau de chaque tenant (client), du mode de stockage des documents entre :

- **Suppression automatique** des documents après traitement ou délai configuré
- **Archivage persistant** des documents et résultats de comparaison

Cette configuration conditionne le comportement du cycle de vie des documents et doit être administrable de manière sécurisée et traçable.

---

### 2. Périmètre fonctionnel du ticket

Ce ticket couvre :

- La mise en place du paramétrage du mode de stockage au niveau du tenant.
- La possibilité pour un administrateur tenant de modifier ce paramétrage.
- L’enregistrement et la persistance de cette configuration.
- La journalisation de toute modification.

Ce ticket ne couvre pas :

- L’implémentation complète du mécanisme de suppression automatique (traité dans un ticket dédié).
- L’implémentation des workers de suppression asynchrone.

---

### 3. Configuration du mode de stockage

#### 3.1 Paramètres configurables

Pour chaque tenant, les paramètres suivants doivent être configurables :

- `storageMode` :
  - `SUPPRESSION_AUTOMATIQUE`
  - `ARCHIVAGE`
- `retentionDelay` (optionnel) :
  - Délai de conservation avant suppression automatique (exprimé en heures ou jours selon convention technique retenue).
  - Applicable uniquement si `storageMode = SUPPRESSION_AUTOMATIQUE`.

En mode `ARCHIVAGE`, le champ `retentionDelay` est ignoré.

---

### 4. Gestion des droits

- Seuls les utilisateurs disposant d’un rôle d’administration du tenant peuvent :
  - Consulter la configuration actuelle.
  - Modifier le mode de stockage.
  - Modifier le délai de rétention.

- Les utilisateurs non administrateurs ne doivent pas pouvoir modifier cette configuration.

---

### 5. Effets métier attendus

- Le mode configuré devient la règle de référence pour tous les documents importés sous le tenant.
- Le changement de configuration :
  - N’a pas d’effet rétroactif automatique sur les documents déjà supprimés.
  - Est pris en compte pour les nouveaux documents et pour les traitements futurs.

Toute modification doit être tracée dans le journal d’audit avec :
- Identifiant du tenant
- Identifiant de l’utilisateur administrateur
- Ancienne valeur
- Nouvelle valeur
- Date/heure de modification

---

### 6. Contraintes fonctionnelles

- La configuration est strictement isolée par tenant.
- Elle doit être persistante et disponible pour les autres modules (notamment Gestion des documents).
- Elle ne doit pas impacter les performances globales du système.
- Elle doit être compatible avec les déploiements SaaS et on-premise.

---

### 7. Dépendances

- Module Gestion des documents (cycle de vie et stockage).
- Module Authentification & Gestion des rôles (contrôle d’accès administrateur).
- Module Audit & Journalisation (traçabilité des changements).
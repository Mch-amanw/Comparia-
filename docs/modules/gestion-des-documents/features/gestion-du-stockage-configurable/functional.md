# Fonctionnalité : Gestion du stockage configurable

## 1. Objectif métier

Permettre à chaque client (tenant) de configurer le mode de stockage des documents et des résultats de comparaison, afin de répondre à ses contraintes réglementaires, contractuelles et de confidentialité.

Deux modes principaux sont proposés :
- **Suppression automatique après traitement**
- **Archivage des documents et comparaisons**

Cette fonctionnalité garantit que le comportement du système est conforme à la politique de stockage définie par le client, tout en respectant les exigences de sécurité, d’audit et de performance du projet Comparia.

---

## 2. Périmètre fonctionnel

### 2.1 Configuration par tenant

Le mode de stockage est défini au niveau du tenant (client) et s’applique à l’ensemble des documents importés dans son périmètre.

La configuration comprend :
- Mode sélectionné :
  - `SUPPRESSION_AUTOMATIQUE`
  - `ARCHIVAGE`
- Option de délai de rétention (si applicable en mode suppression automatique).

Seuls les utilisateurs disposant des droits d’administration du tenant peuvent modifier cette configuration.

Toute modification de configuration doit être journalisée.

---

### 2.2 Mode 1 : Suppression automatique

#### 2.2.1 Principe
Les documents sources sont supprimés automatiquement :
- Après traitement complet (extraction, OCR, normalisation),
- Ou après un délai configuré,
- Ou après génération des résultats de comparaison.

Le comportement exact dépend de la configuration retenue pour le tenant.

#### 2.2.2 Effets fonctionnels
- Les fichiers originaux ne sont plus accessibles après suppression.
- Les représentations normalisées peuvent être conservées temporairement si nécessaires au calcul du score ou à l’affichage.
- Les résultats de comparaison peuvent être conservés ou non selon la configuration globale.

Une fois supprimé :
- Le document passe au statut `DELETED`.
- Il n’est plus visible ni sélectionnable pour une nouvelle comparaison.

---

### 2.3 Mode 2 : Archivage

#### 2.3.1 Principe
Les documents originaux et leurs représentations normalisées sont conservés.

Les comparaisons associées sont également conservées, permettant :
- La reconsultation des résultats,
- La génération ultérieure de nouveaux rapports,
- De nouvelles comparaisons avec d’autres versions.

#### 2.3.2 Effets fonctionnels
- Les documents conservent le statut `ARCHIVED` ou `READY` selon leur état.
- Ils restent accessibles selon les droits utilisateur.
- Ils peuvent être supprimés manuellement par un utilisateur autorisé.

---

### 2.4 Impact sur le cycle de vie du document

Le comportement du cycle de vie dépend du mode choisi :

1. Import
2. Validation
3. Traitement (extraction/OCR)
4. Mise à disposition pour comparaison
5. 
   - Mode suppression : suppression automatique → statut `DELETED`
   - Mode archivage : conservation → statut `READY` ou `ARCHIVED`

Toute transition critique (suppression automatique, suppression manuelle, changement de mode) doit être journalisée.

---

## 3. Règles de gestion

- La configuration est strictement isolée par tenant.
- Un changement de mode n’a pas d’effet rétroactif automatique sur les documents déjà supprimés.
- Les documents supprimés physiquement ne peuvent pas être restaurés.
- Les droits d’accès continuent de s’appliquer tant que le document est conservé.
- La suppression automatique doit être effective (suppression physique du stockage) afin de respecter les exigences RGPD.
- Les journaux d’audit liés aux actions restent conservés indépendamment du mode choisi.

---

## 4. Critères d’acceptation

1. Un administrateur peut sélectionner le mode de stockage au niveau du tenant.
2. En mode suppression automatique, un document importé est supprimé conformément à la configuration définie.
3. En mode archivage, un document reste accessible après comparaison.
4. Un document supprimé automatiquement n’est plus visible dans l’interface utilisateur.
5. Toutes les suppressions automatiques sont tracées dans le journal d’audit.
6. Le changement de configuration est journalisé avec l’identifiant du tenant et de l’utilisateur administrateur.

---

## 5. Dépendances

- Module Gestion des documents (cycle de vie, statuts, stockage).
- Module Authentification & Gestion des rôles (droits administrateur).
- Module Audit & Journalisation (traçabilité des suppressions et changements de configuration).
- Infrastructure de stockage (SaaS ou on-premise).
## TICKET – Développer connecteur SharePoint et Google Drive

### 1. Objectif

Permettre aux utilisateurs autorisés d’importer directement des documents depuis **SharePoint** et **Google Drive** dans Comparia afin de lancer une comparaison documentaire sans téléchargement manuel préalable.

Ce ticket couvre :
- L’authentification sécurisée aux plateformes.
- La navigation dans les arborescences accessibles.
- La sélection de documents compatibles.
- L’import du flux binaire vers Comparia.
- Le déclenchement d’une comparaison.
- La journalisation complète des opérations.

L’export vers ces plateformes n’est pas inclus dans ce ticket (évolution future).

---

### 2. Périmètre fonctionnel

#### 2.1 Authentification utilisateur

L’utilisateur doit pouvoir :
- Se connecter à SharePoint via le mécanisme d’authentification standard (OAuth 2.0).
- Se connecter à Google Drive via son compte Google (OAuth 2.0 avec consentement explicite).

Règles de gestion :
- Les permissions existantes sur la plateforme externe doivent être strictement respectées.
- Aucune élévation de privilège n’est autorisée.
- Les tokens d’accès sont liés à l’utilisateur et à son tenant.
- En cas de révocation du consentement, l’accès est invalidé côté Comparia.

---

#### 2.2 Navigation dans l’arborescence

L’utilisateur peut :
- Parcourir les bibliothèques et dossiers SharePoint accessibles.
- Parcourir l’arborescence Google Drive autorisée.
- Visualiser les fichiers compatibles (PDF, Word).

Règles :
- La navigation reflète uniquement les ressources autorisées par la plateforme source.
- Les fichiers non compatibles peuvent être visibles mais non sélectionnables (selon stratégie UI globale).

---

#### 2.3 Sélection et import d’un document

L’utilisateur peut :
- Sélectionner un document depuis SharePoint ou Drive.
- Lancer son import dans Comparia.

À l’import :
- Le fichier est récupéré via API.
- Il est transmis au module de stockage interne.
- Le mode de stockage (suppression ou archivage) est appliqué selon la configuration du tenant.
- L’utilisateur peut ensuite sélectionner un second document (local ou externe) pour lancer une comparaison.

L’import doit être compatible avec le traitement asynchrone si le document dépasse les seuils standards.

---

### 3. Sécurité et traçabilité

Pour chaque opération (authentification, navigation, import) :
- L’action est journalisée dans le module d’audit.
- Les informations minimales enregistrées : utilisateur, tenant, type d’opération, horodatage, identifiant du fichier source.

Les données doivent :
- Être transmises via HTTPS.
- Respecter les exigences RGPD.
- Ne pas être conservées au-delà de la politique de stockage configurée.

---

### 4. Contraintes fonctionnelles

- Isolation stricte multi-tenant.
- Respect des rôles et permissions internes Comparia.
- Aucun impact significatif sur les performances globales.
- Aucune synchronisation automatique permanente.

---

### 5. Dépendances

- Module d’authentification et gestion des rôles.
- Module d’audit.
- Service de stockage documentaire.
- Moteur de comparaison.
- Infrastructure de traitement asynchrone.

Ce ticket constitue la première version opérationnelle des connecteurs externes (import uniquement).
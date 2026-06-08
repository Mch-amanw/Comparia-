## Ticket : Implémenter configuration marque blanche – Spécification technique

### 1. Modèle de données

#### 1.1 Extension de la configuration tenant

Ajout ou enrichissement d’une entité `tenantConfiguration` avec un bloc dédié au branding :

```json
{
  "tenantId": "string",
  "version": "number",
  "branding": {
    "enabled": true,
    "instanceName": "string",
    "logoUrl": "string",
    "primaryColor": "#RRGGBB",
    "secondaryColor": "#RRGGBB"
  },
  "updatedAt": "datetime",
  "updatedBy": "userId"
}
```

Contraintes :
- `tenantId` obligatoire.
- `primaryColor` et `secondaryColor` validés via regex hexadécimale.
- `logoUrl` référence un fichier stocké de manière sécurisée.

La configuration est :
- Versionnée.
- Historisée via le module d’audit.

---

### 2. Stockage des assets

- Les logos sont stockés dans le module de stockage existant.
- Isolation stricte par tenant (namespace ou préfixe dédié).
- Accès sécurisé via URL signée ou contrôlée.
- Vérification du type MIME (image uniquement).

En cas d’échec de chargement : fallback automatique sur le logo par défaut.

---

### 3. API backend

Ajout d’endpoints (dans le cadre des API internes d’administration tenant) :

- `GET /api/v1/tenants/{tenantId}/branding`
- `PUT /api/v1/tenants/{tenantId}/branding`

Règles :
- Authentification obligatoire.
- Vérification des droits d’administration du tenant.
- Journalisation automatique des modifications.
- Isolation stricte : impossible de modifier le branding d’un autre tenant.

---

### 4. Injection dynamique côté frontend

#### 4.1 Chargement

Au login ou à l’initialisation :
- Récupération de la configuration du tenant.
- Stockage en mémoire applicative (ex : store global).

#### 4.2 Application des couleurs

- Utilisation de variables CSS dynamiques (ex : CSS custom properties).
- Injection des couleurs primaires et secondaires dans le thème.
- Aucun build spécifique par client.

Exemple :

```css
:root {
  --primary-color: #xxxxxx;
  --secondary-color: #yyyyyy;
}
```

#### 4.3 Logo et nom

- Le logo est affiché via l’URL configurée.
- Le nom d’instance remplace dynamiquement le nom par défaut dans :
  - Header.
  - Titre de page.

---

### 5. Sécurité

- Validation stricte des entrées (taille logo, format image, format couleurs).
- Protection contre injection via champs texte (instanceName).
- Aucune donnée de branding ne doit permettre d’injection de code (sanitization obligatoire).

---

### 6. Compatibilité SaaS et on-premise

- Même modèle de données dans les deux modes.
- En on-premise : configuration via API ou fichier de configuration initial.
- Aucune dépendance à une infrastructure SaaS spécifique.

---

### 7. Audit et traçabilité

Chaque modification de branding doit générer un événement d’audit contenant :
- tenantId
- utilisateur
- ancienne valeur
- nouvelle valeur
- timestamp

Conformément aux exigences globales d’audit du projet.
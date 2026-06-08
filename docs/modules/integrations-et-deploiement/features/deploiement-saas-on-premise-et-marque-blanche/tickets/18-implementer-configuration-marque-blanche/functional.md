## Ticket : Implémenter configuration marque blanche

### 1. Objectif

Permettre la personnalisation de l’identité visuelle de Comparia pour un tenant donné, sans modification fonctionnelle du produit.

La marque blanche doit permettre :
- La personnalisation du logo affiché dans l’application.
- La personnalisation des couleurs principales de l’interface.
- La personnalisation du nom de l’instance (nom affiché dans l’interface).

Cette personnalisation doit être strictement isolée par tenant et ne doit en aucun cas impacter :
- Le moteur de comparaison.
- Le moteur de scoring.
- Les mécanismes de sécurité.
- Les règles d’audit.

---

### 2. Périmètre fonctionnel

#### 2.1 Paramètres personnalisables

Pour chaque tenant, les éléments suivants peuvent être configurés :

1. **Logo**
   - Logo principal affiché dans le header.
   - Utilisé également sur les pages publiques éventuelles (ex : page de connexion si applicable).

2. **Couleurs principales**
   - Couleur primaire (ex : boutons, éléments actifs).
   - Couleur secondaire (ex : accents, liens, survol).
   - Les couleurs doivent respecter un format standard (ex : code hexadécimal).

3. **Nom de l’instance**
   - Nom affiché dans l’en-tête de l’application.
   - Nom visible dans les titres de page (ex : balise title HTML si applicable).

---

### 2.2 Activation par tenant

- La marque blanche est une option activable par tenant.
- Si aucun paramètre spécifique n’est défini, l’application utilise le branding par défaut de Comparia.
- Les paramètres ne sont visibles et appliqués que pour le tenant concerné.

---

### 2.3 Règles de gestion

- La marque blanche ne doit modifier que l’apparence visuelle.
- Aucune fonctionnalité ne peut être activée/désactivée via cette configuration.
- Les paramètres de branding sont indépendants des pondérations de score et des autres paramètres techniques.
- Toute modification des paramètres de marque blanche doit être journalisée dans le module d’audit (qui a modifié, quand, quoi).
- En mode on-premise, les mêmes capacités de configuration doivent être disponibles.

---

### 2.4 Comportement en cas d’erreur

- Si le logo personnalisé est indisponible ou invalide, le logo par défaut est utilisé.
- Si les couleurs sont invalides ou absentes, les couleurs par défaut sont appliquées.
- L’application doit rester entièrement fonctionnelle même en cas d’erreur de configuration visuelle.

---

### 3. Impacts UX

- Le branding est chargé dynamiquement à l’ouverture de session ou à l’initialisation de l’application.
- Le changement de branding doit être visible après rechargement de la session.
- Aucun redéploiement de l’application ne doit être nécessaire pour appliquer une nouvelle configuration de marque blanche.
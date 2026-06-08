## Ticket : Développer la vue côte à côte avec surlignage – Spécification technique

### 1. Positionnement

Ce ticket implémente la base des composants suivants dans le module Interface de visualisation :
- ViewerContainer (structure et états)
- DocumentPane (x2)
- Intégration initiale de DiffHighlighter

Aucune modification backend n’est requise.

---

### 2. Flux de données

1. ViewerContainer récupère via API REST :
   - Métadonnées de la comparaison (id, statut).
   - Structure normalisée des documents A et B.
   - Liste des différences (id, type, positions A/B, section).
2. Les données sont transmises aux deux DocumentPane.
3. DiffHighlighter applique les marqueurs de différences lors du rendu.

Contraintes :
- Les positions sont interprétées uniquement à partir des identifiants structurels ou offsets fournis.
- Aucune logique de diff n’est exécutée côté frontend.

---

### 3. Implémentation des composants

#### 3.1 ViewerContainer
Responsabilités :
- Gestion des états : loading, ready, error.
- Vérification du statut de la comparaison.
- Transmission des données aux sous-composants.
- Gestion d’un état local pour la différence sélectionnée.

#### 3.2 DocumentPane
Responsabilités :
- Rendu du contenu document structuré (sections, paragraphes, tableaux).
- Application des classes CSS liées aux différences.
- Gestion des événements de survol et sélection.

Contraintes :
- Virtualisation ou rendu progressif si nécessaire pour limiter le nombre de nœuds DOM.
- Aucune persistance locale durable.

#### 3.3 DiffHighlighter
Responsabilités :
- Injection de balises (ex : <span>) autour des unités concernées.
- Attribution de classes CSS selon le type :
  - diff-add
  - diff-delete
  - diff-modify
  - diff-moved
- Association via attribut data-diff-id pour relier les blocs correspondants.

Interaction :
- Survol : ajout d’une classe supplémentaire (ex : diff-hover) sur tous les éléments partageant le même data-diff-id.
- Sélection : ajout d’une classe persistante (ex : diff-selected).

---

### 4. Gestion des performances

- Mise en place d’un rendu progressif (lazy rendering) par section ou bloc logique.
- Limitation du nombre d’éléments DOM actifs simultanément.
- Éviter toute opération coûteuse lors des événements de survol.

Objectif :
- Affichage initial en quelques secondes après réception des données.
- Interaction fluide même avec un grand nombre de différences.

---

### 5. Sécurité

- Accès conditionné à la réussite de l’appel API authentifié.
- En cas de réponse indiquant absence de droits ou documents supprimés :
  - Ne pas rendre les documents.
  - Afficher un état d’erreur contrôlé.

Aucune donnée sensible ne doit être stockée dans le localStorage ou équivalent.

---

### 6. Accessibilité et i18n

- Toutes les chaînes statiques passent par le système i18n.
- Les différences ne doivent pas être identifiables uniquement par couleur (ajout d’icônes ou styles complémentaires).
- Structure HTML sémantique pour compatibilité lecteurs d’écran.

---

### 7. Compatibilité

- Compatible environnement SaaS multi-tenant.
- Compatible déploiement on-premise.
- Respect du système de thème pour marque blanche (classes neutres + variables CSS).
## Spécification technique – Affichage côte à côte interactif

### 1. Composants impactés

Cette fonctionnalité repose principalement sur les composants suivants du module Interface de visualisation :

- ViewerContainer
- DocumentPane (x2)
- DiffHighlighter
- DiffNavigator (intégration)
- CommentOverlay (intégration contextuelle)

Aucun composant backend n’est modifié.

---

### 2. Flux de données

1. Le ViewerContainer récupère via API REST :
   - La structure normalisée des deux documents.
   - La liste des différences (avec identifiant, type, positions A/B, section associée).
2. Les données sont transmises aux deux DocumentPane.
3. DiffHighlighter applique les métadonnées de diff sur le rendu HTML.

Contraintes :
- Le frontend ne génère pas les différences.
- Les positions sont interprétées selon les offsets ou identifiants structurels fournis par l’API.

---

### 3. Rendu et surlignage

#### 3.1 Mapping des différences

Chaque différence contient :
- id unique
- type (addition, deletion, modification, moved)
- position dans Document A et/ou B
- référence de section

Le DiffHighlighter :
- Injecte des marqueurs DOM (ex : spans avec classes dédiées).
- Associe chaque bloc à un identifiant commun pour permettre la synchronisation visuelle.

#### 3.2 Gestion des blocs déplacés

- Les blocs marqués "moved" sont rendus avec un style spécifique.
- Un lien logique (via id partagé) permet de surligner simultanément l’emplacement source et destination.

---

### 4. Synchronisation du scroll

Implémentation :
- Écoute des événements de scroll sur chaque DocumentPane.
- Calcul d’un ratio de progression basé sur la hauteur scrollable.
- Ajustement du scroll de l’autre panneau.

Optimisations :
- Throttling ou debouncing pour limiter les recalculs.
- Désactivation temporaire lors d’un scroll programmatique pour éviter les boucles infinies.

Mode désactivable :
- État booléen géré dans ViewerContainer.

---

### 5. Performance

- Virtualisation du rendu (affichage uniquement des sections visibles).
- Lazy rendering des pages ou sections.
- Limitation du nombre d’éléments DOM simultanément montés.
- Mise en cache mémoire limitée à la session active (pas de stockage persistant).

Objectif :
- Rendu initial en quelques secondes après réception des données.
- Interaction fluide sur documents jusqu’à 300 pages.

---

### 6. Sécurité et permissions

- Accès conditionné par token d’authentification valide.
- Les actions contextuelles (commenter, valider) ne sont activées que si l’API indique les permissions adéquates.
- Si l’API retourne un statut indiquant que les documents ne sont plus disponibles (mode suppression automatique), le composant affiche un état d’erreur contrôlé sans contenu.

---

### 7. Internationalisation et accessibilité

- Tous les libellés (types de différences, info-bulles, messages) passent par le système i18n central.
- Les différences doivent rester identifiables sans dépendre uniquement de la couleur (ex : icônes ou styles supplémentaires) pour accessibilité.

---

### 8. Contraintes techniques

- Aucune logique de scoring ou de calcul de diff côté client.
- Compatibilité SaaS multi-tenant et on-premise.
- Compatibilité avec le module marque blanche (thèmes, couleurs dynamiques).
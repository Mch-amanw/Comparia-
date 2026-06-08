## Module : Interface de visualisation – Spécification technique

### 1. Positionnement architectural

- Module frontend de l’application web Comparia.
- Communique exclusivement avec le backend via API REST.
- Ne contient aucune logique métier de calcul de score ou de diff.
- Consomme les données produites par :
  - Service de comparaison documentaire
  - Module de scoring
  - Module de collaboration
  - Module d’authentification/autorisation

---

### 2. Architecture interne du module

#### 2.1 Composants principaux

1. ViewerContainer
   - Gestion du chargement de la comparaison
   - Gestion des états (loading, ready, error)

2. DocumentPane (x2)
   - Rendu du document A et du document B
   - Gestion du scroll synchronisé

3. DiffHighlighter
   - Application des métadonnées de diff (positions, types)
   - Mapping entre unités comparées et rendu HTML

4. DiffNavigator
   - Navigation séquentielle dans les différences
   - Interaction avec la liste latérale

5. ScorePanel
   - Affichage score global
   - Affichage sous-scores et version du modèle

6. CommentOverlay
   - Affichage et création de commentaires liés à une différence

---

### 3. Modèle de données (côté frontend)

Le module consomme une structure normalisée fournie par l’API, incluant :

- Métadonnées de la comparaison (id, statut, version du scoring)
- Score global et sous-scores
- Structure logique du document (sections, titres, paragraphes, tableaux)
- Liste des différences avec :
  - Identifiant unique
  - Type (ajout, suppression, modification, déplacé)
  - Position dans document A et/ou B
  - Référence de section
- Commentaires associés

Aucune persistance locale durable des documents complets n’est autorisée.

---

### 4. Performance

- Chargement progressif (lazy loading) pour documents volumineux.
- Virtualisation du rendu pour éviter le chargement complet des 300 pages en DOM simultanément.
- Optimisation du scroll synchronisé pour éviter les désynchronisations.
- Temps d’affichage cible : rendu initial en quelques secondes après réception des données.

---

### 5. Sécurité

- Accès conditionné par un token d’authentification valide.
- Vérification des permissions via réponses API (lecture seule, commentaire, validation).
- Aucun calcul sensible effectué côté client.
- Pas d’exposition de données si la comparaison est supprimée (mode suppression automatique).

---

### 6. Gestion des erreurs

- Gestion des statuts : pending, processing, completed, failed.
- Affichage contrôlé des messages d’erreur renvoyés par l’API.
- Gestion des cas où les documents ne sont plus disponibles.

---

### 7. Internationalisation

- Système d’i18n centralisé.
- Toutes les chaînes externalisées.
- Support FR/EN au minimum.

---

### 8. Compatibilité et déploiement

- Compatible avec environnement SaaS multi-tenant.
- Compatible déploiement on-premise.
- Adaptable au module marque blanche (thème, logo, couleurs).

Le module doit rester découplé des spécificités d’intégration (SharePoint, Google Drive), celles-ci étant gérées en amont par le backend.
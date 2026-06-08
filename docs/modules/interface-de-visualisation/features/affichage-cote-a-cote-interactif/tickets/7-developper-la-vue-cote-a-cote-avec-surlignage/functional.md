## Ticket : Développer la vue côte à côte avec surlignage

### 1. Objectif
Implémenter la vue principale d’analyse permettant d’afficher simultanément les deux documents comparés (Document A – référence, Document B – comparé) avec mise en évidence visuelle des différences détectées par le backend.

Ce ticket couvre la mise en place de la structure de la vue, le rendu des documents et l’application du surlignage dynamique des différences conformément aux métadonnées fournies par l’API.

---

### 2. Périmètre fonctionnel

Inclus dans ce ticket :
- Affichage côte à côte des deux documents.
- Identification explicite du Document A (référence).
- Intégration du surlignage des différences (ajouts, suppressions, modifications, déplacements).
- Mise en évidence conjointe des blocs correspondants entre les deux documents.
- Respect strict des données fournies par l’API (aucun recalcul côté frontend).

Exclus :
- Calcul des différences.
- Calcul du score de similarité.
- Gestion complète des commentaires (uniquement préparation de l’intégration visuelle si nécessaire).
- Navigation avancée entre différences (traitée dans un autre ticket si applicable).

---

### 3. Comportement attendu

#### 3.1 Structure de la vue
- L’écran est divisé en deux panneaux verticaux :
  - Panneau gauche : Document A (référence).
  - Panneau droit : Document B.
- Le document de référence doit être visuellement identifiable (badge, libellé ou indication claire).
- Les deux documents doivent être lisibles et alignés visuellement.

#### 3.2 Surlignage des différences
Les différences doivent être rendues selon leur type :
- Ajout (présent uniquement dans Document B).
- Suppression (présent uniquement dans Document A).
- Modification.
- Bloc déplacé.

Règles :
- Les différences sont visibles immédiatement après chargement complet des données.
- Les blocs déplacés sont identifiés explicitement comme "déplacés" et non comme suppression + ajout.
- Le surlignage ne doit pas nuire à la lisibilité du texte.
- La distinction ne repose pas uniquement sur la couleur (accessibilité minimale).

#### 3.3 Interaction de base
- Au survol d’un bloc de différence, les blocs correspondants dans les deux documents sont mis en évidence.
- Lors de la sélection d’un bloc, un état visuel distinct est appliqué.
- Les interactions ne doivent pas modifier les données sous-jacentes.

#### 3.4 Gestion des états
La vue doit gérer les états suivants :
- Chargement des données.
- Comparaison prête.
- Erreur (comparaison introuvable, documents indisponibles, accès refusé).

En cas d’indisponibilité des documents (ex : suppression automatique activée), aucun contenu ne doit être affiché.

---

### 4. Contraintes fonctionnelles

- Respect strict des permissions fournies par l’API.
- Aucune altération ou recalcul des différences côté client.
- Support des documents volumineux (jusqu’à 300 pages) sans blocage majeur.
- Interface compatible multilingue (FR/EN via i18n).
- Compatible avec le système de thème (marque blanche).
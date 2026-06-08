## Module : Interface de visualisation

### 1. Rôle du module

L’Interface de visualisation est le module frontend chargé d’afficher les résultats de comparaison entre deux documents dans Comparia. Elle permet :

- L’affichage côte à côte des deux documents comparés,
- Le surlignage clair et interactif des différences (ajouts, suppressions, modifications, déplacements),
- La consultation du score global de similarité et, si disponible, des sous-scores,
- L’interaction utilisateur (navigation, filtres, commentaires, validation).

Ce module constitue le point d’entrée principal de l’analyse humaine des écarts détectés par le moteur de comparaison.

---

### 2. Fonctionnalités principales

#### 2.1 Affichage côte à côte

- Présentation simultanée du Document A (référence) et du Document B.
- Synchronisation verticale du défilement (scroll synchronisé).
- Mise en correspondance visuelle des sections équivalentes.
- Gestion des documents jusqu’à 300 pages.

Règles de gestion :
- Le document de référence doit être clairement identifié.
- L’utilisateur peut activer/désactiver la synchronisation de défilement.

---

#### 2.2 Surlignage des différences

Les différences détectées par le moteur sont affichées avec un code visuel distinct :

- Ajouts
- Suppressions
- Modifications
- Blocs déplacés

Règles de gestion :
- Les différences doivent être visibles immédiatement au chargement.
- Les blocs déplacés doivent être identifiés comme tels et non comme suppression + ajout.
- Le mode standard n’intègre pas la mise en page dans le score, mais les différences visuelles peuvent être affichées si activées.

---

#### 2.3 Navigation dans les différences

- Navigation "différence suivante / précédente".
- Liste latérale optionnelle des différences détectées.
- Accès direct à une section spécifique.

Règles de gestion :
- La navigation doit conserver le contexte visuel (scroll automatique vers la différence sélectionnée).
- L’ordre de navigation suit l’ordre logique du document de référence.

---

#### 2.4 Affichage du score de similarité

- Affichage du score global (0–100 %).
- Affichage éventuel des sous-scores (texte, titres, tableaux, structure) si fournis par l’API.
- Indication de la version du modèle de scoring utilisée.

Règles de gestion :
- Le score affiché doit correspondre strictement au résultat fourni par le backend.
- Aucun recalcul du score n’est effectué côté frontend.

---

#### 2.5 Interaction collaborative

En lien avec le module de collaboration :

- Ajout de commentaires sur une différence spécifique.
- Visualisation des commentaires existants.
- Validation ou refus d’une différence.

Règles de gestion :
- Les actions sont soumises aux droits d’accès de l’utilisateur.
- Toute action doit déclencher une journalisation via le backend.

---

#### 2.6 Gestion des états de traitement

- Affichage d’un état "en cours de traitement" pour les comparaisons asynchrones.
- Rafraîchissement automatique ou manuel du statut.
- Affichage d’erreurs en cas d’échec (ex : OCR impossible, document corrompu).

---

#### 2.7 Multilingue

- Interface disponible en français et en anglais.
- Les libellés, messages système et statuts sont internationalisés.

---

### 3. Contraintes fonctionnelles

- Clarté visuelle prioritaire pour un usage juridique et professionnel.
- Lisibilité optimale pour documents volumineux.
- Cohérence avec les règles de droits d’accès et de confidentialité.
- Respect du mode de stockage configuré (aucune exposition de documents supprimés automatiquement).
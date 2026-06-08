## Fonctionnalité : Affichage côte à côte interactif

### 1. Objectif métier
Permettre à l’utilisateur d’analyser visuellement et de manière intuitive les différences détectées entre deux documents comparés, en affichant les deux versions simultanément avec un surlignage dynamique des écarts (ajouts, suppressions, modifications, déplacements).

Cette fonctionnalité constitue le cœur de l’expérience d’analyse dans Comparia et doit offrir clarté, précision et fluidité, notamment pour des usages juridiques et administratifs.

---

### 2. Périmètre

Inclus :
- Affichage simultané du document de référence (Document A) et du document comparé (Document B).
- Surlignage dynamique des différences détectées par le backend.
- Synchronisation verticale du défilement (scroll synchronisé).
- Interaction utilisateur avec les blocs de différences (focus, sélection).
- Identification explicite du document de référence.

Exclus :
- Calcul des différences (réalisé exclusivement côté backend).
- Recalcul du score de similarité.
- Gestion complète des commentaires (couverte par la fonctionnalité collaborative dédiée, mais intégrable visuellement ici).

---

### 3. Règles de gestion

#### 3.1 Identification des documents
- Le Document A (référence) doit être clairement identifié visuellement.
- L’ordre des documents est déterminé par le backend et ne doit pas être inversé automatiquement côté frontend.

#### 3.2 Surlignage des différences
Les différences doivent être affichées selon un code visuel distinct et cohérent :
- Ajout (présent uniquement dans Document B)
- Suppression (présent uniquement dans Document A)
- Modification (contenu altéré entre A et B)
- Bloc déplacé (identifié comme déplacé et non comme suppression + ajout)

Règles associées :
- Les différences sont visibles dès le chargement complet de la comparaison.
- Les blocs déplacés doivent être identifiés explicitement comme tels.
- Le surlignage ne doit pas altérer la lisibilité du texte.
- L’affichage respecte strictement les métadonnées de diff fournies par l’API.

#### 3.3 Synchronisation du défilement
- Le défilement vertical est synchronisé par défaut.
- L’utilisateur peut activer/désactiver la synchronisation.
- En cas de différences importantes de longueur, le système doit maintenir un alignement logique basé sur les sections correspondantes.

#### 3.4 Interaction utilisateur
- Survol d’une différence : mise en évidence renforcée du bloc correspondant dans les deux documents.
- Sélection d’une différence : focus visuel et possibilité d’actions contextuelles (ex : commenter, valider si autorisé).
- La navigation entre différences (suivante/précédente) doit recentrer automatiquement l’affichage sur la différence ciblée.

#### 3.5 Gestion des documents volumineux
- La fonctionnalité doit supporter des documents jusqu’à 300 pages.
- L’affichage doit rester fluide même en présence d’un grand nombre de différences.

---

### 4. Critères d’acceptation

- Les deux documents sont affichés simultanément et correctement alignés.
- Les ajouts, suppressions, modifications et déplacements sont visuellement distinguables.
- Le scroll synchronisé fonctionne et peut être désactivé.
- La sélection d’une différence met en évidence les deux côtés correspondants.
- Aucun recalcul de différence ou de score n’est effectué côté frontend.
- L’affichage reste exploitable sur des documents de grande taille sans blocage majeur.

---

### 5. Contraintes

- Respect des droits d’accès : un utilisateur sans droit de consultation ne peut pas accéder à l’affichage.
- Respect du mode de stockage (si suppression automatique activée et documents indisponibles, l’affichage ne doit pas exposer de contenu).
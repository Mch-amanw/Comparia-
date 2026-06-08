## Fonctionnalité : Détection structurelle et déplacements

### 1. Objectif métier
Permettre à l’utilisateur d’identifier rapidement les modifications d’organisation entre deux versions d’un document, notamment :

- Les sections ajoutées ou supprimées,
- Les changements d’ordre des sections,
- Les modifications de hiérarchie (ex : passage de H2 à H3),
- Les blocs textuels ou sections déplacés sans modification majeure de contenu.

L’objectif est de distinguer clairement :
- Les modifications de fond (contenu modifié),
- Les réorganisations structurelles (déplacements, rehiérarchisation),

afin d’améliorer l’analyse juridique, contractuelle ou administrative.

---

### 2. Périmètre fonctionnel

La fonctionnalité couvre :

1. L’analyse de la structure logique du document (sections, titres, niveaux hiérarchiques).
2. La détection des différences suivantes :
   - Section ajoutée,
   - Section supprimée,
   - Section déplacée,
   - Section réordonnée,
   - Changement de niveau hiérarchique d’un titre.
3. L’identification des blocs textuels fortement similaires mais positionnés différemment.
4. La production d’indicateurs exploitables pour :
   - Le calcul du sous-score structurel (SS),
   - L’affichage des déplacements dans l’interface,
   - L’export des résultats.

Ne sont pas couverts :
- Les modifications purement visuelles (marges, polices),
- Les images ou éléments graphiques,
- Les métadonnées internes.

---

### 3. Règles de gestion

#### 3.1 Unité structurelle
L’unité principale d’analyse est la section logique issue de la normalisation du document, comprenant :
- Un titre (avec niveau hiérarchique),
- Un ensemble de paragraphes,
- Éventuellement des tableaux.

#### 3.2 Détection des sections ajoutées ou supprimées
- Une section présente dans le document de référence et absente dans le document comparé est marquée comme "supprimée".
- Une section absente du document de référence mais présente dans le document comparé est marquée comme "ajoutée".
- Ces événements impactent directement le sous-score structurel (SS).

#### 3.3 Détection des sections déplacées
Une section est considérée comme déplacée si :
- Son contenu textuel présente une similarité interne ≥ 95 %,
- Elle apparaît à une position différente dans l’ordre global des sections.

Règles associées :
- Elle est marquée explicitement comme "déplacée".
- Elle n’est pas comptée comme suppression + ajout dans le calcul du score textuel (ST).
- Elle génère une pénalité modérée dans le score structurel (SS).

#### 3.4 Changement de hiérarchie
Si un titre conserve un contenu textuel similaire mais change de niveau (ex : H2 → H3) :
- L’événement est signalé comme "changement de niveau".
- Il impacte prioritairement le sous-score SH (titres).
- Il peut impacter secondairement SS si la hiérarchie globale est altérée.

#### 3.5 Réorganisation globale
Un changement massif d’ordre des sections sans modification textuelle majeure :
- Impacte principalement SS,
- N’impacte pas ST si le contenu reste identique.

---

### 4. Résultat fonctionnel attendu

La fonctionnalité doit produire :

- Une liste structurée des différences structurelles :
  - type (ajoutSection, suppressionSection, deplacementSection, changementNiveau),
  - identifiant de section,
  - position initiale / nouvelle position,
  - niveau hiérarchique avant / après.
- Une liste explicite des blocs déplacés.
- Les éléments nécessaires au calcul du sous-score SS.

---

### 5. Critères d’acceptation

1. Deux documents strictement identiques en structure → aucune différence structurelle détectée, SS = 1.
2. Une section déplacée sans modification textuelle → marquée comme déplacée, pas de double comptage suppression/ajout.
3. Suppression d’une section entière → signalée et impact mesurable sur SS.
4. Changement uniquement de hiérarchie (H2 → H3) → détecté et visible dans les différences.
5. Réorganisation complète des sections sans modification textuelle → ST ≈ 1, SS significativement inférieur à 1.

La détection doit être déterministe pour une même entrée et configuration.
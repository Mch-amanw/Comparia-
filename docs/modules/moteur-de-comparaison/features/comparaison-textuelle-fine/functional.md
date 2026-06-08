## Fonctionnalité : Comparaison textuelle fine

### 1. Objectif métier
Permettre la détection précise des différences entre deux documents au niveau textuel, avec une granularité configurable (mot par défaut, caractère en option), afin d’identifier de manière fiable :

- Les ajouts,
- Les suppressions,
- Les modifications (remplacements),
- Les blocs textuels déplacés (en coordination avec la fonctionnalité dédiée).

Cette fonctionnalité constitue la base du score de similarité textuelle (ST) et alimente l’affichage des différences dans l’interface (surlignage côte à côte), les exports et l’API.

---

### 2. Périmètre
Inclus :
- Comparaison des paragraphes textuels normalisés issus des documents (PDF, Word, inter-format).
- Granularité mot par défaut.
- Option de granularité caractère par caractère.
- Production d’un diff structuré exploitable par le frontend et les rapports.
- Contribution directe au calcul du sous-score ST.

Exclus :
- Analyse de mise en page.
- Analyse des images.
- Calcul des sous-scores titres, tableaux et structure (gérés par d’autres fonctionnalités du module).

---

### 3. Règles de gestion

#### 3.1 Document de référence
- L’un des deux documents est défini comme "document de référence".
- Le calcul du score ST est basé sur le volume d’unités du document de référence (L).

#### 3.2 Granularité
- Mode par défaut : mot (tokenisation simple).
- Mode optionnel : caractère.
- La granularité utilisée est indiquée dans les métadonnées du résultat.

#### 3.3 Types de différences détectées
Chaque différence est classée comme :
- Ajout (présent dans document B uniquement),
- Suppression (présent dans document A uniquement),
- Modification (remplacement d’une ou plusieurs unités),
- Déplacement (si identifié par la fonctionnalité dédiée et exclu du calcul D).

#### 3.4 Calcul du sous-score ST
La formule appliquée est :

ST = 1 − (D / L)

Où :
- D = nombre total d’unités modifiées (ajouts + suppressions + remplacements), hors blocs déplacés détectés.
- L = nombre total d’unités du document de référence.

- ST est borné entre 0 et 1.
- ST = 1 si les contenus textuels sont strictement identiques.

#### 3.5 Gestion des blocs déplacés
- Les blocs textuels détectés comme déplacés (≥ 95 % de similarité interne) ne sont pas comptabilisés dans D.
- Ils sont marqués distinctement dans la liste des différences.

#### 3.6 Cas particuliers
- Document vide vs document non vide → ST = 0.
- Deux documents vides → ST = 1.
- Erreurs OCR : les différences issues du bruit OCR impactent ST sauf si une configuration de tolérance est activée.

---

### 4. Critères d’acceptation

1. Deux documents strictement identiques produisent :
   - ST = 1 (100 %),
   - Aucune différence listée.

2. Un mot ajouté dans le document comparé :
   - Une différence de type "ajout" détectée,
   - ST diminué proportionnellement à 1 / L.

3. Un remplacement d’un mot par un autre :
   - Différence de type "modification",
   - Impact équivalent à 1 unité modifiée.

4. Activation du mode caractère :
   - Les différences sont détectées au niveau des caractères,
   - ST recalculé sur le nombre total de caractères du document de référence.

5. Bloc déplacé sans modification interne :
   - Marqué comme "déplacé",
   - Non comptabilisé dans D.

6. Le résultat est déterministe :
   - Même entrée + même configuration → même liste de différences et même ST.

---

### 5. Dépendances
- Dépend de la normalisation préalable des documents.
- Interagit avec la fonctionnalité de détection des blocs déplacés.
- Alimente le calcul du score global via le sous-score ST.
- Fournit les données au frontend, au module de rapport et à l’API.
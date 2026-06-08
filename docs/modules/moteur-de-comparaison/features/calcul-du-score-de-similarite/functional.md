## Fonctionnalité : Calcul du score de similarité

### 1. Objectif métier
Le calcul du score de similarité a pour objectif de fournir une mesure synthétique, fiable et exploitable du degré de ressemblance entre deux documents.

Ce score :
- Permet une compréhension immédiate du niveau de modification.
- Sert d’indicateur clé dans l’interface (élément prioritaire d’affichage).
- Est utilisé dans les exports (PDF, Word) et exposé via l’API.
- Constitue une base pour des analyses plus fines (détail par section).

Le score doit être déterministe, reproductible et contractuellement traçable.

---

### 2. Périmètre fonctionnel

La fonctionnalité couvre :

1. Le calcul des sous-scores :
   - ST : similarité textuelle (paragraphes),
   - SH : similarité des titres,
   - SB : similarité des tableaux,
   - SS : similarité structurelle.

2. L’agrégation pondérée en un score global S.

3. La normalisation du score en pourcentage (0 à 100 %).

4. La préparation d’une base exploitable pour un détail par section.

Ne sont pas inclus dans cette fonctionnalité :
- L’affichage du score (géré par le frontend).
- La mise en page ou analyse visuelle (hors score standard).

---

### 3. Règles de gestion

#### 3.1 Échelle
- Le score global est compris entre 0 % et 100 %.
- 100 % = documents identiques (contenu textuel et structure).
- 0 % = absence de similarité significative.

#### 3.2 Formule de calcul

S = wT × ST + wH × SH + wB × SB + wS × SS

Avec :
- ST = 1 − (D / L) (similarité textuelle),
- SH = similarité des titres,
- SB = 1 − (Dtable / Ltable),
- SS = similarité structurelle.

Les poids par défaut sont :
- wT = 0,60
- wH = 0,15
- wB = 0,15
- wS = 0,10

La somme des poids doit être égale à 1.

#### 3.3 Configurabilité
- Les pondérations sont configurables par tenant.
- Toute modification de la formule ou des règles entraîne une nouvelle version du modèle de scoring.
- La version du modèle est obligatoirement incluse dans le résultat.

#### 3.4 Gestion des blocs déplacés
- Les blocs identifiés comme déplacés (≥ 95 % de similarité interne) :
  - N’impactent pas ST.
  - Impactent uniquement SS.

#### 3.5 Cas particuliers
- Documents strictement identiques → 100 %.
- Différences uniquement de mise en page → 100 % (mode standard).
- Document B contenant intégralement A + ajouts → score basé sur document de référence.
- Erreurs OCR : peuvent impacter ST, sauf activation d’une tolérance configurée.

#### 3.6 Arrondi et présentation
- Le score final est exprimé en pourcentage.
- Arrondi à deux décimales.

---

### 4. Détail par section (base préparatoire)

En complément du score global, la fonctionnalité doit produire une structure interne permettant :
- Le calcul d’un sous-score par section logique.
- L’identification des sections les plus impactées.

Ce détail :
- N’est pas nécessairement exposé intégralement en v1.
- Doit être exploitable par le frontend et les exports.

---

### 5. Critères d’acceptation

1. Pour une même paire de documents et une même configuration, le score est strictement identique.
2. Deux documents identiques retournent 100 %.
3. Une modification textuelle isolée impacte ST et donc S de manière proportionnelle.
4. Un bloc déplacé n’est pas compté comme suppression + ajout.
5. La version du modèle de scoring est présente dans le résultat.
6. Les poids personnalisés d’un tenant sont correctement appliqués.
7. Le score est disponible dans la structure de sortie exploitable par l’API et le module de reporting.

---

### 6. Dépendances

- Dépend des résultats des modules :
  - Comparaison textuelle,
  - Analyse des titres,
  - Analyse des tableaux,
  - Analyse structurelle.
- Utilise la configuration tenant (pondérations, tolérances).
- Alimente le frontend, les exports et le module d’audit.
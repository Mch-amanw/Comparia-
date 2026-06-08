## Ticket : Développer l’algorithme de diff mot et caractère

### 1. Objectif
Implémenter l’algorithme de comparaison textuelle fine permettant de détecter précisément les différences entre deux documents, avec une granularité configurable :

- Mode **mot** (par défaut),
- Mode **caractère** (optionnel).

Cet algorithme constitue le cœur opérationnel du calcul du sous-score de similarité textuelle (ST) et alimente la liste structurée des différences utilisée par l’interface, les exports et l’API.

---

### 2. Périmètre fonctionnel

Inclus :
- Comparaison de deux séquences d’unités textuelles normalisées (issues du pipeline de normalisation).
- Support des deux granularités : mot et caractère.
- Détection des opérations élémentaires :
  - Ajout,
  - Suppression,
  - Modification (remplacement),
  - Marquage des unités appartenant à un bloc déplacé (fourni par le module de détection de déplacements).
- Calcul de la valeur D (nombre total d’unités modifiées).
- Contribution au calcul du sous-score ST.

Exclus :
- Tokenisation brute (gérée en amont par la couche de normalisation).
- Détection des blocs déplacés (déjà fournie par la fonctionnalité dédiée).
- Calcul des sous-scores titres, tableaux et structure.

---

### 3. Comportement attendu

#### 3.1 Entrée logique
L’algorithme reçoit :
- Une séquence d’unités du document de référence A.
- Une séquence d’unités du document comparé B.
- La granularité utilisée ("mot" ou "caractere").
- La liste éventuelle des plages correspondant à des blocs déplacés (à exclure du calcul D).

Les séquences sont ordonnées et déjà normalisées.

#### 3.2 Détection des différences
L’algorithme doit produire un diff minimal cohérent permettant d’identifier :
- Les unités présentes uniquement dans B → ajouts.
- Les unités présentes uniquement dans A → suppressions.
- Les unités remplacées → modifications.

Les modifications peuvent concerner une ou plusieurs unités consécutives.

#### 3.3 Gestion des blocs déplacés
- Les unités appartenant à un bloc identifié comme déplacé (≥ 95 % de similarité interne, détecté en amont) ne doivent pas être comptabilisées dans D.
- Ces unités doivent être marquées comme "deplacement" dans la liste des différences.

#### 3.4 Calcul de D et ST
- L = nombre total d’unités du document de référence.
- D = nombre total d’unités impliquées dans des ajouts, suppressions ou remplacements (hors déplacements).

Formule :

ST = 1 − (D / L)

Règles :
- ST est borné entre 0 et 1.
- Si L = 0 et B vide → ST = 1.
- Si L = 0 et B non vide → ST = 0.

#### 3.5 Déterminisme
Pour une même entrée (A, B, granularité, configuration) :
- La séquence d’opérations retournée doit être identique.
- La valeur de D et de ST doit être strictement identique.

---

### 4. Résultat fonctionnel attendu

L’algorithme doit retourner :
- La liste ordonnée des différences textuelles.
- La valeur D.
- La valeur L.
- La valeur ST (non pondérée, entre 0 et 1).
- La granularité utilisée.

Ce résultat est intégré dans l’objet `ComparisonResult` au niveau du sous-score textuel.

---

### 5. Contraintes fonctionnelles

- Doit supporter des documents jusqu’à 300 pages.
- Doit fonctionner dans un temps compatible avec l’objectif global (< 1 minute pour une comparaison standard).
- Doit fonctionner en mode inter-format (après normalisation).
- Ne doit pas intégrer de logique spécifique à un tenant.
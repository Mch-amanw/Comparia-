## Ticket : Implémenter le calcul du score global v1

### 1. Objectif
Implémenter le calcul du score global de similarité (version v1) conformément à la définition formelle fournie dans la spécification du projet et de la fonctionnalité « Calcul du score de similarité ».

Ce ticket couvre exclusivement :
- L’agrégation pondérée des sous-scores (ST, SH, SB, SS),
- La validation et l’application des pondérations,
- La normalisation et l’arrondi du score final,
- L’intégration du scoringModelVersion dans le résultat.

Il ne couvre pas le calcul des sous-scores eux-mêmes (ST, SH, SB, SS), déjà produits par les modules en amont.

---

### 2. Périmètre fonctionnel

Le composant doit :

1. Recevoir en entrée :
   - Les sous-scores ST, SH, SB, SS (valeurs normalisées entre 0 et 1),
   - Les pondérations wT, wH, wB, wS,
   - La version du modèle de scoring (ex : "v1").

2. Valider les pondérations :
   - Vérifier que la somme des poids est égale à 1 (avec tolérance flottante contrôlée).
   - En cas d’incohérence, déclencher une erreur structurée.

3. Calculer le score global S selon la formule :

   S = wT × ST + wH × SH + wB × SB + wS × SS

4. Appliquer les règles suivantes :
   - S est borné entre 0 et 1.
   - Le score retourné est exprimé en pourcentage (S × 100).
   - Arrondi à deux décimales.

5. Intégrer dans le résultat :
   - Le score global en pourcentage,
   - Les sous-scores en pourcentage,
   - La version du modèle de scoring utilisée,
   - Les poids effectivement appliqués.

---

### 3. Règles de gestion spécifiques

- Le calcul doit être strictement déterministe.
- Aucune logique métier supplémentaire ne doit altérer les sous-scores fournis.
- Les blocs déplacés sont déjà exclus de ST en amont : aucune logique spécifique supplémentaire n’est ajoutée ici.
- En cas de valeurs en dehors des bornes [0,1] pour un sous-score, une erreur technique doit être levée (incohérence pipeline).
- Le modèle implémenté correspond à la version "v1" du scoring.

---

### 4. Hors périmètre

- Calcul interne de ST, SH, SB, SS.
- Détail du score par section (préparé ailleurs).
- Affichage frontend.
- Persistance des résultats.

---

### 5. Dépendances

- Dépend des métriques produites par :
  - Comparaison textuelle,
  - Analyse des titres,
  - Analyse des tableaux,
  - Analyse structurelle.
- Dépend du module de configuration tenant pour la récupération des pondérations.
- Alimente l’objet ComparisonResult consommé par l’API, le reporting et l’audit.
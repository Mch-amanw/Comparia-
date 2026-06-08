# Fonctionnalité – Extraction OCR multilingue

## 1. Positionnement technique

Cette fonctionnalité s’inscrit dans le module OCR et étend le moteur OCR pour gérer explicitement le support multilingue (FR/EN).

Pipeline concerné :
1. Upload PDF
2. Détection PDF scanné
3. Extraction OCR multilingue (cette fonctionnalité)
4. Normalisation
5. Comparaison
6. Scoring

---

## 2. Paramétrage linguistique

### 2.1 Modes de fonctionnement

Le moteur OCR doit supporter :
- Mode détection automatique de langue (FR/EN).
- Mode langue forcée (définie par configuration tenant).

Configuration possible par tenant :
- Langue par défaut.
- Autorisation ou non de la détection automatique.

Le choix linguistique utilisé doit être stocké dans les métadonnées de traitement.

---

## 3. Exigences techniques du moteur

### 3.1 Support des jeux de caractères

Le moteur doit gérer correctement :
- UTF-8 complet.
- Accents français.
- Caractères typographiques (guillemets, tirets longs, apostrophes courbes).

La sortie doit être normalisée en UTF-8 avant transmission au module de normalisation.

---

### 3.2 Reconstruction logique

Le traitement inclut :
1. Conversion PDF → images par page.
2. Prétraitement image (contraste, redressement si nécessaire).
3. Reconnaissance OCR avec modèle linguistique FR/EN.
4. Reconstruction en blocs textuels ordonnés.

Le texte doit être structuré en :
- Pages
- Blocs (paragraphes)
- Tableaux détectés si possible

---

## 4. Performance

- Traitement parallélisable par page.
- Intégration dans la file de tâches existante pour traitement asynchrone.
- Optimisation mémoire pour documents jusqu’à 300 pages.

Objectif :
- Ne pas dégrader l’objectif global (< 1 minute pour document standard incluant OCR).

Des métriques doivent être exposées :
- Temps moyen par page.
- Temps total par document.

---

## 5. Gestion des erreurs et seuil de confiance

Le moteur doit produire :
- Un taux de confiance global.

En cas de :
- Échec technique → statut "échec".
- Confiance anormalement faible → statut exploitable mais signalé (ou échec si seuil bloquant défini ultérieurement).

Les erreurs sont :
- Loguées avec ID document et tenant.
- Transmises au module d’audit.

---

## 6. Sécurité et conformité

- Traitement en environnement isolé (conteneur recommandé en SaaS).
- Fichiers temporaires supprimés après traitement.
- Données chiffrées en transit.
- Respect du mode de stockage configuré (suppression ou archivage).

Isolation logique stricte des données par tenant.

---

## 7. Versionnement

Toute évolution majeure du moteur linguistique (ex : changement de modèle OCR impactant significativement les résultats) doit :
- Être versionnée.
- Être traçable dans les métadonnées.
- Permettre la reproductibilité des comparaisons passées.

---

## 8. Dépendances techniques

- Module de stockage (lecture PDF, écriture résultats).
- Module de normalisation.
- Moteur de comparaison.
- Module d’audit.
- Infrastructure de file de tâches.

La fonctionnalité ne doit pas introduire de dépendance externe non maîtrisée incompatible avec les exigences SaaS et on-premise définies au niveau projet.
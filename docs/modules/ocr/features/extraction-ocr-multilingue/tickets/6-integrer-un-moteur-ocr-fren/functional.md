## Ticket : Intégrer un moteur OCR FR/EN

### 1. Objectif

Ce ticket a pour objectif d’intégrer concrètement un moteur OCR compatible **français (FR)** et **anglais (EN)** dans le module OCR de Comparia, conformément à la fonctionnalité *Extraction OCR multilingue*.

L’intégration doit permettre :
- L’extraction fiable de texte à partir de PDF scannés,
- La prise en charge des caractères spécifiques FR/EN,
- La gestion robuste des erreurs,
- Le respect des contraintes de performance du projet.

Ce ticket couvre l’intégration technique du moteur OCR, pas la refonte du pipeline global.

---

### 2. Périmètre fonctionnel

#### 2.1 Support des langues

Le moteur OCR intégré doit :
- Supporter au minimum le français et l’anglais.
- Permettre l’utilisation :
  - Soit d’un mode détection automatique FR/EN,
  - Soit d’une langue forcée selon la configuration tenant.

La langue effectivement utilisée doit être enregistrée dans les métadonnées de traitement.

---

#### 2.2 Extraction exploitable

Le moteur intégré doit produire :
- Un texte exploitable par le module de normalisation.
- Un encodage UTF-8 correct.
- Une restitution fidèle des caractères accentués (é, è, à, ç, etc.) et caractères typographiques.

Le résultat doit être structuré au minimum par :
- Pages,
- Blocs textuels (paragraphes lorsque détectables).

---

#### 2.3 Gestion des erreurs

L’intégration doit gérer les cas suivants :
- Fichier PDF corrompu ou illisible.
- Erreur interne du moteur OCR.
- Timeout ou dépassement de ressources.

En cas d’erreur :
- Le statut du traitement doit être marqué comme "échec".
- L’erreur doit être journalisée.
- Aucun résultat partiel incohérent ne doit être transmis au moteur de comparaison.

---

#### 2.4 Performance

L’intégration doit permettre :
- Un traitement compatible avec l’objectif global (< 1 minute pour un document standard incluant OCR).
- Une exécution parallélisable page par page lorsque possible.

L’intégration ne doit pas dégrader les performances globales du pipeline.

---

#### 2.5 Traçabilité

Pour chaque traitement OCR :
- La langue utilisée,
- Le taux de confiance global (si fourni par le moteur),
- La durée de traitement,
- Le statut (succès / échec)

Doivent être accessibles et transmis au module d’audit.

---

### 3. Hors périmètre

- Implémentation du module de détection de PDF scanné (déjà couvert ailleurs).
- Évolution vers d’autres langues.
- Optimisation avancée de reconstruction structurelle (hors intégration de base).

---

### 4. Dépendances

- Module de stockage (lecture PDF).
- Infrastructure de file de tâches.
- Module de normalisation.
- Module d’audit.
- Configuration tenant (langue par défaut / détection auto).

---

### 5. Règles de gestion

1. Le moteur OCR intégré doit produire un résultat déterministe pour un même document et une même configuration linguistique.
2. La sortie doit être compatible avec le moteur de normalisation existant.
3. Les erreurs OCR doivent empêcher la poursuite vers la comparaison.
4. Les données traitées respectent le mode de stockage configuré (suppression ou archivage).
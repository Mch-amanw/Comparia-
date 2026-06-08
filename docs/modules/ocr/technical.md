Spécification Technique

## 1. Positionnement dans l’architecture

Le module OCR est un service applicatif intégré à l’architecture modulaire de Comparia.

Il s’insère dans le pipeline suivant :
1. Upload du document
2. Détection type de PDF
3. OCR (si nécessaire)
4. Normalisation
5. Moteur de comparaison
6. Calcul du score

Le module peut être :
- Un microservice dédié (recommandé en SaaS),
- Ou un service interne au backend en environnement on-premise.

---

## 2. Entrées / Sorties

### 2.1 Entrées
- Fichier PDF (jusqu’à 50 Mo, 300 pages max cible).
- Paramètres :
  - Langue (optionnelle)
  - Configuration tenant (mode stockage, normalisation avancée).

### 2.2 Sorties
- Texte structuré par :
  - Pages
  - Blocs (paragraphes)
  - Éventuels tableaux détectés
- Métadonnées :
  - Confiance globale OCR
  - Langue détectée
  - Durée de traitement

Le format de sortie doit être compatible avec le moteur de normalisation et de comparaison.

---

## 3. Moteur OCR

### 3.1 Exigences
- Support multilingue (FR/EN prioritaire).
- Bonne précision sur documents administratifs et juridiques.
- Capacité à traiter des documents multi-pages volumineux.

### 3.2 Traitement
Étapes typiques :
1. Conversion PDF → images par page.
2. Prétraitement image (contraste, redressement si nécessaire).
3. Reconnaissance optique des caractères.
4. Reconstruction logique du texte.
5. Structuration minimale (blocs).

---

## 4. Performance et scalabilité

- Traitement parallélisable page par page.
- Intégration à une file de tâches pour traitement asynchrone.
- Scalabilité horizontale en environnement SaaS (multiplication des workers OCR).

Contraintes :
- Respect de l’objectif global < 1 minute pour document standard.
- Gestion contrôlée de la mémoire pour documents volumineux.

---

## 5. Sécurité et conformité

- Traitement en environnement sécurisé (conteneur isolé recommandé).
- Chiffrement des fichiers en transit.
- Suppression des fichiers temporaires après traitement.
- Respect du mode de stockage configuré (suppression automatique ou archivage).

En environnement SaaS :
- Isolation logique des données par tenant.

---

## 6. Journalisation technique

Logs techniques :
- ID document
- ID tenant
- Temps de traitement
- Nombre de pages
- Statut
- Taux de confiance moyen

Ces logs alimentent :
- Le module d’audit (niveau fonctionnel)
- Les métriques de performance (monitoring).

---

## 7. Contraintes et dépendances

Dépendances :
- Module de stockage (accès aux fichiers PDF)
- Module de normalisation
- Moteur de comparaison
- Module d’audit
- Infrastructure de file de tâches (pour asynchrone)

Contraintes :
- Résultat déterministe à configuration identique.
- Versionnement du moteur OCR si évolution majeure impactant le scoring.

---
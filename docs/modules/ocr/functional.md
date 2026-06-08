# Module OCR – Spécification Fonctionnelle

## 1. Rôle du module dans Comparia

Le module OCR (Optical Character Recognition) permet l’extraction de texte à partir de PDF scannés (documents image) afin de les rendre comparables par le moteur de comparaison textuelle et structurelle de Comparia.

Il constitue une étape de prétraitement indispensable lorsque le document ne contient pas de couche texte exploitable.

Le module OCR n’a pas vocation à modifier le contenu métier du document : il produit une version textuelle fidèle, normalisée et exploitable par les autres modules (moteur de comparaison, scoring, affichage des différences).

---

## 2. Périmètre fonctionnel

### 2.1 Détection automatique des PDF scannés

Lors du dépôt d’un document PDF :
- Le système détecte la présence ou l’absence de couche texte exploitable.
- Si le PDF est considéré comme "scanné" (image uniquement ou texte non exploitable), le module OCR est déclenché automatiquement.
- Si le PDF contient déjà une couche texte fiable, l’OCR n’est pas exécuté (sauf configuration spécifique future).

Règle de gestion :
- L’OCR est transparent pour l’utilisateur final (aucune action requise).

---

### 2.2 Extraction du texte

Le module doit :
- Extraire le texte page par page.
- Conserver l’ordre logique de lecture.
- Identifier les blocs textuels (paragraphes).
- Extraire le texte des tableaux lorsque détectables.

Le texte extrait est transmis au moteur de normalisation puis au moteur de comparaison.

---

### 2.3 Support multilingue

Le module doit supporter au minimum :
- Français (prioritaire)
- Anglais (prioritaire)

La langue peut être :
- Détectée automatiquement,
- Ou déterminée selon le paramétrage du tenant.

---

### 2.4 Intégration avec le scoring

Le texte issu de l’OCR alimente :
- Le calcul de similarité textuelle (ST),
- Le calcul de similarité des titres (SH),
- Le calcul des tableaux (SB),
- Le calcul structurel (SS) lorsque la structure peut être reconstruite.

Règle importante :
- Les erreurs OCR peuvent impacter le score de similarité.
- Un mécanisme de normalisation avancée peut être activé pour atténuer l’impact du bruit OCR (ex : normalisation des caractères spéciaux, espaces multiples).

---

### 2.5 Performance et traitement asynchrone

- L’OCR doit s’intégrer dans le pipeline de traitement global.
- Pour les documents volumineux (jusqu’à 300 pages), le traitement peut être asynchrone.
- L’utilisateur doit être informé de l’état : en cours, terminé, échec.

Objectif global :
- Permettre une comparaison complète (OCR inclus si nécessaire) en moins d’une minute pour un document standard.

---

### 2.6 Gestion des erreurs

En cas d’échec OCR :
- L’erreur est journalisée.
- L’utilisateur est informé que le document n’a pas pu être traité correctement.
- Aucun résultat de comparaison ne doit être produit sur un texte partiellement incohérent.

---

### 2.7 Journalisation et traçabilité

Les éléments suivants doivent être journalisés :
- Déclenchement de l’OCR,
- Durée du traitement,
- Langue utilisée/détectée,
- Statut (succès / échec).

Ces événements sont intégrés au module d’audit global de Comparia.

---

## 3. Règles de gestion principales

1. L’OCR ne s’exécute que si nécessaire (absence de texte exploitable).
2. Le texte extrait doit respecter l’ordre logique du document.
3. Le résultat OCR doit être déterministe pour un même document et une même configuration.
4. Les données extraites sont soumises aux mêmes règles de stockage (suppression ou archivage) que le document source.
5. Le module doit respecter les exigences RGPD et de confidentialité définies au niveau projet.

---

# technicalSpec

# Module OCR –
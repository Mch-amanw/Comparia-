# Comparia – Spécification Fonctionnelle

## 1. Objectifs du produit
Comparia est une application web permettant la comparaison avancée de documents professionnels, principalement PDF et Word (Excel en évolution). 

L’objectif est de :
- Détecter les différences fines de contenu (mot par mot, voire caractère par caractère),
- Identifier les différences structurelles et les déplacements de sections,
- Faciliter l’analyse, la validation et le partage des écarts,
- Garantir un haut niveau de confidentialité et de traçabilité.

## 2. Périmètre fonctionnel

### 2.1 Formats supportés
- PDF (y compris PDF scannés via OCR)
- Word (.doc/.docx)
- Évolution prévue : Excel
- Comparaison inter-format (ex : PDF vs Word)

### 2.2 Types de comparaison

#### 2.2.1 Comparaison textuelle
- Détection fine mot par mot
- Option de granularité caractère par caractère
- Détection des ajouts, suppressions et modifications

#### 2.2.2 Comparaison structurelle
- Détection des modifications de structure
- Détection des sections déplacées

#### 2.2.3 Comparaison visuelle
- Option de comparaison de mise en page
- Mise en évidence visuelle des différences

### 2.3 OCR
- Intégration d’un module OCR pour les PDF scannés
- Extraction du texte afin de permettre la comparaison textuelle

### 2.4 Affichage des résultats
- Vue côte à côte des documents
- Surlignage des différences
- Score global de similarité (prioritaire)
- Détail du score par section (si possible)

### 2.5 Collaboration
- Ajout de commentaires sur les différences détectées
- Validation ou refus des différences
- Partage sécurisé d’un lien de comparaison

### 2.6 Rapports exportables
- Export des résultats en PDF
- Export en Word
- Évolution prévue : Excel et JSON

### 2.7 Gestion des versions
- Comparaison de deux versions à la demande (priorité)
- Historique centralisé des versions (évolution future)

### 2.8 Gestion des utilisateurs et droits
- Gestion des rôles utilisateurs
- Droits d’accès par document
- Option marque blanche pour certains clients

### 2.9 Journal d’audit
- Traçabilité des actions :
  - Qui a comparé
  - Quels documents
  - Quand
  - Résultats associés

### 2.10 Stockage des documents
Deux modes configurables selon le client :
- Suppression automatique après traitement
- Archivage des documents et comparaisons

### 2.11 Intégrations
- Intégration avec SharePoint
- Intégration avec Google Drive
- API ouvertes pour intégrations tierces

## 3. Utilisateurs cibles
- Entreprises
- Juristes
- Équipes administratives
- Équipes métiers

## 4. Exigences de performance
- Documents moyens : 10 à 50 Mo
- Jusqu’à 300 pages
- 100 à 500 comparaisons par jour au lancement
- Objectif : comparaison standard en moins d’1 minute
- Traitement asynchrone pour les fichiers volumineux

## 5. Multilingue
- Support multilingue dès le lancement
- Priorité : français et anglais

## 6. Règles de gestion principales
- Toute comparaison génère un score global de similarité.
- Les différences doivent être visualisables clairement dans l’interface.
- Les droits d’accès conditionnent la consultation et l’action sur les documents.
- Toutes les actions importantes doivent être journalisées.
- Le mode de stockage (suppression ou archivage) dépend de la configuration client.

---

# technicalSpec

## 1. Architecture générale

### 1.1 Type d’application
- Application web en priorité (SaaS)
- Option de déploiement on-premise pour clients sensibles

### 1.2 Architecture logique
Architecture modulaire comprenant :
- Frontend web (interface de comparaison et collaboration)
- Backend applicatif (API REST)
- Service de comparaison documentaire
- Service OCR
- Service de génération de rapports
- Module d’authentification et gestion des rôles
- Module d’audit

Traitement asynchrone pour les comparaisons volumineuses.

## 2. Moteur de comparaison
- Extraction du texte depuis PDF et Word
- Normalisation des contenus pour comparaison inter-format
- Algorithme de diff fin (mot et caractère)
- Détection des déplacements de sections
- Calcul d’un score global de similarité
- Calcul possible de scores par section

## 3. OCR
- Intégration d’un moteur OCR pour PDF scannés
- Traitement en amont de la comparaison
- Support multilingue (FR/EN prioritaire)

## 4. Performance et scalabilité
- Support de fichiers jusqu’à 50 Mo et 300 pages
- Capacité cible : 100 à 500 comparaisons par jour
- Temps cible < 1 minute pour un document standard
- Mise en place de traitements asynchrones et file de tâches
- Scalabilité horizontale côté services de traitement en environnement SaaS

## 5. Sécurité

### 5.1 Accès
- Authentification sécurisée
- Gestion des rôles et permissions par document

### 5.2 Confidentialité
- Chiffrement des données en transit
- Chiffrement des données stockées
- Conformité RGPD

### 5.3 Hébergement
- SaaS prioritaire
- Option on-premise pour clients sensibles

## 6. Stockage
- Stockage des documents et résultats selon configuration client
- Mode suppression automatique possible
- Mode archivage avec conservation des historiques

## 7. Journalisation et audit
- Enregistrement des actions utilisateur
- Conservation des logs d’accès et de comparaison

## 8. Intégrations
- Connecteurs vers SharePoint
- Connecteurs vers Google Drive
- API ouvertes pour intégration avec outils tiers

## 9. Marque blanche
- Paramétrage de l’identité visuelle
- Adaptation du branding pour clients spécifiques (notamment on-premise)

---
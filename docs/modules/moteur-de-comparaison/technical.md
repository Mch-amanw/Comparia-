## 1. Rôle du module
Le moteur de comparaison est un service backend central responsable de :
- L’extraction et la normalisation du contenu des documents.
- L’analyse textuelle fine (mot et caractère).
- L’analyse structurelle (titres, sections, tableaux, déplacements).
- Le calcul des sous-scores et du score global de similarité.
- La production d’un résultat structuré exploitable par l’API et le frontend.

Il est conçu comme un service stateless, scalable horizontalement, exposé via API interne (REST ou message queue).

---

## 2. Entrées et sorties

### 2.1 Entrées
- documentA (référence)
- documentB (comparé)
- Métadonnées :
  - format (pdf, docx)
  - langue détectée ou déclarée
  - modeGranularite (mot | caractere)
  - modeScoring (standard v1)
  - configurationTenant (pondérations, tolérances OCR)

Les documents doivent être fournis :
- Soit sous forme de fichiers binaires
- Soit sous forme de représentation intermédiaire normalisée (JSON documentaire)

### 2.2 Sorties
Objet résultat structuré contenant :
- differencesTextuelles (liste d’opérations diff)
- differencesStructurelles
- blocsDeplaces
- sousScores : { st, sh, sb, ss }
- scoreGlobal
- meta :
  - scoringVersion
  - granularite
  - tempsTraitementMs
  - tailleDocuments

Le résultat est déterministe pour une même entrée et configuration.

---

## 3. Pipeline de traitement

### 3.1 Étape 1 – Extraction
- PDF natif : extraction via moteur PDF text layer.
- PDF scanné : passage préalable par module OCR.
- DOCX : parsing XML (OpenXML).

Extraction structurée en :
- Paragraphes
- Titres (avec niveau hiérarchique)
- Tableaux (cellules normalisées)
- Sections logiques

### 3.2 Étape 2 – Normalisation
Objectif : permettre comparaison inter-format.

Normalisations appliquées :
- Uniformisation encodage (UTF-8)
- Suppression espaces multiples
- Normalisation ponctuation
- Option : normalisation casse
- Option : suppression stop-words (mode avancé)

Création d’une représentation interne canonique :
DocumentNormalise = {
  sections: [
    {
      id,
      titre,
      niveau,
      paragraphes[],
      tableaux[]
    }
  ]
}

---

## 4. Algorithmes de comparaison

### 4.1 Similarité textuelle (ST)

Algorithme principal : Myers diff optimisé ou LCS amélioré.
Unité par défaut : mot (tokenisation linguistique simple).
Option : caractère.

Formule :
ST = 1 − (D / L)

Où :
- D = ajouts + suppressions + remplacements
- L = nombre d’unités du document de référence

Optimisations :
- Segmentation par blocs pour réduire complexité sur 300 pages
- Hash partiel pour ignorer blocs strictement identiques

Complexité cible : proche de O(ND) optimisé avec segmentation.

---

### 4.2 Détection des blocs déplacés

Méthode :
- Génération d’empreintes (hash SHA-256 tronqué) par bloc normalisé.
- Indexation des blocs du document A.
- Recherche correspondances dans document B.

Si similarité interne ≥ 95 % :
- Marqué comme blocDeplace
- Exclu du calcul D (ST)
- Impact modéré appliqué à SS

---

### 4.3 Similarité des titres (SH)

Comparaison basée sur :
- Similarité textuelle du titre
- Correspondance du niveau hiérarchique

Pénalités :
- Changement de niveau
- Suppression ou ajout de titre

SH normalisé entre 0 et 1.

---

### 4.4 Similarité des tableaux (SB)

- Extraction texte cellule par cellule
- Diff ligne par ligne
- Détection déplacements de lignes

Formule :
SB = 1 − (Dtable / Ltable)

La mise en forme est ignorée.

---

### 4.5 Similarité structurelle (SS)

Basée sur :
- Présence/absence de sections
- Ordre relatif (distance de Kendall tau simplifiée)
- Nombre de blocs déplacés

SS = 1 − (penalitesStructure / totalSections)

SS borné entre 0 et 1.

---

## 5. Calcul du score global

S = wT * ST + wH * SH + wB * SB + wS * SS

Poids par défaut :
- wT = 0.60
- wH = 0.15
- wB = 0.15
- wS = 0.10

Les poids sont :
- Configurables par tenant
- Validés pour somme = 1

Score final exprimé en pourcentage arrondi à 2 décimales.

---

## 6. Performance et scalabilité

Contraintes :
- ≤ 300 pages
- ≤ 50 Mo
- < 60 secondes cible

Mécanismes :
- Traitement asynchrone via file de tâches
- Worker scalable horizontalement
- Segmentation du document en blocs parallélisables
- Cache des empreintes pour documents déjà analysés

Timeout configurable.

---

## 7. Déterminisme et versioning

- Toute modification d’algorithme incrémente scoringVersion (ex: v1, v2).
- Le moteur conserve la version utilisée dans le résultat.
- Les anciens scores restent recalculables avec leur version d’origine.

---

## 8. Sécurité

- Traitement en mémoire volatile prioritaire.
- Aucun stockage persistant interne au moteur.
- Communication chiffrée (TLS) avec services appelants.
- Isolation stricte des traitements multi-tenant.

---

## 9. Observabilité

Le moteur expose :
- tempsExtractionMs
- tempsNormalisationMs
- tempsComparaisonMs
- utilisationMemoireEstimee
- tailleTokens

Logs techniques sans contenu textuel pour conformité RGPD.

---

## 10. Gestion des erreurs

Cas gérés :
- Format non supporté
- Échec OCR
- Document vide
- Dépassement mémoire
- Timeout

Retour d’erreur structuré :
{
  code,
  message,
  etape,
  retryPossible
}

Aucune fuite de contenu document dans les messages d’erreur.
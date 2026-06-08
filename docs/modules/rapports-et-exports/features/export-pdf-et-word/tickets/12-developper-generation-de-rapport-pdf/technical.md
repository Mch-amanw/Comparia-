## Ticket : Développer génération de rapport PDF – Spécification technique

### 1. Objectif technique
Implémenter le rendu du rapport au format **PDF** à partir du comparisonReportModel v1, en respectant :
- Le déterminisme,
- Les paramètres d’export,
- Les contraintes de performance (≤ 300 pages),
- Les règles de sécurité et de traçabilité.

---

### 2. Composants concernés

- reportBuilder
- templateEngine
- formatRenderer (PDF)
- exportStorageHandler
- exportAuditLogger

Le ticket porte principalement sur l’implémentation ou la complétion du composant **formatRenderer pour le format PDF**.

---

### 3. Flux interne détaillé

1. Réception d’un comparisonReportModel v1 filtré selon paramètres.
2. Transformation en modèle de vue PDF via reportBuilder.
3. Injection des données dans un template HTML structuré.
4. Conversion HTML → PDF via moteur dédié.
5. Calcul du hash SHA-256 du fichier généré.
6. Transmission au exportStorageHandler.
7. Retour de l’exportId et du statut.

---

### 4. Construction du template PDF

#### 4.1 Approche

- Génération via template HTML + feuille de style dédiée.
- Conversion en PDF via moteur compatible serveur.
- Séparation stricte :
  - Données (JSON intermédiaire)
  - Présentation (template HTML + CSS).

#### 4.2 Structure minimale du template

- Page de couverture / synthèse
- Section “Résumé des différences”
- Section “Détail des différences”
- Pied de page contenant :
  - exportId
  - scoringModelVersion
  - date de génération

---

### 5. Gestion des paramètres

Le renderer doit prendre en compte :

- includeComments : inclure ou non la section commentaires.
- includeValidatedOnly : filtrer differences[].
- detailLevel :
  - summary → exclure le détail complet.
  - standard/exhaustive → inclure détail complet.
- language : sélection du template localisé.

Le filtrage des différences doit être effectué avant le rendu final.

---

### 6. Gestion des différences

Pour chaque entrée differences[] :
- type : affichage explicite (addition, deletion, modification, moved).
- moved : label spécifique "Section déplacée".
- originalText / modifiedText : affichage conditionnel selon type.
- validatedStatus : affichage textuel.

Les blocs doivent être paginés proprement (pas de coupure incohérente au milieu d’un bloc logique si possible).

---

### 7. Branding tenant

Le templateEngine doit permettre :
- Injection du logo.
- Application des couleurs principales.
- Affichage du nom du tenant.

En absence de configuration spécifique → template standard.

---

### 8. Performance

- Support jusqu’à 300 pages.
- Pagination automatique gérée par le moteur PDF.
- Optimisation mémoire : éviter de charger inutilement des documents sources complets.

Le composant doit être compatible avec exécution synchrone et asynchrone (orchestration externe).

---

### 9. Sécurité

- Encodage UTF-8 obligatoire.
- Aucune inclusion de chemin ou ressource externe non contrôlée.
- Fichier généré transmis uniquement via exportStorageHandler.

---

### 10. Déterminisme

À données identiques et paramètres identiques :
- Le contenu HTML généré doit être identique.
- Le PDF rendu doit être identique hors timestamp interne.
- Le hash SHA-256 calculé doit être reproductible.

Les timestamps doivent être injectés de manière contrôlée pour ne pas casser la reproductibilité si exigé par la configuration.

---

### 11. Interfaces attendues

Le formatRenderer doit exposer une méthode interne de type :

renderPdf(reportViewModel, exportConfig) → GeneratedFile

GeneratedFile contient :
- binaryContent
- mimeType = application/pdf
- fileName normalisé
- sha256Hash

---

### 12. Journalisation

Après génération réussie :
- Appel à exportAuditLogger avec :
  - exportId
  - comparisonId
  - format = pdf
  - paramètres
  - userId
  - scoringModelVersion
  - sha256Hash

---

Fin de spécification technique du ticket.
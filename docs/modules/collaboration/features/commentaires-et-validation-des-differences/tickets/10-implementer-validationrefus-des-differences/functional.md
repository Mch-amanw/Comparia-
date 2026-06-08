## Ticket : Implémenter validation / refus des différences

### 1. Objectif
Permettre aux utilisateurs autorisés de modifier le statut de revue d’une différence détectée lors d’une comparaison, en la marquant comme validée ou refusée.

Cette fonctionnalité s’inscrit dans le processus de revue collaborative et permet de formaliser la décision prise sur chaque écart détecté sans impacter le score de similarité.

---

### 2. Périmètre fonctionnel

#### 2.1 Statut par défaut
- Toute différence détectée lors de la génération d’une comparaison possède un statut initial : `pending` (À valider).
- Ce statut est visible dans l’interface de comparaison.

#### 2.2 Actions disponibles
Un utilisateur disposant des droits adéquats peut :
- Marquer une différence comme `validated` (Validée).
- Marquer une différence comme `rejected` (Refusée).
- Optionnellement (si activé côté configuration), repositionner une différence vers `needsReview`.

Chaque action :
- Écrase le statut précédent.
- Est immédiatement reflétée dans l’interface.
- Met à jour les indicateurs de synthèse (compteurs par statut).

#### 2.3 Visibilité dans l’interface
- Le statut courant est affiché pour chaque différence.
- Les différences peuvent être filtrées par statut (`pending`, `validated`, `rejected`, `needsReview`).
- Une vue synthétique affiche :
  - Nombre total de différences,
  - Nombre validées,
  - Nombre refusées,
  - Nombre restantes.

#### 2.4 Règles métier
1. Seuls les utilisateurs autorisés (selon rôles et droits sur la comparaison) peuvent modifier un statut.
2. La validation ou le refus d’une différence n’impacte ni le score global de similarité, ni les sous-scores (ST, SH, SB, SS).
3. Les statuts sont liés à une version spécifique de la comparaison (`comparisonId`).
4. En cas de suppression de la comparaison (selon politique de stockage), les statuts deviennent inaccessibles.
5. Toute action de changement de statut est tracée dans le module d’audit.

---

### 3. Hors périmètre
- Validation globale de la comparaison (gérée par une entité distincte si activée).
- Modification du contenu des différences.
- Impact sur le moteur de scoring ou recalcul automatique.

---

### 4. Dépendances
- Module Comparaison (référentiel des différences).
- Module Authentification & Rôles (contrôle d’accès).
- Module Audit (journalisation).
- Module Rapport (consommation des statuts pour export).
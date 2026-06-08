# Fonctionnalité – Extraction OCR multilingue

## 1. Objectif métier

La fonctionnalité **Extraction OCR multilingue** permet d’extraire automatiquement le texte de PDF scannés en prenant en charge au minimum le **français** et l’**anglais**, afin de rendre ces documents exploitables par le moteur de comparaison textuelle et structurelle de Comparia.

Elle garantit que les documents scannés, fréquents dans les contextes juridiques et administratifs, puissent être comparés avec le même niveau d’analyse que les documents natifs (PDF texte ou Word).

---

## 2. Périmètre fonctionnel

### 2.1 Langues supportées

La fonctionnalité doit supporter au minimum :
- Français (FR)
- Anglais (EN)

La langue utilisée par le moteur OCR peut être :
- Détectée automatiquement à partir du contenu,
- Définie via un paramétrage tenant (priorité ou langue par défaut).

En cas de document multilingue (FR/EN), le moteur doit produire un texte exploitable sans nécessiter une segmentation manuelle.

---

### 2.2 Déclenchement

L’extraction OCR multilingue est déclenchée uniquement si :
- Le document PDF est détecté comme scanné (absence de couche texte exploitable), conformément aux règles du module OCR.

La fonctionnalité est transparente pour l’utilisateur :
- Aucun choix technique n’est requis lors du dépôt.
- Le traitement est automatique.

---

### 2.3 Qualité d’extraction

Le texte extrait doit :
- Respecter l’ordre logique de lecture.
- Conserver les séparations en paragraphes lorsque détectables.
- Extraire le texte des tableaux lorsque possible.
- Gérer correctement les caractères spécifiques :
  - Accents et caractères spéciaux français (é, è, à, ç, etc.)
  - Apostrophes, guillemets, tirets typographiques

Une attention particulière doit être portée aux documents juridiques comportant :
- Numérotations structurées
- Titres hiérarchisés
- Articles et clauses

---

### 2.4 Intégration avec le moteur de comparaison

Le texte issu de l’OCR multilingue alimente directement :
- Le calcul de similarité textuelle (ST),
- Le calcul de similarité des titres (SH),
- Le calcul des tableaux (SB),
- Le calcul structurel (SS) lorsque la structure est détectable.

Règles de gestion :
- Les erreurs OCR peuvent impacter le score.
- La normalisation avancée (si activée au niveau tenant) peut atténuer l’impact des erreurs typiques (espaces multiples, caractères spéciaux mal reconnus).

---

### 2.5 Indicateurs et traçabilité

Pour chaque traitement OCR multilingue, les informations suivantes doivent être disponibles :
- Langue détectée ou utilisée,
- Taux de confiance global,
- Durée de traitement,
- Statut (succès / échec).

Ces informations sont journalisées dans le module d’audit.

---

## 3. Règles de gestion

1. L’OCR multilingue ne s’exécute que si le PDF est scanné.
2. La prise en charge FR/EN est obligatoire.
3. Le résultat doit être déterministe pour un même document et une même configuration linguistique.
4. Les données extraites respectent le mode de stockage configuré (suppression automatique ou archivage).
5. En cas d’échec ou de confiance trop faible rendant la comparaison incohérente, le traitement global doit être marqué en échec.

---

## 4. Critères d’acceptation

- ✅ Un PDF scanné en français produit un texte exploitable avec accents correctement reconnus.
- ✅ Un PDF scanné en anglais produit un texte exploitable sans dégradation notable.
- ✅ Un document mixte FR/EN est traité sans erreur bloquante.
- ✅ Les informations de langue et de confiance sont journalisées.
- ✅ Le texte extrait peut être transmis sans erreur au moteur de comparaison.
- ✅ Le traitement respecte l’objectif global de performance (< 1 minute pour document standard, OCR inclus si nécessaire).

---

## 5. Dépendances

- Détection des PDF scannés (module OCR – étape préalable).
- Module de normalisation.
- Moteur de comparaison et scoring.
- Module d’audit.
- Infrastructure de traitement asynchrone.

---

# Conclusion

La fonctionnalité Extraction OCR multilingue garantit que les documents scannés en français et en anglais sont traités de manière fiable, sécurisée et compatible avec l’ensemble du pipeline de comparaison de Comparia.
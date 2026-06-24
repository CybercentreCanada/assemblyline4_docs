# Triage des alertes

Lorsque les soumissions ont `generate_alert` activé et déclenchent des conditions définies, des alertes sont émises pour attirer l'attention et permettre un examen complémentaire.

Il s'agit d'un sous-ensemble des données de soumission jugées intéressantes, comme un fichier signalé comme malveillant ou une heuristique déclenchée.

## Gestion des alertes

Pour trier les alertes dans Assemblyline, une interface dédiée vous permet de visualiser, filtrer et gérer les alertes facilement.

Vous pouvez accéder à cette interface en cliquant sur l'onglet "Alerts" (<svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" ><path d="M18 16v-5c0-3.07-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.63 5.36 6 7.92 6 11v5l-2 2v1h16v-1zm-5 0h-2v-2h2zm0-4h-2V8h2zm-1 10c1.1 0 2-.9 2-2h-4c0 1.1.89 2 2 2"></path></svg>) dans le menu de navigation principal.

### Naviguer dans l'interface

Pour vous familiariser avec l'interface Alerts, voici quelques fonctionnalités clés de la vue principale.

<video controls src="../../../user_manual/assets/triage_alerts_view.mp4" title="Interface Alerts" autoplay></video>

#### Filtres

Cliquer sur le bouton "Filters" (:material-filter-variant:) ouvre un panneau où vous pouvez sélectionner les filtres à appliquer à la liste des alertes.

Vous pouvez aussi partager des filtres avec d'autres utilisateurs en cliquant sur le bouton "Share" (:material-star:). Cela ouvre un panneau dans lequel vous choisissez quels filtres partager et avec qui. Cela peut être utile pour partager des vues spécifiques d'alertes avec d'autres analystes ou pour créer des vues partagées pour une équipe.

<video controls src="../../../user_manual/assets/triage_filters.mp4" title="Filtres" autoplay></video>


#### Utiliser les workflows

Définissez des actions automatisées pour les alertes qui répondent à certaines conditions via des workflows. Par exemple, marquer toutes les alertes étiquetées malveillantes et contenant "invoice" comme "PHISHING".

Il existe deux approches sur la page Alerts :

- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg>+ **Workflows persistants** :
    En utilisant le filtre actuel, créez un workflow persistant qui sera appliqué à toutes les futures alertes correspondantes.
    Vous pouvez gérer ces workflows persistants depuis la page "[Manage Workflows](./#workflow-management)".

- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg>:material-play: **Workflows éphémères** :
    Créez une action ponctuelle sur les alertes correspondantes actuelles, sans l'enregistrer comme workflow persistant.
    Cela peut être utile pour effectuer rapidement des actions sur un ensemble spécifique d'alertes sans créer un nouveau workflow.

Les workflows vous permettent de :

- **Assigner un statut** : Définir le statut de l'alerte (ex. MALICIOUS, NON-MALICIOUS).
- **Assigner une priorité** : Spécifier le niveau d'urgence de l'alerte (ex. LOW, MEDIUM, HIGH).
- **Assigner des étiquettes** : Catégoriser l'alerte pour une gestion organisée.

<video controls src="../../../user_manual/assets/triage_workflow_actions.mp4" title="Actions de workflow" autoplay></video>

### Détails d'une alerte

Cliquer sur une carte d'alerte ouvre la vue de détails, où vous pouvez voir toutes les informations liées à cette alerte spécifique.

Cela inclut la classification, le verdict, les étiquettes, les détails du fichier, les métadonnées et les indicateurs.

<div class="grid" markdown>
<div class="grid" markdown>
Les détails d'alerte couvrent :

- **Classification** : Niveau de sécurité de l'alerte.
- **Informations de base** : Données clés sur l'alerte.
- **Verdict et étiquettes** : Niveau de menace évalué et tags pertinents.
- **Détails du fichier** : Informations telles que le nom de fichier, le type, la taille et les hachages.
- **Métadonnées et indicateurs** : Données additionnelles et signaux de sécurité potentiels.

Les actions pouvant être effectuées par les analystes de triage incluent :

- :material-briefcase-clock-outline: **Voir l'historique** : Examiner le journal des changements de l'alerte.
- :material-image-filter-center-focus-weak: **Afficher toutes les alertes du groupe** : Se concentrer sur les alertes de la même catégorie.
- :material-clipboard-account-outline: **Prendre possession** : Revendiquer l'alerte pour la gestion de cas.
- :material-view-carousel-outline: **Aller à la soumission liée** : Passer aux détails de la soumission associée.
- <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 24 24" height="18px" width="18px" xmlns="http://www.w3.org/2000/svg"><path d="M19 3c-1.654 0-3 1.346-3 3 0 .502.136.968.354 1.385l-1.116 1.302A3.976 3.976 0 0 0 13 8c-.739 0-1.425.216-2.02.566L9.566 7.152A3.449 3.449 0 0 0 10 5.5C10 3.57 8.43 2 6.5 2S3 3.57 3 5.5 4.57 9 6.5 9c.601 0 1.158-.166 1.652-.434L9.566 9.98A3.972 3.972 0 0 0 9 12c0 .997.38 1.899.985 2.601l-1.692 1.692.025.025A2.962 2.962 0 0 0 7 16c-1.654 0-3 1.346-3 3s1.346 3 3 3 3-1.346 3-3c0-.476-.121-.919-.318-1.318l.025.025 1.954-1.954c.421.15.867.247 1.339.247 2.206 0 4-1.794 4-4a3.96 3.96 0 0 0-.439-1.785l1.253-1.462c.364.158.764.247 1.186.247 1.654 0 3-1.346 3-3s-1.346-3-3-3zM7 20a1 1 0 1 1 0-2 1 1 0 0 1 0 2zM5 5.5C5 4.673 5.673 4 6.5 4S8 4.673 8 5.5 7.327 7 6.5 7 5 6.327 5 5.5zm8 8.5c-1.103 0-2-.897-2-2s.897-2 2-2 2 .897 2 2-.897 2-2 2zm6-7a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg> **Exécuter une action de workflow** : Exécuter des actions prédéfinies sur les alertes du groupe.
- :material-shield-check-outline: **Marquer comme non malveillante** : Étiqueter l'alerte comme non menaçante.
- :material-bug-outline: **Marquer comme malveillante** : Étiqueter l'alerte comme menace confirmée.
</div>
<div class="grid" markdown style="align-content: center">
![Détail d'alerte](../../../user_manual/assets/triage_alert_detail.png)
</div>
</div>

## Gestion des workflows

Vous pouvez trier automatiquement les alertes dès leur génération en créant des workflows.

Cela peut être un moyen efficace d'automatiser le triage de requêtes à forte confiance, réduisant la fatigue liée aux alertes pour les analystes.

![Voir les workflows](../../../user_manual/assets/triage_view_workflows.png)

### Créer des workflows

Vous pouvez cliquer sur le bouton "Add Workflow" (:material-plus-circle-outline:) pour créer un nouvel objet Workflow, qui nécessitera :

- **Name** : Un nom pour le workflow, afin de faciliter sa revue.
- **Query** : Quelles alertes doivent correspondre à ce workflow ?
- **Labels** : Une liste d'étiquettes à assigner aux alertes correspondant à la requête.
- **Priority** : Comment prioriser les alertes correspondant à ce workflow ?
- **Status** : Quel doit être l'état de l'alerte lors d'une correspondance ?

Vous pouvez optionnellement cocher la case pour appliquer votre workflow aux alertes existantes. Pour enregistrer, cliquez sur le bouton :material-check:.

### Gérer les workflows

La page fournit deux filtres préprogrammés :

1. :material-calendar-remove-outline: **Afficher les workflows jamais utilisés** : Workflows n'ayant aucun enregistrement d'application aux alertes émises par le système.
2. :material-calendar: **Afficher les workflows non utilisés au cours des 3 derniers mois** : Workflows non utilisés durant les 90 derniers jours.

Les deux requêtes sont utiles pour faire le ménage des workflows qui ne sont plus pertinents ou qui doivent être ajustés,
car ils peuvent se déclencher trop souvent ou trop rarement, entraînant un triage d'alertes inefficace.

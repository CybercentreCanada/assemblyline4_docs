# Révision de l'analyse

Après qu'un fichier est soumis à l'analyse et traité, vous pouvez examiner les résultats dans l'interface utilisateur.

## Vue d'ensemble de la soumission

!!! note
    Les captures d'écran présentées dans cette section sont basées sur l'analyse de [cet échantillon de MalwareBazaar](https://bazaar.abuse.ch/sample/d6d876c7327482a6293fb5014393ace99e14aa7e0638bbda9fc602d35b8a72c9/) avec Assemblyline prêt à l'emploi. Vos résultats d'analyse peuvent varier selon la configuration de votre système, les services activés et les jeux de règles importés.

### Vue Rapport

Assemblyline fournit une vue synthétique de l'analyse axée sur les informations les plus importantes du fichier. Cela inclut les métadonnées du fichier (nom, taille, type et hachages), ainsi qu'une vue d'ensemble des résultats d'analyse (verdict, score et attribution).

Cette vue n'est accessible que lorsque l'analyse est terminée. Sinon, le site vous redirigera vers la vue détaillée où vous pourrez suivre la progression de l'analyse en temps réel.

!!! tip "Enregistrer en PDF"
    Vous pouvez imprimer ou enregistrer le rapport en PDF en utilisant le bouton d'icône d'imprimante en haut à droite du rapport.

<video controls src="../../../user_manual/assets/submission_report_view.mp4" title="Vue Rapport" autoplay></video>

### Vue détaillée

Pour un examen plus approfondi de l'analyse, vous pouvez accéder à la vue détaillée. Cette vue fournit des informations complètes sur le fichier, y compris toutes les métadonnées, les résultats de chaque service ayant analysé le fichier, ainsi que toute information ou tout contexte supplémentaire pertinent pour comprendre les résultats d'analyse.

<video controls src="../../../user_manual/assets/submission_detail_view.mp4" title="Vue détaillée" autoplay></video>

#### Informations de soumission

Cette section conserve un enregistrement des détails de la soumission tels que l'heure de soumission, l'utilisateur ayant soumis le fichier et les paramètres utilisés pour l'analyse.

<video controls src="../../../user_manual/assets/submission_submission_information.mp4" title="Informations de soumission" autoplay></video>

#### Informations d'analyse

??? tip "Analyse interactive prise en charge"
    Certains éléments prennent en charge des interactions, comme le clic gauche pour surligner toutes les occurrences de la donnée dans les résultats d'analyse, et le clic droit pour ouvrir un menu contextuel avec des options permettant de rechercher cette donnée dans le système ou de la copier dans votre presse-papiers.

    <video controls src="../../../user_manual/assets/submission_interactive_data.mp4" title="Éléments interactifs" autoplay></video>

##### Attribution

L'attribution représente les associations que le système peut établir entre le fichier et des entités connues, telles que des familles de malwares, des acteurs de menace ou des campagnes, que nous classons à l'aide du framework ATT&CK. L'attribution provient des signatures YARA (lorsque le tag d'acteur est fourni dans les métadonnées de la règle) et des noms de fournisseurs antivirus.

##### Arborescence des fichiers

L'arborescence des fichiers vous permet de naviguer dans le contenu du fichier racine et de ses fichiers enfants, puis d'examiner les résultats d'analyse pour chaque fichier individuel.

Ceci est particulièrement utile pour analyser des fichiers packés ou obfusqués, car cela fournit une trace claire du processus d'analyse à travers les étapes de dépaquetage ou de désobfuscation.

![Arborescence des fichiers](../../../user_manual/assets/submission_file_tree.png)

## Détails du fichier

Lorsque vous sélectionnez un fichier à examiner, l'interface ouvre un panneau contenant une vue détaillée des résultats d'analyse de ce fichier.

<video controls src="../../../user_manual/assets/file_detail.mp4" title="Analyse de fichier" autoplay></video>

### Actions sur le fichier

Le coin supérieur droit de la vue de détail du fichier contient une série de boutons d'action :

 - :material-view-carousel-outline: **Soumissions liées** : Trouver toutes les soumissions qui incluent ce fichier
 - :material-download-outline: **Télécharger** : Télécharger le fichier. Par défaut, il est empaqueté au format [CaRT](https://pypi.org/project/cart/) afin d'éviter une exécution accidentelle
 - :material-file-find-outline: **Visionneuse de fichier** : Ouvrir la visionneuse intégrée (ASCII, Strings, Hex)
 - :material-replay: **Resoumettre** : Resoumettre le fichier pour une nouvelle analyse
 - :material-shield-check-outline: **Safelist** : Ajouter le fichier à la safelist système
 - :material-bug-outline: **Badlist** : Ajouter le fichier à la badlist système

### Visionneuse de fichier

L'interface possède une visionneuse de fichiers intégrée qui prend en charge une grande variété de formats, vous permettant d'examiner le contenu du fichier directement dans l'interface sans avoir à le télécharger et l'ouvrir avec une application externe.

Cela vous permet d'inspecter le contenu du fichier tout en consultant les résultats d'analyse, ce qui peut vous aider à comprendre le contexte des données générées et à prendre des décisions éclairées.

<video controls src="../../../user_manual/assets/file_viewer.mp4" title="Visionneuse de fichier" autoplay></video>

### Fréquence du fichier

La section Fréquence du fichier indique combien de fois ce fichier a été vu dans le système, ainsi que les horodatages de première et de dernière observation. Ces valeurs sont affectées par la période de rétention des données configurée sur le système.

### Tags et heuristiques générés

Cette section est similaire à la section "Tags" et "Heuristics" de la soumission, mais elle se concentre spécifiquement sur les données générées à la suite de l'analyse du fichier observé.

Cliquer sur un tag mettra en évidence le service dont il provient, ce qui facilite la traçabilité de l'extraction d'un indicateur spécifique.

### Résultats des services

C'est ici que vous pouvez examiner les résultats de chaque service ayant analysé le fichier. Les services sont listés par ordre alphabétique et vous pouvez développer chaque service pour voir les détails de ses résultats d'analyse.

Un service qui a analysé le fichier peut avoir trois issues possibles : résultats vides, résultats d'erreur ou résultats générés.

=== "Résultats générés"

    Chaque résultat de service est une composition de sections et sous-sections qui organisent les données générées par le service. La structure de ces sections peut varier fortement d'un service à l'autre, puisque chaque service est conçu pour analyser différents aspects du fichier et produire différents types de données.

    Les sections de résultat peuvent contenir des "tags" (:material-label-outline:) utilisables pour le pivoting et des "heuristics" (:material-sim-outline:) qui indiquent les comportements relevés par le service.

    Lorsqu'un service est mis à jour, les résultats en cache d'une version précédente ne sont plus utilisés. Si plusieurs versions de résultats sont disponibles pour un service, une liste déroulante apparaît pour vous permettre de comparer les résultats d'anciennes exécutions d'analyse.

    ![Résultats générés](../../../user_manual/assets/file_service_result.png)

=== "Résultats vides"

    Cette section indique que le service n'a généré aucun résultat pour le fichier. Cela peut être dû au fait que le service n'a rien trouvé d'intéressant dans le fichier.

    ![Résultats vides](../../../user_manual/assets/file_service_emptyresult.png)

=== "Résultats d'erreur"

    Cette section indique qu'une erreur est survenue pendant l'analyse du fichier par le service. Cela peut être dû à diverses raisons, comme un dépassement de délai, un problème du service lui-même ou un problème du fichier ayant provoqué l'échec du service.

    Cette section peut aussi contenir des avertissements émis par le service, qui ne sont pas nécessairement des erreurs, mais qu'il est important de prendre en compte lors de l'examen des résultats d'analyse. Un exemple d'un tel avertissement pourrait être lié à un tag, comme marquer "abc.com" en tant que "IP", ce qui déclencherait un avertissement de validation interne du système puisque "abc.com" n'est pas une adresse IP valide.

    ![Résultats d'erreur](../../../user_manual/assets/file_service_error.png)

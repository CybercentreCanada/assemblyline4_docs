# Soumettre à Assemblyline

Pour commencer à utiliser Assemblyline pour l'analyse de malwares, la première étape consiste à soumettre un fichier ou une URL pour analyse.

## Soumettre des fichiers

L'interface prend en charge plusieurs méthodes de soumission de fichiers pour analyse, vous offrant la flexibilité de choisir celle qui correspond le mieux à vos besoins. Les méthodes les plus courantes sont :

### Téléversement de fichier

Vous pouvez glisser-déposer des fichiers directement dans la zone de dépôt de l'interface, ou cliquer pour sélectionner des fichiers de votre système local à téléverser pour analyse sous l'onglet "File".

![Téléversement de fichier](../../../user_manual/assets/input_upload.png)

### Utiliser des identifiants

Vous pouvez également soumettre des fichiers pour analyse en fournissant leurs hachages (MD5, SHA1 ou SHA256) si le fichier a déjà été analysé ou est connu du système. Cela peut être fait sous l'onglet "Hash/URL".

Le système prend aussi en charge l'utilisation d'identifiants personnalisés pour récupérer des fichiers depuis des sources de fichiers, mais cela doit être configuré par l'administrateur.

Si votre déploiement dispose de "File Sources" configurées, l'interface tentera de détecter automatiquement le type d'identifiant de fichier que vous soumettez et vous proposera une liste d'options que le système peut utiliser pour récupérer le fichier à analyser. Un exemple de source de fichiers serait VirusTotal ou MalwareBazaar, qui peuvent exploiter les hachages de fichiers pour la récupération.

<video controls src="../../../user_manual/assets/input_file_id.mp4" title="Soumission par hachage" autoplay></video>

### Depuis le presse-papiers

L'interface prend aussi en charge la soumission de fichiers directement depuis votre presse-papiers. Pour le moment, le système vous permet de coller le contenu de votre presse-papiers dans l'onglet "Raw", ce qui vous permet de modifier le contenu avant de le soumettre à l'analyse.

Le plus souvent, cette fonctionnalité est utilisée pour soumettre des extraits de texte ou de code à l'analyse. Par exemple, vous pouvez coller un extrait de code PowerShell dans l'onglet "Raw" et le soumettre à l'analyse. Le système traitera le contenu collé comme un fichier et l'analysera en conséquence.

<video controls src="../../../user_manual/assets/input_raw.mp4" title="Soumission depuis le presse-papiers" autoplay></video>

## Soumettre des URL

Si vous souhaitez soumettre une URL pour analyse, le système vous le permet également sous l'onglet "Hash/URL".

Lorsque vous soumettez une URL, le système crée un fichier URI qui sert de point d'entrée à l'analyse. Assemblyline peut aussi interagir avec des ressources externes, comme récupérer une charge utile de seconde étape via un lien HTTP/HTTPS trouvé dans un fichier malveillant.

Pour que la récupération d'une URI fonctionne correctement, des services pertinents comme URLDownloader doivent être sélectionnés.

### Fournir l'URL

Lorsque vous soumettez une URL pour analyse, le système vous propose une liste de services de la catégorie "Internet Connected" qui peuvent être utilisés pour une récupération spécialisée (p. ex. prise en charge de proxy, user-agents personnalisés, etc.).

![Soumission d'URL](../../../user_manual/assets/input_url.png)

### Utiliser un fichier URI

Un fichier URI est un fichier YAML avec une structure de base comme celle-ci :

```yaml
# Assemblyline URI file
uri: <scheme>://<host>
```

C'est un fichier qui peut contenir plus d'éléments pour des cas d'usage où les services peuvent les exploiter. Les parties les plus importantes sont la clé "uri" du fichier yaml, qui doit être une URI valide avec un schéma et un hôte, et le commentaire en haut, qui aide à l'identification. Le schéma sera utilisé pour créer le type de fichier. Ainsi, si vous utilisez `uri: http://site.com`, ce sera un fichier de type `uri/http`, et si vous utilisez `uri: ftp://site.com`, vous aurez un `uri/ftp`. Cela permet de router vers différents services selon le schéma. Dans le cas de `uri/ftp`, vous aurez probablement besoin d'informations supplémentaires, comme des clés yaml telles que `passive: True`.

Voici un exemple plus complet et plus complexe de fichier URI :

```yaml
# Assemblyline URI file
uri: https://mb-api.abuse.ch/api/v1/
data: query=get_info&hash=52307f9ce784496218f2165be83c2486ad809da98026166b871dc279d40a4d1f
headers:
  Content-Type: application/x-www-form-urlencoded
method: POST
```

Le type de fichier sera `uri/https` et les autres clés yaml seront ignorées lors de l'identification. Les clés supplémentaires du fichier yaml peuvent être utilisées par le service qui traite ce fichier spécifique afin d'offrir un comportement plus personnalisé, plus proche de la demande utilisateur. Un user-agent spécifique, un referer ou d'autres en-têtes peuvent être utilisés pour récupérer une seconde étape sur un serveur nécessitant des valeurs particulières. Grâce à ces valeurs supplémentaires, URLDownloader prend désormais en charge d'autres méthodes comme POST. Un simple changement de `query=get_info` à `query=get_file` dans `data`, et le service devrait télécharger ce fichier depuis MalwareBazaar.

Comme les fichiers URI sont très spécifiques à Assemblyline, nous prenons le temps de réécrire tout fichier entrant de sorte que le commentaire `# Assemblyline URI file` soit sur la première ligne, puis la clé uri, puis toutes les autres clés par ordre alphabétique. Cela permet de dédupliquer les fichiers "identiques" et d'utiliser le cache. Une clé comme `extra_key: ["first", "second", "third", "fourth"]` conservera son ordre et sera convertie ainsi :

```yaml
extra_key:
- first
- second
- third
- fourth
```


## Personnaliser l'analyse

En plus de fournir l'entrée d'analyse, vous pouvez aussi définir les paramètres de votre analyse pour obtenir les résultats les plus pertinents du système. Cela inclut la sélection des services à utiliser pour l'analyse et la configuration de paramètres spécifiques pour ces services.

### Profils de soumission

Ce sont des profils d'analyse définis par les administrateurs de votre système, conçus pour des types d'analyse spécifiques. Par exemple, il peut exister un profil de soumission pour analyser des pièces jointes d'e-mail, incluant les services pertinents pour ce type de fichiers.

L'équipe Assemblyline fournit certains profils de soumission par défaut, conçus pour des types d'analyse courants.

![Profils de soumission](../../../user_manual/assets/params_profiles.png)

### Paramètres de soumission

Pour celles et ceux qui connaissent davantage le système et souhaitent plus de contrôle sur l'analyse, vous pouvez définir vos propres paramètres de soumission. Cela vous permet de choisir les services à utiliser et de configurer leurs paramètres spécifiques.

Pour afficher les paramètres de soumission, cliquez sur l'icône :material-tune: à côté du bouton "Submit".

<video controls src="../../../user_manual/assets/params_submission_parameters.mp4" title="Paramètres de soumission" autoplay></video>

Le tableau ci-dessous fournit une vue d'ensemble des différents types de paramètres que vous pouvez définir pour votre analyse :

| Section | Description | Exemple |
|:--: |:--|:--:|
| Paramètres système | Ce sont des paramètres utilisés par le noyau du système pour définir comment l'analyse doit être effectuée. Par exemple, vous pouvez spécifier la priorité de l'analyse, ce qui peut affecter sa rapidité et les ressources allouées.<br><br>La section "Submission Data" est un emplacement courant pour insérer des données éphémères exploitables par tous les services. Cela peut être utile pour définir un mot de passe ou un jeton que les services utiliseront pendant l'analyse, plutôt que de le configurer service par service.<br><br>Pour une liste complète des paramètres système et de leurs descriptions, veuillez consulter le [modèle Submission Parameters](../../../odm/models/submission/#submissionparams). | <video controls src="../../../user_manual/assets/params_system_parameters.mp4" title="Sélection de services" autoplay></video> |
| Sélection de services | La sélection de services vous permet de choisir les services que vous souhaitez utiliser pour l'analyse.<br><br>Les services sont classés par fonctionnalité (analyse statique, analyse dynamique, extraction, etc.), vous pouvez donc sélectionner tous les services d'une catégorie en cliquant sur le nom de la catégorie, ou choisir des services spécifiques. | <video controls src="../../../user_manual/assets/params_service_selection.mp4" title="Sélection de services" autoplay></video> |
| Paramètres de service | Ce sont des paramètres propres à chaque service, qui permettent de configurer son comportement d'analyse. Ces paramètres varient fortement selon le service et peuvent inclure le niveau d'analyse, des réglages spécifiques, ou tout autre paramètre requis par le service.<br><br>Par exemple, si vous souhaitez analyser un fichier protégé par mot de passe, vous pouvez fournir ce mot de passe en paramètre de service pour qu'Extract l'utilise. | <video controls src="../../../user_manual/assets/params_service_parameters.mp4" title="Paramètres de service" autoplay></video> |

## Vérifier les analyses existantes

Une fois l'entrée d'analyse fournie, vous pouvez demander au système de vérifier s'il existe déjà une analyse correspondante pour cette entrée.

Cela peut être fait en cliquant sur l'icône :material-magnify: à côté du bouton "Submit".
<video controls src="../../../user_manual/assets/input_check_existing.mp4" title="Vérifier les analyses existantes" autoplay></video>

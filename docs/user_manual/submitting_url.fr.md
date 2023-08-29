# Soumission d'un lien URL pour l'analyse

## Soumission
Soumettre un lien URL pour analyse est très similaire à la soumission de fichier; cela peut être fait directement dans l'interface graphique web d'Assemblyline. Pour l'automatisation et l'intégration il est possible d'utiliser l'API [REST API](../../integration/python/#submit-a-file-url-or-sha256-for-analysis).

![Soumission de fichier](./images/submit.fr.png)

Cliquer sur l'onglet "URL/SHA256".

![Soumission d'URL/SHA256](./images/submit_url.fr.png)

### Partage et classification
Si votre systême est configuré avec le contrôle de partage(TLP) ou la classification  de configuration, les restrictions disponibles peuvent être selectionné en cliquant sur la bannière de classification.

### Choisir un URL pour analyse
Contrairement à la soumission par fichier où un fichier est glisser ou selectionné du disque, il suffit d'entrer dans la zone de texte d'entrée le lien URL que vous voulez analyser en le copiant/collant, puis cliquer sur "ANALYSER"!

### Note important par rapport aux soumissions URL avec Assemblyline
Puisqu'Assemblyline est un système de triage de fichier malveillant, il commence chaque soumission avec un fichier, même si vous soumettez un URL. La façon dont cela fonctionne est que lorsque vous soumettez un URL pour analyse, Assemblyline fera une requête de connection au URL soumit, téléchargera la ressource qui est hébergée à cet endroit et soumettra ce fichier pour analyse. S'il n'y a plus de ressource à cet endroit, ou si Assemblyline ne peut pas se connecter au server où le URL pointe, la soumission échouera.

Ceci est important car si vous avez un URL qui héberge un fichier malveillant et que vous ne voulez pas exposer votre système Assemblyline au serveur duquel le URL pointe, il est recommandé de configurer un serveur proxy qui agira comme intermédiaire entre l'architecture d'Assemblyline et le serveur qui héberge un fichier malveillant. Vous pouvez configurer ce paramètre dans votre déploiement k8s sous la section `ui`: https://cybercentrecanada.github.io/assemblyline4_docs/odm/models/config/#ui. Le paramètre en question est `url_submission_proxies`.

## Options
Options de soumission additionels non limité à:

- Choisir quelle(s) catégorie(s) de services ou quel(s) service(s) spécifique(s) à utiliser pour l'analyse
- Spécifier les paramètres de soumission des services (example: indiquer un mot de passe à utiliser, ou encore un temps limite pour l'analyse dynamique)
- Ignorer la filtration des services: Ne pas prendre compte des services de la liste sûre
- Ignoner les résultats en cache: Forcer la re-soumission même si le même fichier a été analyser récemment avec les mêmes versions de service
- Ignorer la prévention de la récurssion dynamique: Déactiver l'itération limite sur un fichier
- Profiler l'analyse courante
- Effectuer une analyse approfondie: Permet un decortiquate maximal (**Grandement recommendé pour les fichiers connus malveillants ou très suspicieux afin de détection le contenu camouflé**)
- Temps de vie: Temps (en jours) avant que le fichier soit effacé du système.

![Options de soumission](./images/submit_options.fr.png)

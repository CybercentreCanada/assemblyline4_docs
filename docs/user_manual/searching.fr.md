# Recherche

Assemblyline tire avantage des puissantes capacités de recherche pratiquement infinies d'[Elasticsearch](https://www.elastic.co/).

## Banque de documents

Les index d’information sont un des principaux concepts à approfondir. Ils permettent à Assemblyline de dédupliquer la plupart des résultats dans le système, ce qui explique pourquoi l’outil peut s’adapter aussi efficacement. Les recherches dans les champs indexés s’effectuent très rapidement.

Assemblyline dispose de six indices principaux:

-   **[Alerte](../odm/models/alert.md)**: Permet aux utilisateurs d'effectuer des recherches détaillées sur les alertes afin d'identifier et de prioriser rapidement les incidents de sécurité, en tenant compte de divers attributs tels que les indicateurs de menace, la classification et les horodatages.

-   **[Fichier](../odm/models/file.md)**: Permet aux utilisateurs de rechercher des fichiers spécifiques dans une soumission, d'identifier les doublons et de recueillir des informations contextuelles sur les propriétés d'un fichier, telles que sa classification, son entropie et les valeurs de hachage observées.

-   **[Résultat](../odm/models/result.md)**: Permet aux utilisateurs de rechercher des résultats de service spécifiques, ce qui leur permet d'examiner les analyses effectuées par divers services, y compris les scores détaillés, les sections et les données de réponse.

-   **[Retrohunt](../odm/models/retrohunt.md)**: Permet aux utilisateurs de rechercher les résultats rétrospectifs de la chasse aux menaces à partir des règles Yara appliquées à des échantillons précédemment soumis. Cela facilite l'identification et l'analyse des nouvelles menaces sur la base des informations actualisées sur les menaces.

-   **[Signature](../odm/models/signature.md)**: Permet aux utilisateurs de rechercher des signatures spécifiques à un service (par exemple, des règles YARA) et toutes les métadonnées pertinentes, y compris la source, les statistiques, la classification et le statut.

-   **[Soumission](../odm/models/submission.md)**: Permet aux utilisateurs de gérer et de suivre les soumissions, de consulter les fichiers concernés, les erreurs d'analyse, les scores maximaux et le statut du cycle de vie de la soumission, ce qui offre une vue d'ensemble du processus d'analyse.

Vous pouvez afficher tous les index et les champs indexés connexes après avoir assuré le bon fonctionnement d’Assemblyline en consultant l’aide sous `Help > Search help menu`.

## Recherche de comportements et limitations

Lors d’une recherche dans l’interface utilisateur, la requête est transmise dans tous les index et renvoie tous les résultats correspondants.

!!! tip "Vous devez limiter vos critères de recherche à un seul indexe. En d’autres mots, vous ne pouvez pas rechercher de l’information dans deux index différents ou plus."

Il est possible de contourner cette limitation en faisant appel à l’API REST. La méthode consiste alors à effectuer une recherche dans un indexe, puis à l’élargir ou l’affiner en recherchant ces éléments dans d’autres index.

## Exemples de Recherche

Un moyen rapide de se familiariser avec les index de recherche consiste à utiliser l'élément "*Trouver les résultats associés*" dans le menu déroulant des balises.

![Searching](./images/magnifier.png){: .center }

Cliquer sur l'élément "*Trouver les résultats associés*" sur la balise `av.virus_name` (`HEUR/Macro.Downloader.MRAA.Gen`) générera la requête suivante :

```ruby
result.sections.tags.av.virus_name:"HEUR/Macro.Downloader.MRAA.Gen"
```

Vous pouvez également générer des recherches plus complexes en utilisant une syntaxe de requête complète. En voici quelques exemples :

```ruby
# Trouver tous les résultats où le service ViperMonkey a extrait l’adresse IP 10.10.10.10
result.sections.tags.network.static.ip:"10.10.10.10" AND response.service_name:ViperMonkey

# Trouver toutes les soumissions pour lesquelles une note de 2000 ou plus a été attribuée
# au cours des deux derniers jours
max_score:[2000 TO *] AND times.submitted:[now-2d TO now]

# Trouver tous les résultats de l’antivirus dont le nom de signature correspond à Emotet
result.sections.tags.av.virus_name:*Emotet*
```

Le système prend en charge un large éventail de paramètres de recherche, comme les caractères génériques, les plages et les expressions régulières. Vous trouverez la syntaxe complète dans l’aide sous ```Help > Search Help```.

Les requêtes de recherche peuvent également être utilisées dans le [client d’Assemblyline](../../integration/python) pour instaurer un puissant mécanisme qui s’exécutera automatiquement lorsque de nouveaux fichiers seront analysés par le système.

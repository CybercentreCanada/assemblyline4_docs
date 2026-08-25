# Recherche et pivotement

Assemblyline indexe les données qu’il génère pendant l’analyse, ce qui vous permet d’effectuer des recherches dans les soumissions, les fichiers, les résultats, les signatures, les alertes et les recherches rétrospectives depuis une interface unique, en utilisant la [syntaxe de requête Lucene](https://www.elastic.co/guide/en/kibana/current/lucene-query.html).

## Interface de recherche

### Recherche simple

La barre de recherche située en haut de l’interface vous permet d’effectuer rapidement une recherche dans tous les index à la fois. Les résultats sont regroupés par index, vous pouvez donc sélectionner un onglet pour afficher les correspondances de l’index concerné.

<video controls src="../../../user_manual/assets/searching_general.mp4" title="Recherche"autoplay loop></video>

### Recherche dans un index spécifique

Pour effectuer des recherches plus ciblées, utilisez la page de recherche dédiée à chaque index. Vous pouvez y accéder en cliquant sur l’icône de recherche, puis en sélectionnant l’index souhaité.

Vous pouvez consulter tous les index disponibles ainsi que leurs champs interrogeables dans **Aide > Aide à la recherche** de votre instance Assemblyline.

Un index est un ensemble de données connexes qui peut être interrogé indépendamment. Assemblyline gère les index suivants :

| Index | Éléments recherchables |
|---|---|
| [Alerte](../../odm/models/alert) | Alertes déclenchées pendant l’analyse, utiles pour trier et hiérarchiser les incidents de sécurité |
| [Fichier](../../odm/models/file) | Fichiers présents dans toutes les soumissions, recherchables par hachage, type, entropie, classification, etc. |
| [Résultat](../../odm/models/result) | Résultats des services, notamment les scores, les sections extraites et les données de réponse des services individuels |
| [Signature](../../odm/models/signature) | Signatures de services telles que les règles YARA, avec leur source, leur état et leurs statistiques |
| [Soumission](../../odm/models/submission) | Enregistrements de soumission, permettant de suivre les fichiers concernés, les erreurs, les scores maximaux et l’état du cycle de vie |

Les recherches sont limitées à un seul index : les requêtes inter-index (JOIN) ne sont pas prises en charge par Elasticsearch.

#### Construction des requêtes

Lorsque vous consultez un index spécifique, la barre de recherche suggère les noms de champs disponibles au fur et à mesure de votre saisie, ce qui facilite la construction de requêtes valides. Vous pouvez combiner des champs, des valeurs, des caractères génériques et des plages pour créer des requêtes complexes.

Par exemple, imaginons que vous souhaitiez trouver tous les résultats dans lesquels une adresse IP a été extraite. Vous pouvez accéder à la page de recherche **Résultat** et exécuter une requête telle que :

```
result.sections.tags.network.static.ip:*
```

<video controls src="../../../user_manual/assets/searching_specific.mp4" title="Recherche d’adresses IP"autoplay loop></video>

## Trouver des résultats associés sans effectuer de recherche

Lorsque vous consultez une étiquette dans l’interface, vous pouvez cliquer dessus avec le bouton droit pour ouvrir un menu contextuel contenant l’option « Trouver les résultats associés ».

Cette action génère et exécute automatiquement une requête de recherche pour la valeur de cette étiquette dans l’ensemble du système. Vous pouvez ainsi trouver rapidement toutes les données associées sans avoir à construire manuellement une requête.

<video controls src="../../../user_manual/assets/searching_related_results.mp4" title="Trouver les résultats associés"autoplay loop></video>

## Automatisation des recherches

Les requêtes peuvent être exécutées automatiquement à l’aide du [client Assemblyline](../../integration/python/). Cette méthode est utile pour automatiser les recherches répétitives ou enrichir les résultats en enchaînant des requêtes sur plusieurs index.

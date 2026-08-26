# Model Context Protocol (MCP)

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/docs/latest) est un standard open source permettant de connecter des applications d’IA à des systèmes externes.

Il est particulièrement utile si vous disposez d’agents ou de systèmes d’orchestration d’IA capables d’interagir avec Assemblyline.

## Méthodes d’authentification prises en charge

### Authentification OAuth2 et On-Behalf-Of

!!! tip "Recommandé pour une utilisation en production"
    Cette méthode d’authentification est recommandée pour les environnements de production dans lesquels des applications doivent accéder à Assemblyline au nom des utilisateurs.

Si vous utilisez l’authentification On-Behalf-Of (OBO), vous pouvez mettre en place des pratiques de sécurité telles que l’autorisation incrémentielle, ce qui n’est pas possible avec les clés API :
cela consiste à associer des portées OAuth personnalisées aux rôles Assemblyline afin de créer des jetons offrant un contrôle d’accès précis (consultez la section [Authentification OAuth](../../installation/configuration/authentication/#incremental-authorization-and-fine-grained-access-control) pour plus d’informations).

L’autorisation incrémentielle signifie qu’un agent ou un client commence avec un ensemble de permissions limité, par exemple la lecture des alertes, puis élève ses permissions uniquement lorsque cela est nécessaire, par exemple pour effectuer le triage des alertes.

Pour effectuer une action nécessitant des permissions élevées, l’agent ou le client doit demander des permissions supplémentaires à l’utilisateur. Cette approche permet de renforcer la sécurité et le contrôle des ressources,
car elle réduit les risques liés aux agents disposant de privilèges excessifs et garantit que les utilisateurs sont informés des actions effectuées en leur nom, en imposant une intervention humaine ([human-in-the-loop](https://www.ibm.com/think/topics/human-in-the-loop)).

### Clé API

!!! tip "Recommandé pour le développement et l’utilisation personnelle"
    Cette méthode d’authentification constitue une solution adaptée aux environnements de développement et de test. Elle n’est pas recommandée en production et peut entraîner une exposition des données si elle n’est pas correctement gérée.

Son fonctionnement est similaire à celui de l’API d'Assemblyline, qui permet d’utiliser une clé API pour s’authentifier et autoriser l’accès au système. Cette méthode est plus simple à mettre en œuvre, mais ses permissions sont fixes et elle ne prend pas en charge l’autorisation incrémentielle.

## Connexion au MCP

Lorsqu’il est déployé, le serveur est configuré pour écouter les requêtes entrantes sur le point de terminaison `/mcp` de votre instance Assemblyline. Au moment de la rédaction de cette documentation, le serveur transmet les appels au point de terminaison `/api/v4` au nom du client.

!!! example "Intégration à VS Code"
    En suivant la [configuration JSON du MCP](https://gofastmcp.com/v3/integrations/mcp-json-configuration), vous pouvez utiliser la configuration `mcp.json` suivante dans VS Code afin de donner à Copilot l’accès à votre instance Assemblyline.

    === "OAuth2"
        ```json
        {
            "servers": {
                "assemblyline": {
                    "url": "http://localhost/mcp",
                    "type": "http",
                    "headers": {
                        # En-têtes standard pour l’authentification OAuth2
                        "Authorization": "Bearer <oauth2 token>"
                    }
                }
                ...
            }
        }
        ```

    === "Clé API"
        ```json
        {
            "servers": {
                "assemblyline": {
                    "url": "http://localhost/mcp",
                    "type": "http",
                    "headers": {
                        # En-têtes personnalisés pour l’authentification par clé API
                        "X-APIKEY": "<api key>",
                        "X-USER": "<username>"
                    }
                }
                ...
            }
        }
        ```

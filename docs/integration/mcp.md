# Model Context Protocol (MCP)

[MCP (Model Context Protocol)](https://modelcontextprotocol.io/docs/latest) is an open-source standard for connecting AI applications to external systems.

This is convenient if you have AI agents or harnesses that you'd like to use to interact with Assemblyline.

## Supported Authentication Methods

### OAuth2 & On-Behalf-Of Authentication

!!! tip "Recommended for Production Use"
    This is the recommended authentication method for production environments where you have applications that need to access Assemblyline on behalf of users.

If you're using On-Behalf-Of (OBO) authentication, then you can implement security practices such as incremental authorization, which is not possible with API keys:
this would involve binding custom OAuth scopes to Assemblyline roles to create tokens with fine-grained access control (See [OAuth Authentication](../../installation/configuration/authentication/#incremental-authorization-and-fine-grained-access-control) for more information).

Incremental authorization is the idea that an agent/client starts with a limited set of permissions (ie. read alerts), and then escalates permissions but only when needed (ie. triage alerts).

In order to perform an action that requires elevated permissions, the agent/client must request the additional permissions from the user. This allows for a more secure and controlled access to resources,
as it reduces the risk of over-privileged agents and ensures that users are aware of what actions are being performed on their behalf by enforcing [human-in-the-loop](https://www.ibm.com/think/topics/human-in-the-loop).

### API Key

!!! tip "Recommended for Development and Home Use"
    This is an alternative authentication method that is suitable for development and testing environments. It is not recommended for production use and can lead to data exposure if not handled properly.

This is similar to the way that the Assemblyline API works, where you can use an API key to authenticate and authorize access to the system. This method is simpler to implement but has fixed permissions and does not support incremental authorization.

## Connecting to the MCP

The server is set up to listen for incoming requests on the `/mcp` endpoint on your Assemblyline instance when deployed. At the time of writing, the server will invoke calls to the `/api/v4` endpoint on behalf of the client.

!!! example "VSCode Integration"
    Following the [MCP JSON Configuration](https://gofastmcp.com/v3/integrations/mcp-json-configuration), you can use the following `mcp.json` configuration in VSCode to give Copilot access to your Assemblyline instance.

    === "OAuth2"
        ```json
        {
            "servers": {
                "assemblyline": {
                    "url": "http://localhost/mcp",
                    "type": "http",
                    "headers": {
                        # Standard headers for OAuth2 authentication
                        "Authorization": "Bearer <oauth2 token>"
                    }
                }
                ...
            }
        }
        ```

    === "API Key"
        ```json
        {
            "servers": {
                "assemblyline": {
                    "url": "http://localhost/mcp",
                    "type": "http",
                    "headers": {
                        # Custom headers for API key authentication
                        "X-APIKEY": "<api key>",
                        "X-USER": "<username>"
                    }
                }
                ...
            }
        }
        ```

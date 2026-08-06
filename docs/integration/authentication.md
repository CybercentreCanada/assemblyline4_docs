# Client Authentication

The API supports different ways of authenticating as a user of the system. Your choice of authentication method will depend on the use-case of the integration.

## OAuth Access Tokens

These are tokens issued by the OAuth provider for a particular user. These kinds of tokens can be passed in the `Authorization` header as a Bearer token and the API should be able to reason which OAuth provider the token corresponds to based on the `issuer` claim.

If you're already in possession of an OAuth token in your environment (ie. JupyterLab session), this is a simple way of forwarding the same token to authenticate with Assemblyline (provided the token audiences can be accepted by Assemblylines).

``` mermaid
sequenceDiagram
  autonumber
  actor Client
  participant KeyCloak as OAuth Provider
  participant Assemblyline as Assemblyline API

  Client->>KeyCloak: Request access token for "user" (ie. grant_type=password)
  KeyCloak->>Client: Return access token
  Client->>Assemblyline: POST /api/v4/auth/login/ (access token from KeyCloak)
```

### On-Behalf-Of (OBO)

These are tokens issued after exchanging an access token representing a user from one audience to another. Like normal access tokens, these can be passed in the `Authorization` header as a Bearer token as long as the audience is intended for Assemblyline.

These kinds of tokens are ideal for services like [Clue](https://cybercentrecanada.github.io/clue/) where you want the middle-tier server to impersonate the user while making requests to Assemblyline on their behalf. When impersonating a user, Assemblyline will audit these interactions based on the `azp` claim in the exchanged token.

``` mermaid
sequenceDiagram
  autonumber
  actor Client
  participant Clue as Clue API
  participant Assemblyline as Assemblyline API
  participant KeyCloak as OAuth Provider

  Client->>Clue: POST /api/login/ (client signs into Clue)
  Clue->>KeyCloak: Request access token for "user"
  KeyCloak->>Clue: Return access token (token for "user" with audience for Clue is cached for future requests)

  Client->>Clue: POST /enrich/ (enrichment request for "user")
  Clue->>KeyCloak: Perform a token exchange to access Assemblyline on behalf of "user"
  KeyCloak->>Clue: Return OBO access token with the audience for Assemblyline
  Clue->>Assemblyline: GET /api/v4/search/results (OBO access token from KeyCloak inserted into Authorization header)
```

## API Keys

### Generating an API key

While integrating Assemblyline to another system, you should not save your username and password into another app. Instead, you should create an API Key with only the appropriate requirements for that specific integration.

### Role-based Access Controls

All APIs have role-based access controls (RBAC) and require users to be authenticated to use those APIs through basic authentication like username and password, API keys, certificates, etc.

![RBAC Controls for an Administrator User](./images/rbac_admin.png)

API keys can be defined to have specific restraints that are less than or equal to those imposed on the owner.

The APIs also drive whether information should be made accessible to a user by comparing the classification of the requester against the data asked to retrieve.

### Create an API key

Here is how to do this:

![Key generation](./images/apikey.gif){: .center }

- [x] Login to Assemblyline's user interface with the user that will perform API requests
- [x] Click on your avatar in the top-right corner of the Assemblyline UI and select "Manage Account"
- [x] Scroll down to the bottom to the "Security" section and select "Manage API Keys"
- [x] Add the API Key name, select access privileges then click the "Add" button.
- [x] The API KEY will only be displayed once and can't be recovered. Copy it somewhere safe so that you can use it later.
- [x] Click the "Done" button.

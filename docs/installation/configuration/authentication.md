---
layout: default
title: Authentication
parent: Configuration
grand_parent: Installation
nav_order: 2
has_toc: false
---

# Authentication

Assemblyline comes with a built-in user management database, so no external identity sources is required. 
However in order to facilitate user mangement in larger organisations you can integrate Assemblyline with external identity providers.

Your configuration file location will depend on your deployment type:

<table>
<tr>
<td style="background-color:#009c7b"><text style="color:white;">Appliance deployment</text></td>
<td> edit `$HOME/assemblyline4_beta_4/config/config.yml` </td>
</tr>
<tr>
<td style="background-color:#2869e6"><text style="color:white;">Cluster deployment</text></td>
<td> see <a href="https://github.com/CybercentreCanada/assemblyline-helm-chart/blob/master/assemblyline/values.yaml"> Default helm chart</a> </td>
</tr>
</table>

<hr>

## LDAP Authentication (optional)

You can easily add authentication via your LDAP server by turning on and configuring the LDAP authentication module in the configuration file:

Here is a example configuration block to add to your configuration file that will allow you to connect to the docker-test-openldap server from: https://github.com/rroemhild/docker-test-openldap

    auth:
        internal:
            # Disable internal login, you could also leave it on if you want
            enabled: false
        ldap:
            # Enable LDAP
            enabled: true

            # DN of the group or the user who will get admin priviledges
            admin_dn: cn=admin_staff,ou=people,dc=planetexpress,dc=com
            
            # Auto-create users if they are missing
            auto_create: true
            
            # Auto-sync LDAP values with the internal DB at each login
            auto_sync: false
            
            # URI to the LDAP server
            uri: ldaps://<ldap_ip_or_domain>:636
            
            # Base DN for the users
            base: ou=people,dc=planetexpress,dc=com
            
            # Field name for the uid
            uid_field: uid
            
            # How the group lookup is queried
            group_lookup_query: (&(objectClass=Group)(member=%s))
  
## oAuth Authentication (optional)

Here is a example configuration block to add to your configuration file that will allow you to connect to all 3 supported oAuth providers:

    auth:
        internal:
            # Disable internal login, you could also leave it on if you want
            enabled: false
        oauth:
            # Enable oAuth
            enabled: true

            # Setup the providers (You can just remove the ones you don't want)
            providers: 
                auth0:
                    # It is safe to auto-create users here 
                    # because it is your own oAuth tenant
                    auto_create: true
                    auto_sync: false

                    # Put your client ID and secret here
                    client_id: <YOUR_CLIENT_ID>
                    client_secret: <YOUR_CLIENT_SECRET>
                    
                    # Set your tenant name in the following urls
                    access_token_url: https://<TENANT_NAME>.auth0.com/oauth/token
                    authorize_url: https://<TENANT_NAME>.auth0.com/authorize
                    api_base_url: https://<TENANT_NAME>.auth0.com/
                
                azure_ad:
                    # Be careful when you set auto-create here beecause if you've
                    # set your client ID to allow everyone, you'll give access
                    # to anyone that has a Microsoft account...
                    auto_create: false
                    auto_sync: false

                    # Put your client ID and secret here 
                    client_id: <YOUR_CLIENT_ID>
                    client_secret: <YOUR_CLIENT_SECRET>

                google:
                    # Be careful when you set auto-create here because if you've
                    # set your client ID to allow everyone, you'll give access
                    # to anyone that has a Google account...
                    auto_create: false
                    auto_sync: false

                    # Put your client ID and secret here 
                    client_id: <YOUR_CLIENT_ID>
                    client_secret: <YOUR_CLIENT_SECRET>

# Auto-properties

You can perform select action automatically based on user properties

You have to provide the following information

| field | User attribute (such as email, group) |
| pattern | Regex on the field |
| type | Assemblyline user attribute to set (such as role, classification)|
| value | The value to set the attribute to |

## Example

This example assume you are using TLP:W and TLP:A in your classification engine.

    oauth:
        enabled: true
        gravatar_enabled: false
        providers:
            azure_ad:
                auto_properties:
                    # any user with a @gmail.com email will be given TLP:Amber classification
                    - field: email
                        pattern: .*@gmail\.com$
                        type: classification
                        value: "TLP:A"
                    # any user with in the admins-sg will be made administrator in the system
                    - field: groups
                        pattern: ^admins-sg$
                        type: role
                        value: admin
                    # any user with in the tlpwhiteusers-sg will be given TLP:White classification
                    - field: groups
                        pattern: ^tlpwhiteusers-sg$
                        type: classification
                        value: "TLP:W"
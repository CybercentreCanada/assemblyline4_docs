#---
#layout: default
#title: AssemblyLine client
#nav_order: 1
#parent: User's manual
#has_children: true
#---

# AssemblyLine client
The assemblyline client library facilitates issuing requests to AssemblyLine.

## Pre-requisites

To install the client you'll need to make sure the you have the folowing installed:

    # APT/YUM
    libffi-dev
    libssl-dev

    # pypi
    pycryptodome
    requests
    requests[security]
    python-baseconv
    python-socketio[client]
    socketio-client==0.5.7.4

## Using the client

You can instantiate the client using the following snippet of code:

    # The new v4 client will test connection to detect if the server is v3 or v4. You should now use the get_client method.
    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", auth=('user', 'password'))
    
    # or with an apikey
    
    al_client = get_client("https://localhost:443", apikey=('user', 'key'))
    
    # or with a cert 
    
    al_client = get_client("https://localhost:443", cert='/path/to/cert/file.pem')

    # and if your assemblyline server is using a self-signed cert

    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify=False)
    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify='/path/to/server.crt')

The assemblyline client is fully documented in the docstrings so if you use an interactive client like ipython you can use the help feature.

    al_client.search.alert?
    Signature: al_client.search.alert(query, *args, **kwargs)
    Docstring:
    Search alerts with a lucene query.

    Required:
    query   : Lucene query. (string)

    Search parameters can be passed as key/value tuples or keyword parameters.

    Returns all results.
    File:      /usr/local/lib/python2.7/dist-packages/assemblyline_client/__init__.py
    Type:      instancemethod

### Examples

#### Submit a file

Submitting a file to the system is just as simple as passing the file path

    al_client.submit('/path/to/my/file.txt')

#### Getting a key

To get a key of a given bucket, you simply need to pass it it's ID

    submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")

#### Using search

You can use the search engine in the client by simply passing a lucene query

    search_res = al_client.search.submission("submission.submitter:user")

#### Using search iterator

Instead of using a strait search and getting a page of result, you can use the search iterator to go through all results.

    for submission in al_client.search.stream.submission("submission.submitter:user"):
        # It only return the indexed fields if you want the full thing you need to go get it
        full_submission = al_client.submission(submission['submission.sid'])

        # Then do stuff with full submission (print for example)
        print(full_submission)

#### Using search parameters

##### Version 3
You can pass search parameters for any given query. The following examples a Lucene facet search to get the top users submitting to a server.

    kwargs = {'facet':'on', 'facet.field':'submission.submitter', 'facet.sort':'count', 'facet.limit':50, 'rows':0}  # rows=0 so that only facet results return
    c.search.submission('times.submitted:[NOW-7DAYS TO NOW]', **kwargs)

##### Version 4
Version 4 server will support facet query out of the box, no need to learn the Lucene facetting syntax.
    
    c.search.facet.submission('submission.submitter', query='times.submitted:[NOW-7DAYS TO NOW]')

#### Listen for message instead of querying for data

You can listen on the different message queues and execute a callback on each message.

    def callback(callback_data):
        print callback_data

    al_client.socketio.listen_on_dashboard_messages(callback)

**NOTE**: Depending on the volume of data, you might process a ton of messages!

----------------------------------------------------------------------------------------------

# BibliothÃ¨que cliente dâ€™Assemblyline

La bibliothÃ¨que cliente dâ€™Assemblyline facilite la soumission de demandes Ã  Assemblyline.

## Exigences prÃ©alables

Avant de procÃ©der Ã  lâ€™installation du client, vous devez vous assurer dâ€™installer ce qui suit :

    # APT/YUM
    libffi-dev
    libssl-dev

    # pypi
    pycryptodome
    requests
    requests[security]
    python-baseconv
    python-socketio[client]
    socketio-client==0.5.7.4

## Utilisation du client

Vous pouvez instancier le client au moyen de lâ€™extrait de code suivant :

    # Le nouveau client v4 dÃ©tecte si le serveur est a la version 3 ou 4. Vous devez maintenant utilisÃ© la fonction get_client.
    from assemblyline_client import get_client
    al_client = get_client("https://localhost:443", auth=('user', 'password'))

    # ou dâ€™une clÃ© API :

    al_client = get_client("https://localhost:443", apikey=('user', 'key'))

    # ou dâ€™un certificat :

    al_client = get_client("https://localhost:443", cert='/path/to/cert/file.pem')

    # et si votre server assemblyline a un certificat auto-signÃ©

    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify=False)
    al_client = get_client("https://localhost:443", auth=('user', 'password'), verify='/path/to/server.crt')

Le client dâ€™Assemblyline est pleinement documentÃ© dans les docstrings. Si vous utilisez un client interactif comme ipython, vous serez en mesure dâ€™utiliser la fonction dâ€™aide.

    al_client.search.alert?
    Signature: al_client.search.alert(query, *args, **kwargs)
    Docstring:
    Search alerts with a Lucene query.

    Required:
    query   : Lucene query. (string)

    Search parameters can be passed as key/value tuples or keyword parameters.

    Returns all results.
    File:      /usr/local/lib/python2.7/dist-packages/assemblyline_client/__init__.py
    Type:      instancemethod

### Exemples

#### Soumission dâ€™un fichier

Pour soumettre un fichier au systÃ¨me, il suffit dâ€™envoyer le chemin dâ€™accÃ¨s du fichier.

    al_client.submit('/chemin/acces/de/mon/fichier.txt')

#### Obtention dâ€™une clÃ©

Pour obtenir une clÃ© pour un compartiment donnÃ©, il suffit dâ€™envoyer son ID.

    submission_details = al_client.submission("4nxrpBePQDLH427aA8m3TZ")

#### Utilisation de la recherche

Pour utiliser le moteur de recherche du client, il suffit de transmettre une demande Lucene.

    search_res = al_client.search.submission("submission.submitter:user")

#### Utilisation de lâ€™itÃ©rateur de recherche

PlutÃ´t que dâ€™utiliser une recherche directe et dâ€™obtenir une page de rÃ©sultats, vous pouvez utiliser lâ€™itÃ©rateur de recherche pour passer Ã  travers tous les rÃ©sultats.

    for submission in al_client.search.stream.submission("submission.submitter:user"):
        # Seuls les champs indexÃ©s sont renvoyÃ©s. Pour obtenir les rÃ©sultats dans leur intÃ©gralitÃ©, vous devez y accÃ©der manuellement,
        full_submission = al_client.submission(submission['submission.sid'])

        # puis faire quelque chose avec la soumission complÃ¨te (imprimer, par exemple)
        print(full_submission)

#### Utilisation des paramÃ¨tres de recherche

##### Version 3
Vous pouvez transmettre des paramÃ¨tres de recherche pour une requÃªte donnÃ©e. Les exemples suivants dÃ©montrent une recherche de facettes Lucene pour obtenir les utilisateurs les plus frÃ©quants soumettant Ã  un server.

    kwargs = {'facet':'on', 'facet.field':'submission.submitter', 'facet.sort':'count', 'facet.limit':50, 'rows':0}  # rows=0 pour que seuls les rÃ©sultats de la facette soient retournÃ©
    c.search.submission('times.submitted:[NOW-7DAYS TO NOW]', **kwargs)

##### Version 4
Le serveur version 4 supporte directement les recherches de facettes, vous navez donc pas besoin den apprendre la syntaxe.
    
    c.search.facet.submission('submission.submitter', query='times.submitted:[NOW-7DAYS TO NOW]')
    
#### Lâ€™Ã©coute du message plutÃ´t que la recherche de donnÃ©es

Vous pouvez Ã©couter les diffÃ©rentes files dâ€™attente de messages et effectuer un rappel pour chaque message.

    def callback(callback_data):
        print callback_data

    al_client.socketio.listen_on_dashboard_messages(callback)

**REMARQUE** : Selon le volume de donnÃ©es, vous pourriez traiter une grande quantitÃ© de messages!
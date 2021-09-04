# RESTful API

When it is impossible to integrate your application using the dedicated python client, you can use Assemblyline's RESTful API to perform any tasks that you can think of.

## API documentation

Each instances of Assemblyline comes with it's internal API documentation which can be viewed by browsing to: [https://localhost/help/api](https://localhost/help/api) (Replace localhost by the domain/ip of your deployment)

## Connecting to the API

For easy integration, it is recommended that you generate an API KEY for the user who will perform RESTful queries. Otherwise, you will have to build yourself a library which will handle session cookies and XSRF tokens and you probably want something simpler.

### Using the API KEY 

To use your newly you can simply add the `X-USER` and `X-APIKEY` headers to your request and the system will identify you with that key at each requests instead of relying on a session cookie.

**Exemple**

Lets use an hypotetical API KEY to ask the system who we are. (Using the `/api/v4/user/whoami/` API)


=== "CURL"
    ``` shell 
    curl -X GET "https://localhost/api/v4/user/whoami/" \
         -H 'x-user: <your_user_id>' \
         -H 'x-apikey: <key_name:randomly_generated_password>' \
         -H 'accept: application/json'
    ```

=== "Javascript (fetch)"
    ``` javascript
    fetch(
      "https://localhost/api/v4/user/whoami/", 
      {
        "headers": {
          "accept": "application/json",
          "x-apikey": "<key_name:randomly_generated_password>",
          "x-user": "<your_user_id>"
        },
        "method": "GET"
      }
    );
    ```

=== "Python (requests)"
    ``` python
    import requests
    requests.get(
        "https://localhost/api/v4/user/whoami/", 
        headers={
            "x-user": "<your_user_id>", 
            "x-apikey": "<key_name:randomly_generated_password>", 
            "accept": "application/json"
        }
    )
    ```

## API Gotcha! 

Here is a list of the most common issues user's are facing while using the API.

### Wrong content type

All Assemblyline APIs are built around receiving and returning JSON data. Do not forget to set your `Content-Type` and `Accept` headers to `"application/json"` or you might encounter some issues.

### Trailing forward slash

All Assemblyline APIs end with a trailing forward slash `"/"`. Make sure that the API URL has it at the end of the url otherwise you may get a `"Method not allowed"` error and you'll have issues figuring out why.
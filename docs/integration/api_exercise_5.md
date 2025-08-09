# What About Custom Tradecraft?

## Scenario

> “I can’t use Assemblyline’s Python/Java client to integrate with my existing tradecraft. What can I do?”

In previous exercises, we’ve shown that you can use the Assemblyline client or a language’s network requests library to interact with the Assemblyline’s API, but what about tools like cURL or Postman?

## Expected Results

```
Send files to Assemblyline through Ingest/Submit API using cURL
```

## REST APIs Involved
```
POST /api/v4/submit/
POST /api/v4/ingest/
```

??? info "cURL Cheatsheet"
    ```
    to pass headers: -H 'key: value'
    to set the request type: -X GET
    to stop cert validation: -k
    to add a multipart form data:
          -F 'name=data'
      OR  -F 'name=@path_to_file'

    # Ingest / Submit API cheatsheet
     * JSON parameters to the submission are passed inside a multipart object named 'json'
     * The file binary is passed inside a multipart object named 'bin'

    # ** Tip: you can pipe the curl output to json_pp so you can actually read it
    ```

## Solution

=== "Using Submit API"
    ```bash
    # Send a file for live processing using CURL
    # ** API to use: /api/v4/submit/
    echo "Send to submit API:"
    curl -s -k -X POST https://$AL_HOST/api/v4/submit/ \
        -H "x-user: ${AL_USER}" \
        -H "x-apikey: ${AL_APIKEY}" \
        -H 'accept: application/json' \
        -F 'json={"params": {"description": "My CURL test"}, "metadata": {"any_key": "any_value"}}' \
        -F 'bin=@myfile.txt' | json_pp
    ```
=== "Using Ingest API"
    ```bash
    # Send a file for asynchronous processing using CURL
    # ** API to use: /api/v4/ingest/ (POST)
    echo "Send to ingest API:"
    curl -s -k -X POST https://$AL_HOST/api/v4/ingest/ \
        -H "x-user: ${AL_USER}" \
        -H "x-apikey: ${AL_APIKEY}" \
        -H 'accept: application/json' \
        -F 'json={"params": {"description": "My CURL test"}, "metadata": {"any_key": "any_value"}}' \
        -F 'bin=@myfile.txt' | json_pp
    ```

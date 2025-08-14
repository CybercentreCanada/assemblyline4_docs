# Performing Filtered File Collection

## Scenario

> “I want to collect all files with a very high score in Assemblyline (score ≥ 7000). I would like to also store these files on my antivirus-protected host so that I can feed them into another process.”

This exercise has the potential to iterate through a lot of data.

If we want to scan through all the contents in a Search API result without circling back on data we’ve already seen, we need to use a pointer.

You have to keep in mind the number of items the API can return per search request is bound to the number of documents Elasticsearch can return (10K) per request, so you won’t be able to get everything in a single request if there’s a lot of items to return.

This is where the idea of paging comes in and the Search API already gives you a `paging_id` that you can use in a later request to pick up from where you left off. Similar to what a bookmark does when reading a book.

Does this sound complex? It is, which is why we will be performing a “stream search” through the Assemblyline client rather than attempting to manage `paging_id` values with a native network request library:

```python
Client.search.stream.<index>()
```

## Expected Results

```text
A directory containing downloaded files that scored >= 7000

The downloaded files should have cART-encoding so that they do not trigger AV
```

## APIs Involved

=== "REST"
    ```text
    GET /api/v4/search/<index>/
    GET /api/v4/submission/full/<sid>/
    GET /api/v4/file/download/<sha256>/
    ```
=== "Python"
    ```python
    Client.search.stream.<index>()
    Client.submission.full(<sid>)
    Client.file.download(<sha256>)
    ```

## Solutions

=== "Python: Using `assemblyline_client`"
    ```python
    import os
    from assemblyline_client import get_client

    AL_HOST = os.getenv('AL_HOST', '<AL URL>')
    AL_USER = os.getenv('AL_USER', '<AL user>')
    AL_APIKEY = os.getenv('AL_APIKEY', '<AL API key>')

    # Filter for files within a submission that exceed a certain score
    FILE_SCORE_THRESHOLD = 7000

    # Download files that meet the FILE_SCORE_THRESHOLD into this directory
    OUTPUT_DIRECTORY = '/tmp/ex2'
    os.makedirs(OUTPUT_DIRECTORY, exist_ok=True)

    # This is the connection to the Assemblyline client that we will use
    client = get_client(f"https://{AL_HOST}:443", apikey=(AL_USER, AL_APIKEY), verify=False)

    # For all submissions that are over the file score threshold
    # client.search.stream.submission --> /api/v4/search/submission/?deep_paging_id=*
    for record in client.search.stream.submission(query=f"max_score:>={FILE_SCORE_THRESHOLD}", fl='sid'):
        sid = record['sid']

        # Download the full submission result and compute the score for each file
        # client.submission.full(<sid>) --> /api/v4/submission/full/<sid>/
        submission_results = client.submission.full(sid)

        # Compute the score of each files in the submission
        files_scores = dict()
        for result in submission_results['results'].values():
            # Initialize the default score for the file if the file is not in the list
            files_scores.setdefault(result['sha256'], 0)

            # Add the score of the result record to the file
            files_scores[result['sha256']] += result['result']['score']

        # For each files where the score is greater than threshold, download in cARTed format
        # client.file.download --> /api/v4/file/download/sha256?encoding=cart/
        for sha256, score in files_scores.items():
            if score >= FILE_SCORE_THRESHOLD:
                client.file.download(sha256, encoding="cart", output=os.path.join(OUTPUT_DIRECTORY, f"{sha256}.cart"))

    print(os.listdir(OUTPUT_DIRECTORY))
    ```
=== "Python: Using `requests`"
    ```python
    import requests
    import os

    headers = {
        "x-user": os.getenv('AL_USER', '<AL user>'),
        "x-apikey": os.getenv('AL_APIKEY', '<AL API key>'),
        "accept": "application/json"
    }

    # Filter for files within a submission that exceed a certain score
    FILE_SCORE_THRESHOLD = 7000

    # Download files that meet the FILE_SCORE_THRESHOLD into this directory
    OUTPUT_DIRECTORY = '/tmp/ex2'
    os.makedirs(OUTPUT_DIRECTORY, exist_ok=True)

    # This is the connection to the Assemblyline client that we will use
    host = f"https://{os.getenv('AL_HOST', '<AL URL>')}:443"

    # For all submissions that are over the file score threshold
    # client.search.stream.submission --> /api/v4/search/submission/
    # ** NOTE: this works for the purpose of this exercise but in real life you'd have to
    #          keep track of the deep_paging_id in a while loop.
    data = requests.get(f"{host}/api/v4/search/submission/?fl=sid&query=max_score:>={FILE_SCORE_THRESHOLD}",
                        headers=headers, verify=False).json()['api_response']['items']

    for submission in data:
        # Download the full submission result and compute the score for each file
        # client.submission.full(<sid>) --> /api/v4/submission/full/<sid>/
        submission_results = requests.get(f"{host}/api/v4/submission/full/{submission['sid']}/",
                                        headers=headers, verify=False).json()['api_response']

        # Compute the score of each files in the submission
        files_scores = dict()
        for result in submission_results['results'].values():
            # Initialize the default score for the file if the file is not in the list
            files_scores.setdefault(result['sha256'], 0)

            # Add the score of the result record to the file
            files_scores[result['sha256']] += result['result']['score']

        # For each files where the score is greater than threshold, download in cARTed format
        # client.file.download --> /api/v4/file/download/sha256?encoding=cart/
        for sha256, score in files_scores.items():
            if score >= FILE_SCORE_THRESHOLD:
                with open(os.path.join(OUTPUT_DIRECTORY, f"{sha256}.cart"), 'wb') as file:
                    raw_file = requests.get(f"{host}/api/v4/file/download/{sha256}/?encoding=cart",
                                            headers=headers, verify=False).content
                    file.write(raw_file)

    print(os.listdir(OUTPUT_DIRECTORY))
    ```

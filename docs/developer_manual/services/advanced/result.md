# *Result* class
All services in Assemblyline must create a `Result` object containing the different [ResultSections](../result_section) which encapsulates their findings. This `Result` object must then be set as the [Request's](../request) `result` parameter which will then in turn be saved in the database to show to the user.

You can view the source for the class here: [Result class source](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/result.py)

## Class variables
The `Result` class, even if critical for the service, does not have a whole lot of variables that should be used by the service.

| Variable Name | Description |
|:---|:---|
| sections | List of all the section objects that have been added to the `Result` object so far. |

## Class functions
There is only one function that the service writter should ever use in a `Result` object.

### add_section()
This function allows the service to add a [ResultSection](../result_section) object to the current `Result` object.

It can take the following parameters:

* `section`: The [ResultSection](../result_section) object to add to the list
* `on_top`: (Optional) Boolean value to tell if the section should be on top of the others or not

??? Example
    Excerpt from Assemblyline Result sample service: [result_sample.py](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/result.py)

    ```python
    ...
    kv_body = {
        "a_str": "Some string",
        "a_bool": False,
        "an_int": 102,
    }
    kv_section = ResultSection('Example of a KEY_VALUE section', body_format=BODY_FORMAT.KEY_VALUE,
                                body=json.dumps(kv_body))
    result.add_section(kv_section)
    ...
    ```


## Tips on writing good results
Here's a few tip on writing services:

!!! success "DO's"
    1. Make sure your results are easily readable for the user
    2. Include only necessary information
    3. Have less important section being collapse by default
    4. If your service has informations that can be tagged, don't forget to add those tags
    5. Choose a [ResultSection type](../result_section/#section-types) that will represent your data well

!!! fail "DON'T's"
    1. If your service has nothing to say, do not create add [ResultSection](../result_section) objects only containing that the service found nothing. There are a lot of shortcuts that the system takes with empty results and this will circumvent them.
    2. Don't overwhelm the users with information or they will stop reading it and just skim through
    3. Don't re-submit too many embedded files when you are not sure those files are worth it otherwise the results will be hard to read and will take a very long time to execute.

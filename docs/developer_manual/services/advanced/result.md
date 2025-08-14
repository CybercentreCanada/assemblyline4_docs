# *Result* class

All services in Assemblyline must create a `Result` object containing the different [ResultSections](../result_section) which encapsulate their findings. This `Result` object must then be set as the [Request's](../request) `result` parameter which will then be saved in the database to show to the user.

You can view the source for the class here: [Result class source](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/result.py)

## Class variables

Even though the `Result` class is critical for the service, it does not have many variables that should be used by the service.

| Variable Name | Description |
|:---|:---|
| sections | List of all the `ResultSection` objects that have been added to the `Result` object so far. |

## Class functions

There is only one function that the service writer should ever use in a `Result` object.

### add_section()

This function allows the service to add a [ResultSection](../result_section) object to the current `Result` object.

It can take the following parameters:

* `section`: The [ResultSection](../result_section) object to add to the list
* `on_top`: (Optional) Boolean value that indicates if the section should be on top of the other sections or not

??? Example
    Excerpt from Assemblyline ResultSample service: [result_sample.py](https://github.com/CybercentreCanada/assemblyline-v4-service/blob/master/assemblyline_v4_service/common/result.py)

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

Here's a few tips on writing services:

!!! success "DO's"
    1. Make sure your results are easily readable for the user
    2. Include only necessary information
    3. Collapse sections that are less important by default
    4. If your service has information that can be tagged, don't forget to add those tags
    5. Choose a [ResultSection type](../result_section/#section-types) that will represent your data well

!!! fail "DON'T's"
    1. If your service has nothing to say, do not create [ResultSection](../result_section) objects that contain that the service found nothing. There are a lot of shortcuts that the system takes with empty results and this will circumvent them.
    2. Don't overwhelm the users with information or they will stop reading it and just skim through
    3. Don't resubmit too many embedded files when you are not sure if those files are worth analyzing, otherwise the results will be hard to read and will take a very long time to execute.

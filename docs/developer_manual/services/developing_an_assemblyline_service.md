# Developing an Assemblyline service 

This guide has been created for developers who are looking to develop services for Assemblyline. It is aimed at individuals with general software development knowledge and basic Python skills. In-depth knowledge of the Assemblyline framework is not required to develop a service. 

## Pre-requisites
Before getting started, ensure you have the following setup:
- [Setting up dev environment](../run_your_service#setting-up-dev-environment)

## Build your first service
This section will guide you through the bare minimum steps required to create a running, but functionally useless service. Each sub-section below outlines the steps required for each of the different files required to create an Assemblyline service. All files created in the following sub-sections must be placed in a common directory, for example `result_sample`. 

### Service Python code
Create a Python file named `result_sample.py` with the following contents.
```python
import json
import os
import random
import tempfile

from assemblyline.common.dict_utils import flatten
from assemblyline.common.hexdump import hexdump
from assemblyline_v4_service.common.base import ServiceBase
from assemblyline_v4_service.common.result import Result, ResultSection, BODY_FORMAT

# DO NOT IMPORT IN YOUR SERVICE. These are just for creating randomized results.
from assemblyline.odm.randomizer import get_random_phrase, get_random_ip, get_random_host, get_random_tags
# DO NOT LIST BODY FORMATS LIKE THIS. This is again for the data randomizer.
FORMAT_LIST = [BODY_FORMAT.TEXT, BODY_FORMAT.MEMORY_DUMP]


class ResultSample(ServiceBase):
    def __init__(self, config=None):
        super(ResultSample, self).__init__(config)

    def start(self):
        # ==================================================================
        # On Startup actions:
        #   Your service might have to do some warming up on startup to make things faster

        self.log.info(f"start() from {self.service_attributes.name} service called")

    def execute(self, request):
        # ==================================================================
        # Execute a request:
        #   Every time your service receives a new file to scan, the execute function is called
        #   This is where you should execute your processing code.
        #   For the purpose of this example, we will only generate results ...

        # You should run your code here...

        # ==================================================================
        # Check if we're scanning an embedded file
        #   This service always drop two embedded file which one generates random results and the other empty results
        #   We're making a check to see if we're scanning the embedded file.
        #   In a normal service this is not something you would do at all but since we are using this
        #   service in our unit test to test all features of our report generator, we have to do this
        if request.sha256 not in ['d729ecfb2cf40bc4af8038dac609a57f57dbe6515d35357af973677d5e66417a',
                                  'cc1d2f838445db7aec431df9ee8a871f40e7aa5e064fc056633ef8c60fab7b06']:
            # Main file results...

            # ==================================================================
            # Write the results:
            #   First, create a result object where all the result sections will be saved to
            result = Result()

            # ==================================================================
            # Standard text section: BODY_FORMAT.TEXT - DEFAULT
            #   Text sections basically just dumps the text to the screen...
            #     All sections scores will be SUMed in the service result
            #     The Result classification will be the highest classification found in the sections
            text_section = ResultSection('Example of a default section')
            # You can add lines to your section one at a time
            #   Here we will generate a random line
            text_section.add_line(get_random_phrase())
            # Or your can add them from a list
            #   Here we will generate random amount of random lines
            text_section.add_lines([get_random_phrase() for _ in range(random.randint(1, 5))])
            # If the section needs to affect the score of the file you need to set a heuristics
            #   Here we will pick one at random
            text_section.set_heuristic(random.randint(1, 4))
            # Make sure you add your section to the result
            result.add_section(text_section)

            # ==================================================================
            # Color map Section: BODY_FORMAT.GRAPH_DATA
            #     Creates a color map bar using a minimum and maximum domain
            #     e.g. We are using this section to display the entropy distribution in some services
            cmap_min = 0
            cmap_max = 20
            color_map_data = {
                'type': 'colormap',
                'data': {
                    'domain': [cmap_min, cmap_max],
                    'values': [random.random() * cmap_max for _ in range(50)]
                }
            }
            section_color_map = ResultSection("Example of colormap result section", body_format=BODY_FORMAT.GRAPH_DATA,
                                              body=json.dumps(color_map_data))
            result.add_section(section_color_map)

            # ==================================================================
            # URL section: BODY_FORMAT.URL
            #   Generate a list of clickable urls using a json encoded format
            #     As you can see here, the body of the section can be set directly instead of line by line
            random_host = get_random_host()
            url_section = ResultSection('Example of a simple url section', body_format=BODY_FORMAT.URL,
                                        body=json.dumps({"name": "Random url!", "url": f"https://{random_host}/"}))

            # Since urls are very important features we can tag those features in the system so they are easy to find
            #   Tags are defined by a type and a value
            url_section.add_tag("network.static.domain", random_host)

            # You may also want to provide a list of url!
            #   Also, No need to provide a name, the url link will be displayed
            host1 = get_random_host()
            host2 = get_random_host()
            ip1 = get_random_ip()
            urls = [{"url": f"https://{host1}/"}, {"url": f"https://{host2}/"}, {"url": f"https://{ip1}/"}]
            url_sub_section = ResultSection('Example of a url section with multiple links', body_format=BODY_FORMAT.URL,
                                            body=json.dumps(urls))
            url_sub_section.set_heuristic(random.randint(1, 4))
            url_sub_section.add_tag("network.static.ip", ip1)
            url_sub_section.add_tag("network.static.domain", host1)
            url_sub_section.add_tag("network.dynamic.domain", host2)
            # Since url_sub_section is a sub-section of url_section
            # we will add it as a sub-section of url_section not to the main result itself
            url_section.add_subsection(url_sub_section)
            result.add_section(url_section)

            # ==================================================================
            # Memory dump section: BODY_FORMAT.MEMORY_DUMP
            #     Dump whatever string content you have into a <pre/> html tag so you can do your own formatting
            data = hexdump(b"This is some random text that we will format as an hexdump and you'll see "
                           b"that the hexdump formatting will be preserved by the memory dump section!")
            memdump_section = ResultSection('Example of a memory dump section', body_format=BODY_FORMAT.MEMORY_DUMP,
                                            body=data)
            memdump_section.set_heuristic(random.randint(1, 4))
            result.add_section(memdump_section)

            # ==================================================================
            # KEY_VALUE section:
            #     This section allows the service writer to list a bunch of key/value pairs to be displayed in the UI
            #     while also providing easy to parse data for auto mated tools.
            #     NB: You should definitely use this over a JSON body type since this one will be displayed correctly
            #         in the UI for the user
            #     The body argument must be a json dumps of a dictionary (only str, int, and booleans are allowed)
            kv_body = {
                "a_str": "Some string",
                "a_bool": False,
                "an_int": 102,
            }
            kv_section = ResultSection('Example of a KEY_VALUE section', body_format=BODY_FORMAT.KEY_VALUE,
                                       body=json.dumps(kv_body))
            result.add_section(kv_section)

            # ==================================================================
            # JSON section:
            #     Re-use the JSON editor we use for administration (https://github.com/josdejong/jsoneditor)
            #     to display a tree view of JSON results.
            #     NB: Use this sparingly! As a service developer you should do your best to include important
            #     results as their own result sections.
            #     The body argument must be a json dump of a Python dictionary
            json_body = {
                "a_str": "Some string",
                "a_list": ["a", "b", "c"],
                "a_bool": False,
                "an_int": 102,
                "a_dict": {
                    "list_of_dict": [
                        {"d1_key": "val", "d1_key2": "val2"},
                        {"d2_key": "val", "d2_key2": "val2"}
                    ],
                    "bool": True
                }
            }
            json_section = ResultSection('Example of a JSON section', body_format=BODY_FORMAT.JSON,
                                         body=json.dumps(json_body))
            result.add_section(json_section)

            # ==================================================================
            # Re-Submitting files to the system
            #     Adding extracted files will have them resubmitted to the system for analysis

            # This file will generate random results on the next run
            fd, temp_path = tempfile.mkstemp(dir=self.working_directory)
            with os.fdopen(fd, "wb") as myfile:
                myfile.write(data.encode())
            request.add_extracted(temp_path, "file.txt", "Extracted by some magic!")

            # This file will generate empty results on the next run
            fd, temp_path = tempfile.mkstemp(dir=self.working_directory)
            with os.fdopen(fd, "wb") as myfile:
                myfile.write(b"EMPTY")
            request.add_extracted(temp_path, "empty.txt", "Extracted empty resulting file")

            # ==================================================================
            # Supplementary files
            #     Adding supplementary files will save them on the datastore for future
            #      reference but wont reprocess those files.
            fd, temp_path = tempfile.mkstemp(dir=self.working_directory)
            with os.fdopen(fd, "w") as myfile:
                myfile.write(json.dumps(urls))
            request.add_supplementary(temp_path, "urls.json", "These are urls as a JSON file")
            # like embedded files, you can add more then one supplementary files
            fd, temp_path = tempfile.mkstemp(dir=self.working_directory)
            with os.fdopen(fd, "w") as myfile:
                myfile.write(json.dumps(json_body))
            request.add_supplementary(temp_path, "json_body.json", "This is the json_body as a JSON file")

            # ==================================================================
            # Wrap-up:
            #     Save your result object back into the request
            request.result = result

        # ==================================================================
        # Empty results file
        elif request.sha256 == 'cc1d2f838445db7aec431df9ee8a871f40e7aa5e064fc056633ef8c60fab7b06':
            # Creating and empty result object
            request.result = Result()

        # ==================================================================
        # Randomized results file
        else:
            # For the randomized  results file, we will completely randomize the results
            #   The content of those results do not matter since we've already showed you
            #   all the different result sections, tagging, heuristics and file upload functions
            embedded_result = Result()

            # random number of sections
            for _ in range(1, 3):
                embedded_result.add_section(self._create_random_section())

            request.result = embedded_result

    def _create_random_section(self):
        # choose a random body format
        body_format = random.choice(FORMAT_LIST)

        # create a section with a random title
        section = ResultSection(get_random_phrase(3, 7), body_format=body_format)

        # choose random amount of lines in the body
        for _ in range(1, 5):
            # generate random line
            section.add_line(get_random_phrase(5, 10))

        # choose random amount of tags
        tags = flatten(get_random_tags())
        for key, val in tags.items():
            for v in val:
                section.add_tag(key, v)

        # set a heuristic a third of the time
        if random.choice([False, False, True]):
            section.set_heuristic(random.randint(1, 4))

        # Create random sub-sections
        if random.choice([False, False, True]):
            section.add_subsection(self._create_random_section())

        return section

```

#### *ServiceBase* class
The main class of your Assemblyline service must inherit from the `ServiceBase` class which can be imported from `assemblyline_v4_service.common.base`. The `ServiceBase` base class includes many instance variables which can be used to access service related information. The following tables describes all of the available instance variables.

| Variable Name | Description |
|:---|:---|
| config | Reference to the service parameters containing values updated by the user for service configuration. |
| log | Reference to the logger. |
| service_attributes | Service attributes from the [service manifest](service_manifest.md). |
| working_directory | Returns the directory path which the service can use to temporarily store files during each task execution. |

#### *start* function
The `start` function is called when the Assemblyline service is initiated and should be used to prepare your service for task execution. This function is optional to implement.

#### *get_tool_version* function
The purpose of the `get_tool_version` function is to the return a string of all the tools used by the service. The tool version should be updated to reflect changes in the service tools, so that Assemblyline can rescan files on the new service version if they are submitted again.

#### *execute* function
The `execute` function is called every time the service receives a new file to scan. This is where you should execute your processing code. In the example above, we only generate static results for illustration purposes.

The `request` object is provided as an input to the `execute` function. The `request` object holds information about the task to be processed by the service. The following table describes all the `request` object variables which the service can use. 

| Variable name | Description |
|:---|:---|
| deep_scan | Returns whether the file should be deep scanned or not. Deep scanning usually takes more time and is better suited for file sent manually. |
| file_contents | Returns the raw bytes content of the file to be scanned. |
| file_name | Returns the name of the file (as submitted by the user) to be scanned. |
| file_path | Returns the path to the file to be scanned. The service can use this path directly to access the file. |
| file_type | Returns the Assemblyline style file type of the file to be scanned. |
| max_extracted | Returns the max number of files that are allowed to be extracted by a service. By default this is set to 500. |
| md5 | Returns the MD5 of the file to be scanned. |
| result | Used to set and get the current result. |
| sha1 | Returns the SHA1 of the file to be scanned. |
| sha256 | Returns the SHA256 of the file to be scanned. |
| temp_submission_data | Can be used to get and set temporary submission data which is passed onto subsequent tasks resulting from adding extracted files. |

The following table describes all the `request` object functions which the service can use.

| Function name | Parameters | Description |
|:---|:---|:---|
| add_extracted | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a file extracted by the service to the result. The extracted file will also be scanned through a set of services, as if it had been originally submitted. |
| add_supplementary | `path`: Complete path to the file <br><br> `name`: Display name of the file <br><br> `description`: Descriptive text about the file <br><br> `classification`: Optional classification of the file | Attach a supplementary file generated by the service to the result. The supplementary file is uploaded for the users informational use only and is not scanned. |
| drop | none | When called, the task will be dropped and will not be further processed by other remaining service(s). |
| get_param | `name`: name of the submission parameter to retrieve | Retrieve a submission parameter for the task. |
| set_service_context | `context`: Service context as string | Set the context of the service which ran the file. For example, if the service ran an AntiVirus engine on the file, then the AntiVirus definition version would be the service context. |

#### *stop* function
The `stop` function is called when the Assemblyline service is stopped and should be used to cleanup your service. This function is optional to implement.

### Service manifest YAML
Create a YAML file named `service_manifest.yml` with the following contents.
```yaml
# Name of the service
name: ResultSample
# Version of the service
version: 1
# Description of the service
description: >
  ALv4 Result example service

  This service provides examples of how to:
     - define your service manifest
     - use the different section types
     - use tags
     - use heuristics to score sections
     - use the att&ck matrix
     - use the updater framework
     - define submission parameters
     - define service configuration parameters

# Regex defining the types of files the service accepts and rejects
accepts: .*
rejects: empty|metadata/.*

# At which stage the service should run (one of: FILTER, EXTRACT, CORE, SECONDARY, POST)
# NOTE: Stages are executed in the order defined in the list
stage: CORE
# Which category the service is part of (one of: Antivirus, Dynamic Analysis, External, Extraction, Filtering, Networking, Static Analysis)
category: Static Analysis

# Does the service require access to the file to perform its task
# If set to false, the service will only have access to the file metadata (e.g. Hashes, size, type, ...)
file_required: true
# Maximum execution time the service has before it's considered to be timed out
timeout: 10
# Does the service force the caching of results to be disabled
# (only use for service that will always provided different results each run)
disable_cache: false

# is the service enabled by default
enabled: true
# does the service make APIs call to other product not part of the assemblyline infrastructure (e.g. VirusTotal, ...)
is_external: false
# Number of concurrent services allowed to run at the same time
licence_count: 0

# service configuration block (dictionary of config variables)
# NOTE: The key names can be anything and the value can be of any types
config:
  str_config: value1
  int_config: 1
  list_config: [1, 2, 3, 4]
  bool_config: false

# submission params block: a list of submission param object that define parameters
#                          that the user can change about the service for each of its scans
# supported types: bool, int, str, list
submission_params:
  - default: ""
    name: password
    type: str
    value: ""
  - default: false
    name: extra_work
    type: bool
    value: false

# Service heuristic blocks: List of heuristics object that define the different heuristics used in the service
heuristics:
  - description: This suspicious heuristic fakes making as a PDF
    filetype: "*"
    heur_id: 1
    name: Masks has PDF
    score: 100
  - description: This malicious heuristic fakes dropping and side-loading a DLL and has an Att&ck ID associated with it
    filetype: "*"
    heur_id: 2
    name: Drops an exe
    score: 1000
    attack_id: T1073
  - description: This informational heuristic fakes extracting a configuration block
    filetype: "*"
    heur_id: 3
    name: Extraction config information
    score: 10
  - description: This suspicious heuristic fakes decoding a configuration block and has an Att&ck ID associated with it
    filetype: "*"
    heur_id: 4
    name: Config decoding
    score: 100
    attack_id: T1027

# Docker configuration block which defines:
#  - the name of the docker container that will be created
#  - cpu and ram allocation by the container
docker_config:
  image: testing/assemblyline-service-resultsample:latest
  cpu_cores: 1.0
  ram_mb: 1024

# Update configuration block
update_config:
  # update method either run or build
  # run = run the provided update container
  # build = build the provided Dockerfile
  method: run
  # list of source object from where to fetch files for update and what will be the name of those files on disk
  sources:
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_2MB.zip
      name: sample_2mb_file
    - uri: https://file-examples.com/wp-content/uploads/2017/02/zip_5MB.zip
      name: sample_5mb_file
  # intervals in seconds at which the updater runs
  update_interval_seconds: 300
  # Should the downloaded files be used to create signatures in the system
  generates_signatures: false
  # options provided to the build or run command
  run_options:
    image: cccs/assemblyline-core:latest
    command: ["python3", "-m", "assemblyline_core.updater.url_update"]
```
A descriptions of all the valid fields in a `service_manifest.yml` are available [here](service_manifest.md).

### Dockerfile
Create a Dockerfile file named `Dockerfile` with the following contents:
```dockerfile
FROM cccs/assemblyline-v4-service-base:latest

ENV SERVICE_PATH result_sample.ResultSample

# Install any service dependencies here
# For example: RUN apt-get update && apt-get install -y libyaml-dev
#              RUN pip install utils

# Switch to assemblyline user
USER assemblyline

# Copy ResultSample service code
WORKDIR /opt/al_service
COPY . .
```

The Dockerfile is required to build a Docker container. When developing a Docker container for an Assemblyline service,
the following must be ensured:
- The parent image must be `cccs/assemblyline-v4-service-base:latest`.
- An environment variable named `SERVICE_PATH` must be set whose value defines the Python module path to the main service
class which inherits from the `ServiceBase` class.
- Any dependency installation must be completed as the `root` user, which is set by default in the parent image. Once all 
dependency installations have been completed, you must should change the user to `assemblyline`.
- The service code and any dependency files must be copied to the `/opt/al_service` directory.

## Conclusion
After you've completed creating your first service, your `result_sample` directory should have the following files at minimum:

```text
.
├── Dockerfile
├── result_sample.py
└── service_manifest.yml
```
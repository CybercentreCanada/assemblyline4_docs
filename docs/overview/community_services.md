# Community services

The Assemblyline community has been hard at work to improve Assemblyline's ability to detect malicious files and extract information about them.

This page lists all the services that our members have created and shared with the public.

!!! warning
These services are not managed by the Assemblyline team so make sure that you check their source thoroughly and that you are comfortable with what they do before you install them on your system.

## Service list

| Service Name                                                                                                             | Description                                                                                                                                                                                                                         | Author                                            |
| ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| [AutoItRipper](https://github.com/NVISOsecurity/assemblyline-service-autoit-ripper)                                      | AutoIt unpacker service                                                                                                                                                                                                             | [NVISO](https://github.com/NVISOsecurity)         |
| [ClamAV](https://github.com/NVISOsecurity/assemblyline-service-clamav)                                                   | Assemblyline service which submits a file to ClamAV and displays the result                                                                                                                                                         | [NVISO](https://github.com/NVISOsecurity)         |
| [HybridAnalysis](https://github.com/boredchilada/AL4-HybridAnalysis)                                                     | Uses the [Hybrid Analysis](https://www.hybrid-analysis.com/) service to provide additional threat intelligence and malware analysis capabilities                                                                                    | [boredchilada](https://github.com/boredchilada)   |
| [MalwareBazaar](https://github.com/NVISOsecurity/assemblyline-service-malware-bazaar)                                    | Assemblyline service fetching Malware Bazaar report                                                                                                                                                                                 | [NVISO](https://github.com/NVISOsecurity)         |
| [MsgParser](https://github.com/NVISOsecurity/assemblyline-service-msg-extractor)                                         | Simple MSG extractor AssemblyLine service                                                                                                                                                                                           | [NVISO](https://github.com/NVISOsecurity)         |
| [MetaDefender Sandbox](https://github.com/OPSWAT/assemblyline-service-metadefender-sandbox)                              | Submits a file or a URL to MetaDefender Sandbox                                                                                                                                                                                     | [OPSWAT](https://github.com/OPSWAT/)              |
| [PythonExeUnpack](https://github.com/NVISOsecurity/assemblyline-service-python-exe-unpacker)                             | Python exe unpacker service                                                                                                                                                                                                         | [NVISO](https://github.com/NVISOsecurity)         |
| [ReversingLabsSpectraIntelligence](https://github.com/reversinglabs/rl_assemblyline/tree/main/alsvc_spectraintelligence) | This service uses [ReversingLabs' Spectra Intelligence](https://www.reversinglabs.com/products/spectra-intelligence) service for obtaining detailed and high precision file reputation and analysis information on submitted files. | [ReversingLabs](https://github.com/reversinglabs) |
| [StegFinder](https://github.com/NVISOsecurity/assemblyline-service-steg-finder)                                          | AssemblyLine service which scans for embedded data in image using StegExpose                                                                                                                                                        | [NVISO](https://github.com/NVISOsecurity)         |
| [Unfurl](https://github.com/NVISOsecurity/assemblyline-service-unfurl)                                                   | Assemblyline service parsing a submitted URL to unshorten it.                                                                                                                                                                       | [NVISO](https://github.com/NVISOsecurity)         |
| [UrlScanIo](https://github.com/NVISOsecurity/assemblyline-service-urlscanio)                                             | This service fetches the results of a [urlscan.io](https://urlscan.io/) scan.                                                                                                                                                       | [NVISO](https://github.com/NVISOsecurity)         |
| WindowsDefender                                                                                                          | Windows defender service being adapted from an Assemblyline community [conversation](https://groups.google.com/g/cse-cst-assemblyline/c/LyziWuD8a9I/m/cg_m5eXpAQAJ)                                                                 | [Adam McHugh](https://github.com/adammchugh)      |

## Building a Community Service

1. Obtain the service source code
2. Edit the service manifest and ensure the following is set

```yaml
version: $SERVICE_TAG
---
docker_config:
  image: ${REGISTRY}<service_container_image>:$SERVICE_TAG
```

3. Build image and push to your local registry:

??? warning
It's strongly recommended to tag service images following the Assemblyline format. Otherwise, the system will disable your service because it will deem it incompatible with the rest of the components.

    Service versions should follow the format `A.B.C.(dev|stable).D`, where:

    - `A, B` represents the framework and system version, respectively.
    - `C, D` can be used to indicate the major and minor of a service, respectively.
    - The `dev` or `stable` portion of the tag should indicate the state of the service build. This is also relevant for providing service updates under a certain channel.

The following is an example of a service build targetted for an Assemblyline deployment running release 4.5.x.x:

```bash
docker build . -t <private_registry>/<service_container_image>:4.5.0.stable0 --build-arg version=4.5.0.stable0
docker push <private_registry>/<service_container_image>:4.5.0.stable0
```

4. Add contents of service manifest to UI or using the REST API to add the service to Assemblyline

## Add your service

Contact us on [Discord](https://discord.gg/GUAy9wErNu) to get your service featured on this page.

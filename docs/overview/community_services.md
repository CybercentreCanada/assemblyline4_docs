# Community services

The Assemblyline community has been hard at work to improve Assemblyline's ability to detect malicious files and extract information about them.

This page lists all the services that our members have created and shared with the public.

!!! warning
    These services are not managed by the Assemblyline team so make sure that you check their source thoroughly and that you are comfortable with what they do before you install them on your system.

## Service list

| Service Name | Description | Author | Source |
| -------------| ----------- | ------ | ------ |
| AutoItRipper | AutoIt unpacker service | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-autoit-ripper) |
| ClamAV | Assemblyline service which submits a file to ClamAV and displays the result | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-clamav) |
| MalwareBazaar | Assemblyline service fetching Malware Bazaar report | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-malware-bazaar) |
| MsgParser | Simple MSG extractor AssemblyLine service | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-msg-extractor) |
| OPSWAT Filescan Sandbox :material-new-box:{ .big_icon } | Submits a file or a URL to OPSWAT Filescan Sandbox | [OPSWAT](https://github.com/OPSWAT/) | [link](https://github.com/OPSWAT/assemblyline-service-opswat-filescan-sandbox) |
| PythonExeUnpack | Python exe unpacker service | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-python-exe-unpacker) |
| StegFinder | AssemblyLine service which scans for embedded data in image using StegExpose | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-steg-finder) |
| Unfurl | Assemblyline service parsing a submitted URL to unshorten it. | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-unfurl) |
| UrlScanIo | URLScan.io AL service | [NVISO](https://github.com/NVISOsecurity) | [link](https://github.com/NVISOsecurity/assemblyline-service-urlscanio) |
| WindowsDefender | Windows defender service being adapted from an Assemblyline community [conversation](https://groups.google.com/g/cse-cst-assemblyline/c/LyziWuD8a9I/m/cg_m5eXpAQAJ) | [Adam McHugh](https://github.com/adammchugh) | [link](https://github.com/adammchugh/Assemblyline-WindowsDefender-Service)

## Building a Community Service
1. Obtain the service source code
2. Edit the service manifest and ensure the following is set
```yaml
version: $SERVICE_TAG
...
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
The following is an example of a service build targetted for an Assemblyline deployment running release 4.4.x.x:
```bash
docker build . -t <private_registry>/<service_container_image>:4.4.0.stable0 --build-arg version=4.4.0.stable0
docker push <private_registry>/<service_container_image>:4.4.0.stable0
```
4. Add contents of service manifest to UI or using the REST API to add the service to Assemblyline

## Add your service

Contact us on [Discord](https://discord.gg/GUAy9wErNu) to get your service featured on this page.

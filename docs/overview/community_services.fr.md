# Services de la communauté

La communauté d'Assemblyline travaille fort pour améliorer cet outil à détecté des fichiers malicieux.

Cette page contient la liste de services crée et partagé avec le publique.

!!! warning "Attention"
    Ces services ne sont pas gèrer par l'équipe d'Assemblyline, nous vous invitons à faire une revue de leur code pour vous assurez d'être confortable avec ce qu'ils font avant de les utilisés dans vôtre système.

## Liste de services (anglais seulement)

| Nom de service | Description | Auteur | Source |
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

## Construire un service de la communauté
1. Obtenir le code source du service
2. Modifiez le manifeste du service et assurez-vous que les éléments suivants sont définis
```yaml
version: $SERVICE_TAG
...
docker_config:
    image: ${REGISTRY}<service_container_image>:$SERVICE_TAG
```
3. Créez une image et transférez-la vers votre registre local:

??? warning "Attention"
    Il est fortement recommandé de baliser les images de service en suivant le format Assemblyline. Sinon, le système désactivera votre service car il le jugera incompatible avec le reste des composants.

    Les versions de service doivent suivre le format « A.B.C.(dev|stable).D », où :

    - « A, B » représentent respectivement la version du framework et du système.
    - « C, D » peuvent être utilisés pour indiquer respectivement le majeur et le mineur d'un service.
    - La partie « dev » ou « stable » de la balise doit indiquer l'état de la construction du service. Ceci est également pertinent pour fournir des mises à jour de service sous un certain canal.

Voici un exemple de build de service destiné à un déploiement Assemblyline exécutant la version 4.4.x.x :
```bash
docker build . -t <private_registry>/<service_container_image>:4.4.0.stable0 --build-arg version=4.4.0.stable0
docker push <private_registry>/<service_container_image>:4.4.0.stable0
```
4. Ajoutez le contenu du manifeste de service à l'interface utilisateur ou utilisez l'API REST pour ajouter le service à Assemblyline.


## Faire nous part de vos services!

Contactez-nous sur [Discord](https://discord.gg/GUAy9wErNu)  pour faire ajouté vos services a cette page.

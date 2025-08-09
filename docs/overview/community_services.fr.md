# Services de la communauté

La communauté d'Assemblyline travaille fort pour améliorer cet outil à détecté des fichiers malicieux.

Cette page contient la liste de services crée et partagé avec le publique.

!!! warning "Attention"
Ces services ne sont pas gèrer par l'équipe d'Assemblyline, nous vous invitons à faire une revue de leur code pour vous assurez d'être confortable avec ce qu'ils font avant de les utilisés dans vôtre système.

## Liste de services (anglais seulement)

| Nom de service                                                                                                           | Description                                                                                                                                                                                                                                    | Auteur                                            |
| ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| [AutoItRipper](https://github.com/NVISOsecurity/assemblyline-service-autoit-ripper)                                      | Service de décompression AutoIt                                                                                                                                                                                                                | [NVISO](https://github.com/NVISOsecurity)         |
| [ClamAV](https://github.com/NVISOsecurity/assemblyline-service-clamav)                                                   | Service d'assemblage qui soumet un fichier à ClamAV et affiche le résultat                                                                                                                                                                     | [NVISO](https://github.com/NVISOsecurity)         |
| [HybridAnalysis](https://github.com/boredchilada/AL4-HybridAnalysis)                                                     | Utilise le service [Hybrid Analysis](https://www.hybrid-analysis.com/) pour fournir des renseignements supplémentaires sur les menaces et des capacités d'analyse des logiciels malveillants.                                                  | [boredchilada](https://github.com/boredchilada)   |
| [MalwareBazaar](https://github.com/NVISOsecurity/assemblyline-service-malware-bazaar)                                    | Service de ligne d'assemblage récupérant le rapport de Malware Bazaar                                                                                                                                                                          | [NVISO](https://github.com/NVISOsecurity)         |
| [MsgParser](https://github.com/NVISOsecurity/assemblyline-service-msg-extractor)                                         | Service AssemblyLine d'extraction simple de MSG                                                                                                                                                                                                | [NVISO](https://github.com/NVISOsecurity)         |
| [MetaDefender Sandbox](https://github.com/OPSWAT/assemblyline-service-metadefender-sandbox)                              | Soumet un fichier ou une URL à MetaDefender Sandbox                                                                                                                                                                                            | [OPSWAT](https://github.com/OPSWAT/)              |
| [PythonExeUnpack](https://github.com/NVISOsecurity/assemblyline-service-python-exe-unpacker)                             | Service de décompression de l'exe Python                                                                                                                                                                                                       | [NVISO](https://github.com/NVISOsecurity)         |
| [ReversingLabsSpectraIntelligence](https://github.com/reversinglabs/rl_assemblyline/tree/main/alsvc_spectraintelligence) | Ce service utilise le service [ReversingLabs' Spectra Intelligence](https://www.reversinglabs.com/products/spectra-intelligence) pour obtenir des informations détaillées et très précises sur la réputation et l'analyse des fichiers soumis. | [ReversingLabs](https://github.com/reversinglabs) |
| [StegFinder](https://github.com/NVISOsecurity/assemblyline-service-steg-finder)                                          | Service AssemblyLine qui recherche des données intégrées dans une image à l'aide de StegExpose.                                                                                                                                                | [NVISO](https://github.com/NVISOsecurity)         |
| [Unfurl](https://github.com/NVISOsecurity/assemblyline-service-unfurl)                                                   | Service Assemblyline qui analyse une URL soumise pour la raccourcir.                                                                                                                                                                           | [NVISO](https://github.com/NVISOsecurity)         |
| [UrlScanIo](https://github.com/NVISOsecurity/assemblyline-service-urlscanio)                                             | Ce service récupère les résultats d'une analyse [urlscan.io](https://urlscan.io/).                                                                                                                                                             | [NVISO](https://github.com/NVISOsecurity)         |
| WindowsDefender                                                                                                          | Service Windows defender adapté d'une communauté Assemblyline [conversation](https://groups.google.com/g/cse-cst-assemblyline/c/LyziWuD8a9I/m/cg_m5eXpAQAJ)                                                                                    | [Adam McHugh](https://github.com/adammchugh)      |

## Construire un service de la communauté

1. Obtenir le code source du service
2. Modifiez le manifeste du service et assurez-vous que les éléments suivants sont définis

```yaml
version: $SERVICE_TAG
---
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

Voici un exemple de build de service destiné à un déploiement Assemblyline exécutant la version 4.5.x.x :

```bash
docker build . -t <private_registry>/<service_container_image>:4.5.0.stable0 --build-arg version=4.5.0.stable0
docker push <private_registry>/<service_container_image>:4.5.0.stable0
```

4. Ajoutez le contenu du manifeste de service à l'interface utilisateur ou utilisez l'API REST pour ajouter le service à Assemblyline.

## Faire nous part de vos services

Contactez-nous sur [Discord](https://discord.gg/GUAy9wErNu) pour faire ajouté vos services a cette page.

# Référence rapide

```Vous trouverez la description des services sous Aide > Description des services```

## Service par type de fichier

Assemblyline propose une détection avancée du type de fichiers et ne mise pas exclusivement sur l’extension des fichiers
{: .label }

| Types de fichiers | Service |
|:-----------|:--------|
| Format de compression <br>_(iso, gz, zip, 7z, etc)_ | Extract |
| Exécutables Windows <br> _(exe, dll, sys)_ | PEFile Cuckoo Characterize Floss FrankenStrings  MetaPeek Unpacker YARA |
| Documents Office <br> _(doc, xls, ppt, macros)_ | Oletools Characterize Cuckoo ViperMonkey YARA MetaPeek DeobfuScripter |
| Courriel <br> _(eml)_ | EmlParser | 
| PDF<br> _(pdf)_ | PeePDF PDFId Cuckoo |
| JAVA | APKaye Characterize Espresso MetaPeek Cuckoo |
| Capture du réseau<br> _(pcap)_ | Suricata |
| Adobe Flash/Shockwave<br> _(swf)_ | Swiffer |
| Antivirus | MetaDefender VirusTotal  |
| Désobscurcissement | FrankenStrings Floss |
| Signatures | YARA Surricata TagCheck |

## Description des services

| Nom du service | Description | Instructions |
|:-------------|:--------------------|:-------------|
| APKaye | Ce service analyse les APK Android. Les APK sont décompilés et inspectés. Il affiche les indicateurs et l’information des réseaux mentionnés dans le fichier manifeste des APK. | [lien](https://github.com/CybercentreCanada/assemblyline-service-apkaye) |
| Characterize | Partitionne le fichier, calcule l’entropie visuelle de chaque partition et extrait les métadonnées Exif | [lien](https://github.com/CybercentreCanada/assemblyline-service-characterize) |
| Cuckoo | Permet d’utiliser l’analyse dynamique de maliciels dans un environnement en bac à sable | [lien](https://github.com/CybercentreCanada/assemblyline-service-cuckoo) |
| DeobfuScripter | Script statique de désobscurcissement. L’objectif n’est pas d’effectuer un désobscurcissement chirurgical, mais d’extraire les indicateurs de compromission obscurcis. | [lien](https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter)|
|EmlParser | Analyse les courriels au moyen de la bibliothèque eml_parser de GOVCERT-LU et extrait l’information de l’en-tête, les pièces jointes et les URI | [lien](https://github.com/CybercentreCanada/assemblyline-service-emlparser)|
| Espresso | Ce service analyse les fichiers JAR de Java. Toutes les classes sont extraites, décompilées et analysées pour détecter les comportements malveillants | [lien](https://github.com/CybercentreCanada/assemblyline-service-espresso)|
| Extract | Ce service extrait les fichiers incorporés dans les conteneurs de fichiers (tels que les fichiers ZIP, RAR, 7z, etc.) | [lien](https://github.com/CybercentreCanada/assemblyline-service-extract)|
| Floss | Extrait automatiquement les chaînes obscurcies des maliciels au moyen du solveur de chaîne des laboratoires FireEye | [lien](https://github.com/CybercentreCanada/assemblyline-service-floss)|
| FrankenStrings | Ce service procède à l’extraction des fichiers et des indicateurs de compromission en faisant appel à la mise en correspondance des modèles, à des décodeurs simples et à des désobscurcisseurs de scripts | [lien](https://github.com/CybercentreCanada/assemblyline-service-frankenstrings)|
| IPArse | Ce service est un analyseur de fichiers IPA (iOS) | [lien](https://github.com/CybercentreCanada/assemblyline-service-iparse)|
| MetaDefender | Service associé à l’antivirus OPSWAT MetaDefender (par plusieurs moteurs) | [lien](https://github.com/CybercentreCanada/assemblyline-service-metadefender)|
| MetaPeek | Vérifie les métadonnées de la soumission afin de relever les indicateurs de comportements potentiellement malveillants (extensions de fichier en double, etc.) | [lien](https://github.com/CybercentreCanada/assemblyline-service-metapeek)|
| Oletools | Ce service extrait les métadonnées et l’information sur le réseau, et signale les anomalies dans les documents Microsoft OLE et XML au moyen de la bibliothèque Python py-oletools de Philippe Lagadec - http://www.decalage.info | [lien](https://github.com/CybercentreCanada/assemblyline-service-oletools)|
| PDFId | Ce service extrait les métadonnées des fichiers PDF au moyen des outils PDFId et PDFParse de Didier Stevens | [lien](https://github.com/CybercentreCanada/assemblyline-service-pdfid)|
| PEFile | Ce service extrait les attributs (importations, exportations, noms de sections, etc.) des fichiers PE de Windows au moyen de la bibliothèque Python pefile | [lien](https://github.com/CybercentreCanada/assemblyline-service-pefile)|
| PeePDF | Ce service utilise l’information de la bibliothèque Python PeePDF tirée des fichiers PDF, notamment les blocs JavaScript qu’il tentera de désobscurcir, le cas échéant, pour une analyse plus poussée |[lien](https://github.com/CybercentreCanada/assemblyline-service-peepdf) |
| Suricata | Analyse la capture du réseau (.pcap) soumise et extraite lors de l’analyse |[lien](https://github.com/CybercentreCanada/assemblyline-service-suricata) |
| Swiffer | Ce service extraits les métadonnées et effectue une analyse des anomalies des fichiers Adobe Shockwave (.swf). | [lien](https://github.com/CybercentreCanada/assemblyline-service-swiffer)|
| TagCheck | Signatures YARA dans les balises d’Assemblyline (on peut concevoir sa propre signature pour détecter des balises en particulier) |[lien](https://github.com/CybercentreCanada/assemblyline-service-yara) |
| TorrentSlicer | Extrait l’information des fichiers torrent. |[lien](https://github.com/CybercentreCanada/assemblyline-service-torrentslicer) |
| Unpacker | Ce service décompresse les exécutables UPX condensés pour une analyse plus poussée |  [lien](https://github.com/CybercentreCanada/assemblyline-service-unpacker)|
| ViperMonkey | ViperMonkey est un moteur d’émulation VBA conçu par http://www.decalage.info | [lien](https://github.com/CybercentreCanada/assemblyline-service-vipermonkey)|
|VirusTotalDynamic | Ce service transmet les fichiers et les URL à VirusTotal et renvoie les résultats |[lien](https://github.com/CybercentreCanada/assemblyline-service-virustotal-dynamic) |
|VirusTotalStatic | Ce service vérifie le condensé dans l’API VirusTotal et renvoie le résultat |[lien](https://github.com/CybercentreCanada/assemblyline-service-virustotal-static) |
| YARA | Signature de fichiers |[lien](https://github.com/CybercentreCanada/assemblyline-service-yara) |
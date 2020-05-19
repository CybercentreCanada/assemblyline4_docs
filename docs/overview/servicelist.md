---
layout: default
title: Service list
parent: Overview
nav_order: 2
---
{: .no_toc }


# Quick reference

```Service listing on a system can be found under Help > Service Listing```

## Service per filetype

Assemblyline has advanced filetype detection and doesn't rely on file extension
{: .label }

| Filetypes | Service |
|:-----------|:--------|
| Compression format <br>_(iso, gz, zip, 7z, etc)_ | Extract |
| Windows executables <br> _(exe, dll, sys)_ | PEFile Cuckoo Characterize Floss FrankenStrings  MetaPeek Unpacker YARA |
| Office documents <br> _(doc, xls, ppt, macros)_ | Oletools Characterize Cuckoo ViperMonkey YARA MetaPeek DeobfuScripter |
| Email <br> _(eml)_ | EmlParser | 
| PDF<br> _(pdf)_ | PeePDF PDFId Cuckoo |
| JAVA | APKaye Characterize Espresso MetaPeek Cuckoo |
| Network capture<br> _(pcap)_ | Suricata |
| Adobe Flash/Shockwave<br> _(swf)_ | Swiffer |
| Anti-virus | MetaDefender VirusTotal  |
| Deobfuscation | FrankenStrings Floss |
| Signatures | YARA Surricata TagCheck |

## Service listing

| Service Name | Description | Instructions |
|:-------------|:--------------------|:-------------|
| APKaye | This service analyzes Android APKs. APKs are decompiled and inspected. Network indicators and information found in the APK manifest file is displayed | [link](https://github.com/CybercentreCanada/assemblyline-service-apkaye) |
| Characterize | Partitions the file and calculates visual entropy for each partition, extract Exif metadata | [link](https://github.com/CybercentreCanada/assemblyline-service-characterize) |
| Cuckoo | Provides dynamic malware analysis through sandboxing. | [link](https://github.com/CybercentreCanada/assemblyline-service-cuckoo) |
| DeobfuScripter | Static script de-obfuscator. The purpose is not to get surgical de-obfuscation, but rather to extract obfuscated IOCs. | [link](https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter)|
|EmlParser | Parse emails using GOVCERT-LU eml_parser library while extracting header information, attachments, URIs | [link](https://github.com/CybercentreCanada/assemblyline-service-emlparser)|
| Espresso | This service analyzes Java JAR files. All classes are extracted, decompiled and analyzed for malicious behavior | [link](https://github.com/CybercentreCanada/assemblyline-service-espresso)|
| Extract | This service extracts embedded files from file containers (like ZIP, RAR, 7z, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-extract)|
| Floss | Automatically extract obfuscated strings from malware using FireEye labs obfuscated string solver | [link](https://github.com/CybercentreCanada/assemblyline-service-floss)|
| FrankenStrings | This service performs file and IOC extractions using pattern matching, simple encoding decoder and script deobfuscators | [link](https://github.com/CybercentreCanada/assemblyline-service-frankenstrings)|
| IPArse | This service is an IPA File (iOS) Analyzer | [link](https://github.com/CybercentreCanada/assemblyline-service-iparse)|
| MetaDefender | Service for OPSWAT MetaDefender anti-virus (multi-engine) | [link](https://github.com/CybercentreCanada/assemblyline-service-metadefender)|
| MetaPeek | Checks submission metadata for indicators of potential malicious behavior (double file extenstions, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-metapeek)|
| Oletools | This service extracts metadata, network information and reports anomalies in Microsoft OLE and XML documents using the Python library py-oletools by Philippe Lagadec - http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-oletools)|
| PDFId | This service extracts metadata from PDFs using Didier Stevens PDFId & PDFParse | [link](https://github.com/CybercentreCanada/assemblyline-service-pdfid)|
| PEFile | This service extracts attributes (imports, exports, section names, ...) from windows PE files using the Python library pefile | [link](https://github.com/CybercentreCanada/assemblyline-service-pefile)|
| PeePDF | This service uses the Python PeePDF library information from PDFs including JavaScript blocks which it will attempt to deobfuscate, if necessary, for further analysis |[link](https://github.com/CybercentreCanada/assemblyline-service-peepdf) |
| Suricata | Scan network capture (.pcap) submitted and extracted from analysis |[link](https://github.com/CybercentreCanada/assemblyline-service-suricata) |
| Swiffer | This service extracts metadata and performs anomaly detection on Adobe Shockwave (.swf) files. | [link](https://github.com/CybercentreCanada/assemblyline-service-swiffer)|
| TagCheck | YARA signatures on Assemblyline Tags (build your own signature to hit on specific tags) |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |
| TorrentSlicer | Extracts information from torrent files. |[link](https://github.com/CybercentreCanada/assemblyline-service-torrentslicer) |
| Unpacker | This service unpacks UPX packed executables for further analysis |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacker)|
| ViperMonkey | ViperMonkey is a VBA Emulation engine by http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-vipermonkey)|
|VirusTotalDynamic | This service submits the files/URLS to VirusTotal and returns the results |[link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-dynamic) |
|VirusTotalStatic | This service performs a hash check against the VirusTotal API and returns the result |[link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-static) |
| YARA | Signature for file |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |
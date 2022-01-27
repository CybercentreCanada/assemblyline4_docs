# Assemblyline services

Services currently installed on a system can be found under `Help > Service Listing`. 

This is the list of all the services that are bundled with Assemblyline and that are maintained by the Assemblyline team:

| Service Name      | Speciality | Description | Source |
| ------------------| -- | -------------------- | ------------- |
| APKaye            | Android APK | APKs are decompiled and inspected. Network indicators and information found in the APK manifest file are displayed | [link](https://github.com/CybercentreCanada/assemblyline-service-apkaye) |
| Anti-virus        | Anti-virus | Generic ICAP client to integrate with most Anti-virus enterprise scanners | [link](https://github.com/CybercentreCanada/assemblyline-service-antivirus) |
| Characterize      | Entropy analysis | Partitions the file and calculates visual entropy for each partition, extract Exif metadata | [link](https://github.com/CybercentreCanada/assemblyline-service-characterize) |
| ConfigExtractor   | IoC extraction | Extract malware configuration file, allowing to get list of C2, encryption material etc. | [link](https://github.com/CybercentreCanada/assemblyline-service-configextractor) |
| Cuckoo            | Sandbox | Provides dynamic malware analysis through sandboxing. | [link](https://github.com/CybercentreCanada/assemblyline-service-cuckoo) |
| DeobfuScripter    | Deobfuscation | Static script de-obfuscator. The purpose is not to get surgical de-obfuscation, but rather to extract obfuscated IOCs. | [link](https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter)|
| EmlParser         | Email |Parse emails using [GOVCERT-LU eml_parser library](https://github.com/GOVCERT-LU/eml_parser) while extracting header information, attachments, URIs | [link](https://github.com/CybercentreCanada/assemblyline-service-emlparser)|
| Espresso          | Java | All classes are extracted, decompiled, and analyzed for malicious behavior | [link](https://github.com/CybercentreCanada/assemblyline-service-espresso)|
| Extract           | Compressed file | This service extracts embedded files from file containers (like ZIP, RAR, 7z, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-extract)|
| Floss             | IoC extraction  | Automatically extract obfuscated strings from malware using [FireEye Labs Obfuscated String Solver](https://github.com/fireeye/flare-floss) | [link](https://github.com/CybercentreCanada/assemblyline-service-floss)|
| FrankenStrings    | IoC extraction | This service performs file and IOC extractions using pattern matching, simple encoding decoder and script de-obfuscators | [link](https://github.com/CybercentreCanada/assemblyline-service-frankenstrings)|
| IPArse            | Apple IOS | Analyze Apple apps | [link](https://github.com/CybercentreCanada/assemblyline-service-iparse)|
| JSJaws            | Javascript | Analyze malicious Javascript | [link](https://github.com/CybercentreCanada/assemblyline-service-jsjaws)|
| MetaDefender      | Anti-virus | Service for OPSWAT MetaDefender anti-virus (multi-engine) | [link](https://github.com/CybercentreCanada/assemblyline-service-metadefender)|
| MetaPeek          | Meta data analysis | Checks submission metadata for indicators of potential malicious behavior (double file extensions, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-metapeek)|
| Oletools          | Office documents | This service extracts metadata, network information and reports anomalies in Microsoft OLE and XML documents using the [Python library py-oletools](https://github.com/decalage2/oletools) by Philippe Lagadec - http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-oletools)|
| Overpower         | PowerShell | De-obfuscate PowerShell scripts |[link](https://github.com/CybercentreCanada/assemblyline-service-overpower) |
| PDFId             | PDF | This service extracts metadata from PDFs using Didier Stevens [PDFId](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdfid.py) & [PDFParse](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdf-parser.py) | [link](https://github.com/CybercentreCanada/assemblyline-service-pdfid)|
| PEFile            | Windows binaries | This service extracts attributes (imports, exports, section names, ...) from windows PE files using the [Python library pefile](https://github.com/erocarrera/pefile) | [link](https://github.com/CybercentreCanada/assemblyline-service-pefile)|
| PeePDF            | PDF | This service uses the [Python PeePDF library](https://github.com/jesparza/peepdf) information from PDFs including JavaScript blocks which it will attempt to de-obfuscate, if necessary, for further analysis |[link](https://github.com/CybercentreCanada/assemblyline-service-peepdf) |
| PixAxe            | Images | Extract text from images | [link](https://github.com/CybercentreCanada/assemblyline-service-pixaxe)|
| Safelist          | Safelisting | Allow for hash, IoC and signature safelisting, including support for downloading [NSRL](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjaptDzi-7yAhWEdN8KHefxC6oQFnoECAsQAw&url=https%3A%2F%2Fwww.nist.gov%2Fitl%2Fssd%2Fsoftware-quality-group%2Fnational-software-reference-library-nsrl&usg=AOvVaw3I05XBysnEGMtwgozhlm8g) |[link](https://github.com/CybercentreCanada/assemblyline-service-safelist) |
| Sigma             | Eventlog signatures | Scan event logs (e.g. from sandbox or a compromised host) using [Sigma](https://github.com/SigmaHQ/sigma)|[link](https://github.com/CybercentreCanada/assemblyline-service-sigma) |
| Suricata          | Network signatures | Scan network capture (.pcap) submitted and extracted from analysis via [Suricata](https://github.com/OISF/suricata)|[link](https://github.com/CybercentreCanada/assemblyline-service-suricata) |
| Swiffer           | Adobe Shockwave | This service extracts metadata and performs anomaly detection on Adobe Shockwave (.swf) files | [link](https://github.com/CybercentreCanada/assemblyline-service-swiffer)|
| TagCheck          | Tag signatures | YARA signatures on Assemblyline Tags (build your own signatures to hit on specific tags) |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |
| TorrentSlicer     | Torrent files | Extracts information from torrent files |[link](https://github.com/CybercentreCanada/assemblyline-service-torrentslicer) |
| Unpacker          | UPX Unpacker | This service unpacks UPX packed executables for further analysis |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacker)|
| Unpac.me          | Unpacker | Integrate with [unpac.me](https://www.unpac.me) |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacme)|
| ViperMonkey       | Office documents | [ViperMonkey](https://github.com/decalage2/ViperMonkey) is a VBA Emulation engine by http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-vipermonkey)|
| VirusTotalDynamic | Anti-virus | This service submits the files/URLs to VirusTotal and returns the results |[link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-dynamic) |
| VirusTotalStatic  | Anti-virus | This service performs a hash check against the VirusTotal API and returns the result |[link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-static) |
| XLMMacroDeobfuscator | Office documents | Analyze Excel 4.0 macros |[link](https://github.com/CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator) |
| YARA              | File signatures | Signature for file |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |

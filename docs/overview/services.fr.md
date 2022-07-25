# Services d'Assemblyline

Les Services installé sur un système peuvent être trouvé sous: `Help > Service Listing`.


Cette liste contient tous les services inclus et maintenue avec Assemblyline:

| Service Name      | Speciality | Description | Source |
| ------------------| -- | -------------------- | ------------- |
| APKaye            | Android APK | APKs are decompiled and inspected. Network indicators and information found in the APK manifest file are displayed | [link](https://github.com/CybercentreCanada/assemblyline-service-apkaye) |
| Anti-virus        | Anti-virus | Generic ICAP client to integrate with most Anti-virus enterprise scanners | [link](https://github.com/CybercentreCanada/assemblyline-service-antivirus) |
| Characterize      | Analyze d'entropy | Calcule l'entropy des fichiers et extrait les meta-donnée Exif. | [link](https://github.com/CybercentreCanada/assemblyline-service-characterize) |
| ConfigExtractor   | Extraction | Extrait la configuration de malware connu, pour trouvé des liste de C2s, cle d'encryption etc. | [link](https://github.com/CybercentreCanada/assemblyline-service-configextractor) |
| Cuckoo            | Sandbox | Intégration avec la platform d'analyze dynamic Cuckoo | [link](https://github.com/CybercentreCanada/assemblyline-service-cuckoo) |
| DeobfuScripter    | Deobfuscation | Déobfuscation de script statique. | [link](https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter)|
| EmlParser         | Email | Analyze de fichier email avec [GOVCERT-LU eml_parser library](https://github.com/GOVCERT-LU/eml_parser) comme les attachements, URL etc | [link](https://github.com/CybercentreCanada/assemblyline-service-emlparser)|
| Espresso          | Java | Extrait les classes Java, décompile et analyze pour comportement malicieux | [link](https://github.com/CybercentreCanada/assemblyline-service-espresso)|
| Extract           | Compressed file | Extrait la plus part des type de compression (like ZIP, RAR, 7z, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-extract)|
| Floss             | IoC extraction  | Extrait des chaîne de charracters obfusqué avec [FireEye Labs Obfuscated String Solver](https://github.com/fireeye/flare-floss) | [link](https://github.com/CybercentreCanada/assemblyline-service-floss)|
| FrankenStrings    | IoC extraction | This service performs file and IOC extractions using pattern matching, simple encoding decoder and script de-obfuscators | [link](https://github.com/CybercentreCanada/assemblyline-service-frankenstrings)|
| IPArse            | Apple IOS | Analyze de fichier Apple IOS | [link](https://github.com/CybercentreCanada/assemblyline-service-iparse)|
| JSJaws            | Javascript | Analyze de fichier Javascript | [link](https://github.com/CybercentreCanada/assemblyline-service-jsjaws)|
| MetaDefender      | Anti-virus | Integration avec MetaDefender (multi-engine anti-virus) | [link](https://github.com/CybercentreCanada/assemblyline-service-metadefender)|
| MetaPeek          | Meta data analysis | Détect les signe malicieux dans les meta-données et les noms de fichier (double extension etc) | [link](https://github.com/CybercentreCanada/assemblyline-service-metapeek)|
| Oletools          | Office documents | Ce service analyze les fichiers Office et extrait des indicateurs de compromis avec [Python library py-oletools](https://github.com/decalage2/oletools) by Philippe Lagadec - http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-oletools)|
| Overpower         | PowerShell | Déobfusque les fichier powershell |[link](https://github.com/CybercentreCanada/assemblyline-service-overpower) |
| PDFId             | PDF | Analyze de fichier PDF avec Didier Stevens [PDFId](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdfid.py) & [PDFParse](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdf-parser.py) | [link](https://github.com/CybercentreCanada/assemblyline-service-pdfid)|
| PEFile            | Windows binaries | Analyze de fichier Windows (imports, exports, section names, ...) avec [Python library pefile](https://github.com/erocarrera/pefile) | [link](https://github.com/CybercentreCanada/assemblyline-service-pefile)|
| PeePDF            | PDF | Ce service utilise [Python PeePDF library](https://github.com/jesparza/peepdf) pour extraire de l'information de fichier PDF |[link](https://github.com/CybercentreCanada/assemblyline-service-peepdf) |
| PixAxe            | Images | Extrait du text des images | [link](https://github.com/CybercentreCanada/assemblyline-service-pixaxe)|
| Safelist          | Safelisting | Permet de "safelister" des indicateur dans le système comme des domaines, hash etc [NSRL](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjaptDzi-7yAhWEdN8KHefxC6oQFnoECAsQAw&url=https%3A%2F%2Fwww.nist.gov%2Fitl%2Fssd%2Fsoftware-quality-group%2Fnational-software-reference-library-nsrl&usg=AOvVaw3I05XBysnEGMtwgozhlm8g) |[link](https://github.com/CybercentreCanada/assemblyline-service-safelist) |
| Sigma             | Eventlog signatures | Analyze de "Windows Event logs" avec les règles [Sigma](https://github.com/SigmaHQ/sigma)|[link](https://github.com/CybercentreCanada/assemblyline-service-sigma) |
| Suricata          | Network signatures | Analyze les captures réseaux (pcap) avec [Suricata](https://github.com/OISF/suricata)|[link](https://github.com/CybercentreCanada/assemblyline-service-suricata) |
| Swiffer           | Adobe Shockwave | Analyze de fichier Shockwave (.swf) | [link](https://github.com/CybercentreCanada/assemblyline-service-swiffer)|
| TagCheck          | Tag signatures | Signatures YARA sur les tags d'Assemblyline Tags |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |
| TorrentSlicer     | Torrent files | Analyze de fichier torrent |[link](https://github.com/CybercentreCanada/assemblyline-service-torrentslicer) |
| Unpacker          | UPX Unpacker | Extrait des executable a partir de fichier "packer" avec UPX |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacker)|
| Unpac.me          | Unpacker | Intégration avec [unpac.me](https://www.unpac.me) |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacme)|
| ViperMonkey       | Office documents | [ViperMonkey](https://github.com/decalage2/ViperMonkey) est un programme d'emulation pour VBA par http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-vipermonkey)|
| VirusTotal | Anti-virus | Ce service vérifie (et soumet éventuellement) les fichiers/URL à VirusTotal pour analyse. | [link](https://github.com/CybercentreCanada/assemblyline-service-virustotal) |
| XLMMacroDeobfuscator | Office documents | Analyse de Macro Excel 4.0 |[link](https://github.com/CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator) |
| YARA              | File signatures | Signatures de fichier |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |

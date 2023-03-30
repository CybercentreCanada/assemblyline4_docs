# Services d'Assemblyline

Les Services installé sur un système peuvent être trouvé sous: `Help > Service Listing`.

Cette liste contient tous les services inclus et maintenue avec Assemblyline:

| Service Name      | Speciality | Description | Source |
| ------------------| -- | -------------------- | ------------- |
| APIVector         | Windows binaries | Extrait les imports de fichiers PE files ou mémoire pour générer un vecteur de classification. | [link](https://github.com/CybercentreCanada/assemblyline-service-apivector) |
| APKaye            | Android APK | Les APK sont décompilés et inspectés. Les indicateurs de réseau et les informations trouvées dans le fichier manifeste APK sont affichés | [link](https://github.com/CybercentreCanada/assemblyline-service-apkaye) |
| Anti-virus        | Anti-virus | Client ICAP générique utilisant plusieurs solution d'Anti-virus commerciales | [link](https://github.com/CybercentreCanada/assemblyline-service-antivirus) |
| BatchDeobfuscator | Deobfuscation | Résolution de variables locales et globales d'un fichier batch | [link](https://github.com/CybercentreCanada/assemblyline-service-batchdeobfuscator) |
| CAPA              | Windows binaries | Intégration de l'outil public [CAPA](https://github.com/mandiant/capa) | [link](https://github.com/CybercentreCanada/assemblyline-service-capa) |
| Characterize      | Analyze d'entropy | Calcule l'entropy des fichiers et extrait les meta-donnée Exif. | [link](https://github.com/CybercentreCanada/assemblyline-service-characterize) |
| ConfigExtractor   | Extraction | Extrait la configuration de malware connu, pour trouvé des liste de C2s, cle d'encryption etc. | [link](https://github.com/CybercentreCanada/assemblyline-service-configextractor) |
| CAPE            | Sandbox | Fournit une analyse dynamique des logiciels malveillants via le sandboxing. | [link](https://github.com/CybercentreCanada/assemblyline-service-cape)|
| DeobfuScripter    | Deobfuscation | Déobfuscation de script statique. | [link](https://github.com/CybercentreCanada/assemblyline-service-deobfuscripter)|
| ELF               | Linux binaries | Analyze de fichier ELF (sections, segments, ...) avec [Python library LIEF](https://github.com/lief-project/LIEF) | [link](https://github.com/CybercentreCanada/assemblyline-service-elf) |
| ELFPARSER         | Linux binaries | Intégration de l'outil public [ELFParser](https://github.com/jacob-baines/elfparser) | [link](https://github.com/CybercentreCanada/assemblyline-service-elfparser) |
| EmlParser         | Email | Analyze de fichier email avec [GOVCERT-LU eml_parser library](https://github.com/GOVCERT-LU/eml_parser) comme les attachements, URL etc | [link](https://github.com/CybercentreCanada/assemblyline-service-emlparser)|
| Espresso          | Java | Extrait les classes Java, décompile et analyze pour comportement malicieux | [link](https://github.com/CybercentreCanada/assemblyline-service-espresso)|
| Extract           | Compressed file | Extrait la plus part des type de compression (like ZIP, RAR, 7z, ...) | [link](https://github.com/CybercentreCanada/assemblyline-service-extract)|
| Floss             | IoC extraction  | Extrait des chaîne de charracters obfusqué avec [FireEye Labs Obfuscated String Solver](https://github.com/fireeye/flare-floss) | [link](https://github.com/CybercentreCanada/assemblyline-service-floss)|
| FrankenStrings    | IoC extraction | Ce service effectue des extractions de fichiers et d'IOC à l'aide de la correspondance de modèles, d'un décodeur d'encodage simple et de désobfuscateurs de script | [link](https://github.com/CybercentreCanada/assemblyline-service-frankenstrings)|
| IntezerDynamic    | File genome identification | Interface entre Intezer Analyze API 2.0, soumet le fichier pour analyse si le hachage n'est pas présent dans la base de données Intezer | [link](https://github.com/CybercentreCanada/assemblyline-service-intezer-dynamic)|
| IntezerStatic     | File genome identification | Interface entre Intezer Analyze API 2.0, effectue des recherches de hachage du fichier soumis | [link](https://github.com/CybercentreCanada/assemblyline-service-intezer-static)|
| IPArse            | Apple IOS | Analyze de fichier Apple IOS | [link](https://github.com/CybercentreCanada/assemblyline-service-iparse)|
| JSJaws            | Javascript | Analyze de fichier Javascript | [link](https://github.com/CybercentreCanada/assemblyline-service-jsjaws)|
| MetaPeek          | Meta data analysis | Détect les signe malicieux dans les meta-données et les noms de fichier (double extension etc) | [link](https://github.com/CybercentreCanada/assemblyline-service-metapeek)|
| Oletools          | Office documents | Ce service analyze les fichiers Office et extrait des indicateurs de compromis avec [Python library py-oletools](https://github.com/decalage2/oletools) by Philippe Lagadec - http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-oletools)|
| OneNoteAnalyzer | OneNote Documents | Ce service extrait les pièces joints et les métadonnées des fichiers Microsoft Onenote avec l'outil C# [OneNoteAnalyzer](https://github.com/knight0x07/OneNoteAnalyzer) | [link](https://github.com/CybercentreCanada/assemblyline-service-onenoteanalyzer)|
| Overpower         | PowerShell | Déobfusque les fichier powershell |[link](https://github.com/CybercentreCanada/assemblyline-service-overpower) |
| PDFId             | PDF | Analyze de fichier PDF avec Didier Stevens [PDFId](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdfid.py) & [PDFParse](https://github.com/DidierStevens/DidierStevensSuite/blob/master/pdf-parser.py) | [link](https://github.com/CybercentreCanada/assemblyline-service-pdfid)|
| PE                | Windows binaries | Analyze de fichier Windows (imports, exports, sections, ...) avec [LIEF](https://github.com/lief-project/LIEF) | [link](https://github.com/CybercentreCanada/assemblyline-service-pe)|
| PeePDF            | PDF | Ce service utilise [Python PeePDF library](https://github.com/jesparza/peepdf) pour extraire de l'information de fichier PDF |[link](https://github.com/CybercentreCanada/assemblyline-service-peepdf) |
| PixAxe            | Images | Extrait du text des images | [link](https://github.com/CybercentreCanada/assemblyline-service-pixaxe)|
| Safelist          | Safelisting | Permet de "safelister" des indicateur dans le système comme des domaines, hash etc [NSRL](https://www.nist.gov/itl/ssd/software-quality-group/national-software-reference-library-nsrl) |[link](https://github.com/CybercentreCanada/assemblyline-service-safelist) |
| Sigma             | Eventlog signatures | Analyze de "Windows Event logs" avec les règles [Sigma](https://github.com/SigmaHQ/sigma)|[link](https://github.com/CybercentreCanada/assemblyline-service-sigma) |
| Suricata          | Network signatures | Analyze les captures réseaux (pcap) avec [Suricata](https://github.com/OISF/suricata)|[link](https://github.com/CybercentreCanada/assemblyline-service-suricata) |
| Swiffer           | Adobe Shockwave | Analyze de fichier Shockwave (.swf) | [link](https://github.com/CybercentreCanada/assemblyline-service-swiffer)|
| TagCheck          | Tag signatures | Signatures YARA sur les tags d'Assemblyline Tags |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |
| TorrentSlicer     | Torrent files | Analyze de fichier torrent |[link](https://github.com/CybercentreCanada/assemblyline-service-torrentslicer) |
| Unpacker          | UPX Unpacker | Extrait des executable a partir de fichier "packer" avec UPX |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacker)|
| Unpac.me          | Unpacker | Intégration avec [unpac.me](https://www.unpac.me) |  [link](https://github.com/CybercentreCanada/assemblyline-service-unpacme)|
| URLDownloader     | URL Fetching | Récupère les URL apparemment malveillantes |  [link](https://github.com/CybercentreCanada/assemblyline-service-urldownloader)|
| ViperMonkey       | Office documents | [ViperMonkey](https://github.com/decalage2/ViperMonkey) est un programme d'emulation pour VBA par http://www.decalage.info | [link](https://github.com/CybercentreCanada/assemblyline-service-vipermonkey)|
| VirusTotal | Anti-virus | Ce service vérifie (et soumet éventuellement) les fichiers/URL à VirusTotal pour analyse. | [link](https://github.com/CybercentreCanada/assemblyline-service-virustotal) |
| XLMMacroDeobfuscator | Office documents | Analyse de Macro Excel 4.0 |[link](https://github.com/CybercentreCanada/assemblyline-service-XLMMacroDeobfuscator) |
| YARA              | File signatures | Signatures de fichier |[link](https://github.com/CybercentreCanada/assemblyline-service-yara) |



End of life, no longer actively supported:

| Service Name      | Speciality | Description | Source |
| ------------------| -- | -------------------- | ------------- |
| Cuckoo            | Sandbox | Intégration avec la platform d'analyze dynamic Cuckoo | [link](https://github.com/CybercentreCanada/assemblyline-service-cuckoo) |
| Lastline          | Sandbox | Fournit une analyse dynamique des logiciels malveillants via le sandboxing. | [link](https://github.com/CybercentreCanada/assemblyline-service-lastline) |
| MetaDefender      | Anti-virus | Integration avec MetaDefender (multi-engine anti-virus) | [link](https://github.com/CybercentreCanada/assemblyline-service-metadefender)|
| PEFile            | Windows binaries | Analyze de fichier Windows (imports, exports, section names, ...) avec [Python library pefile](https://github.com/erocarrera/pefile) | [link](https://github.com/CybercentreCanada/assemblyline-service-pefile)|
| VirusTotalDynamic | Anti-virus | Vérifie et envoie activement les fichiers à VirusTotal pour analyse.  | [link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-dynamic) |
| VirusTotalStatic  | Anti-virus | Vérifie VirusTotal pour une analyse existante sur le fichier soumis. | [link](https://github.com/CybercentreCanada/assemblyline-service-virustotal-static) |

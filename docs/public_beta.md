---
layout: default
title: Public Beta
nav_order: 2
has_children: true
has_toc: false
---

# Public Beta
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This Beta is intended for testing purposes only. Assemblyline 4 is not ready for production yet.

## Before deploying the Public Beta

- Not all services from v3 are available during the Beta
- There are still outstanding stability issues being worked on
- Only appliance/dev deployment setup will be available for the Beta
- The source will be released once the code is final
- Database schema may slightly change still, therefore there will be no upgrade path between Beta, Release Candidates and Final releases.
- Only minimal documentation will be available during the Beta

## Notable changes to Assemblyline in version 4

### Code

- The code base has been completely ported to Python 3. 
- Python 2 support was removed but we still have a way to run Python 2 services if this is absolutely necessary

### Database

- Riak and SOLR were replaced by Elasticsearch 7.6.0 
- The database now uses a strict Object data model
- Underlying data has changed a bit
- Data archiving has been built into the system using Elastic ILM 

### Containerization

- Designed to be run in the cloud or on-premise
- All components / services are now running in their own containers
    - For added security, service containers do not have access to the database or the file system anymore
    - Since all services have their own container, there will not be service dependency conflicts anymore
- Docker-compose is used for dev and appliance deployment 
- Kubernetes is used for cluster deployment

### Services
    
- Service configuration is now stored in a YAML file instead of code
- Scoring is now tied to heuristics, which will allow us to adjust scoring via machine-learning in the future
- Added support for Mitre's Att&ck matrix 

### User Interface

- Added printable report view
- Now explicitly saying if the system thinks the file is malicious or not
- Added feedback loop so that the users of the system can agree or disagree on the scoring of a file
    - This will be used to build the machine-learning model to auto-adjust the score of each heuristic
- Simplified UI to remove hidden menus
- Removed "Adding/Editing Signature" in favor of having a signature source management
    - You can now automatically fetch signature from an external URL or Git repository
- Better support for phones and tablets

### APIs

- Added many new APIs related to search
    - Faceting
    - Histogram
    - Statistics
- Standardized search output

### Authentication

- Added support for webauthn protocol
- Added generic LDAP authentication 
- Added support for oAuth with the following providers pre-configured: Google, Microsoft and Auth0

## Install the Beta

Want to try out the Beta?

[Install Beta](./public_beta/install.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


---
layout: default
title: Public Beta
nav_order: 2
has_children: true
has_toc: false
---

# Public beta 1
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This Beta is intended for testing purposes only. Assemblyline 4 is not ready for production yet.

## Things to consider before deploying the Public Beta

- Not all services from v3 are available during the Beta
- There are still outstanding stability issues being worked on
- Only appliance/dev deployment setup will be available for the Beta
- The source will only be release once the code is final 
- Database schema may still slightly change therefor there will be no upgrade path between Beta, Release Candidates and Final release.
- Only minimal documentation will be available during the Beta

## Noteable changes to Assemblyline in version 4

### Code

- The code base has been completely ported to Python 3. 
- Python 2 support was removed but we still have a way to run python 2 services if this is absolutely necessary

### Database

- Riak and SOLR were replaced by Elasticsearch 7.5.1 
- The database now uses a stric Object data model
- Underlying data has changed a bit
- Data achiving has been built-in the system using Elastic ILM 

### Containerization

- Designed to be run in the cloud first but also on premise
- Every components / services are now running in their own container 
    - Service containers do not have access to the database or the file system anymore for added security
    - Since all services have their own container, there will not be service dependancy conflicts anymore
- Docker compose is used for dev and appliance deployment 
- Kubernetes is used for Cluster deployment

### Services
    
- Get their configuration from a YAML file instead of code
- Scoring is now tied to heuristics which will allow us to adjust scoring via Machine learning in the future
- Added support for Mitre's Att&ck matrix 

### User Interface

- Added printable report view
- Now explicitely saying if the system thinks the file is malicious or not
- Added feedback loop to the user's of the system can agree or disagree on the scoring of a file
    - This will be use to build the machine learning model to auto-adjust the score of each heuristics
- Simplify UI to remove hidden menus
- Remove Adding/Editing signature in favor of having a signature source management
    - You can now automatically fetch signature from an external URL or git repository
- Better support for phones and tablet 

### APIs

- Added a bunch of new APIs related to search
    - Facetting
    - Histogram 
    - Statistics
- Standardized search output
- Cleanup output of some APIs

## Install the Beta

Want to try out the Beta?

[Install Beta](./public_beta/install.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }


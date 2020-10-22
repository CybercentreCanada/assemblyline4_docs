---
layout: default
ref: main
title: Overview
nav_order: 1
description: "CCCS Assemblyline 4 Documentation"
permalink: /
has_children: true
has_toc: false
---

<script>
  function resizeIframe(obj) {
    obj.style.height = obj.contentWindow.document.body.scrollHeight + 'px';
  }
</script>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>


# Assemblyline 4
{: .fs-9 }

Welcome to the [Canadian Centre for Cybersecurity](https://www.cyber.gc.ca/en)'s Assemblyline project!
{: .fs-6 .fw-300 }

[Get started now](./docs/installation.html){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View it on GitHub](https://github.com/CybercentreCanada?q=assemblyline){: .btn .fs-5 .mb-4 .mb-md-0 }

---

Assemblyline 4 is an open source malware analysis platform. It is designed to assist cyber defence teams to automate the analysis of files and to better use the time of security analysts.
Build using cloud technologies, it can scale from small to large scale enterprise security operation scanning millions of files a day and provide triage capabilities.

Assemblyline can be easily integrated in your environment using its powerful restApi and web interfaces. The platform  comes with dozens of services to provide deep file analysis and enable integration with other security platforms such as anti-virus, malware detonation sandboxes and threat knowledge bases. Best of all, with a little bit of Python code you can extend it yourself by creating new analysis and integration services.

Learn [more](./docs/overview/how_it_works.html) !



<iframe class="slideshow-iframe" src="https://cybercentrecanada.github.io/assemblyline4_docs/slides/screenshots.html"
style="width:100%" frameborder="0" scrolling="no" onload="resizeIframe(this)">
</iframe>
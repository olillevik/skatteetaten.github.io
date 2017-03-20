---
title: Coding
layout: page
---

Coding an application targeted at OpenShift is mostly like following the well known [12factor](https://12factor.net/) principles from Heroku.

The following specific demands must also be followed:
 - Java is the only supported language
    - Node is supported in the [Backend for Frontend](http://samnewman.io/patterns/architectural/bff/) case, but only with very limited functionality
 - The delivery mechanism is a assembly bundle zip file uploaded to Nexus
    - Fat-jars are not supported since we security check all dependencies with Nexus CLM
    - The delivery bundle contains at minimum a lib folder with all jars and a metadata/openshift.json file to provide construction time metadata.
 - All applications must implement the management interface demands
    - must be served on a port of its own, not accessible to the internet
    - must expose prometheus metrics, healthchecks, env and other info (build metadata, git metadata, dependencies and other links)
    - must log using standard LOGBACK configuration
 - The application must handle SIGTERM gracefully
 - A Jenkinsfile, preferably using the shared Jenkinsfile script

We provide a [Tailored Service Template](https://www.thoughtworks.com/radar/techniques/tailored-service-template) for a standard application that fullfill all these demands.

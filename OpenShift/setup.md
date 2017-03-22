---
title: Setup
layout: post
category: OpenShift
---

Setting up the required OpenShift objects for deploying an application is done either via the CLI tool AOC or via the WebGUI Aurora Console. In order to do this some deploy time metadata is needed. 

The following features can be configured in the deploy time metadata:
 - location of the artifact in Nexus that we are deploying
 - what database schemas to generate/reuse
 - do you need a security token for secure communication?
 - config variables
 - deployment strategy, what version do you want to deploy
 - generate a route for this application
 - enable rolling upgrades
 - configure Splunk index
 - create other routes/automate opening traffic in network infrastructure (webseal/BiG-IP)
 - what version strategy you want to use

The setup process is idempotent so calling it several times will only update the required parts in the old objects.

The result of this process is illustrated in the diagram below. Here we are using major strategy and deploying all new releases under the 1 major tag.


![Deploy](deploy.png)
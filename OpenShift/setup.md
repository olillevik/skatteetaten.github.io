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

## Versioning strategies
Multiple tags are [pushed for a release](versioning.html). When specifying what version to deploy in the boober configuration you can choose several different strategies.

* use the full **AuroraVersion** of a release to pin the deployment to that release. It will never be automatically bumped.
* use **latest** to always get the latest semantic release deployed
* use **SNAPSHOT-_branchname_** to get the latest build from a branch
* use *1* to get all new releases in the 1 tree, all new features and bugfixes but no breaking changes
* use *1.1* to get all new bugfixes for 1.1 release but no new features
* use *1.1.1* to get no new code, but only updates if the infrastructure changes. 

For all strategies except the first there will be triggered a new deploy if there is [infrastructure changes](patching.html) and your strategy is running the latest built release.

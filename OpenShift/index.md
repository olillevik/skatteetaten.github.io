---
layout: home
---

# The Aurora OpenShift Platform

## Introduction

Welcome to the Aurora OpenShift Platform of [The Norwegian Tax Administration (TNTA)](http://skatteetaten.no/en/person/) on GitHub. The Norwegian Tax 
Administration is an agency under the authority of the Ministry of Finance and is responsible for taxation and the 
population register. 

These pages on GitHub is the home of the initiative to open source the platform that runs part of the TNTA's digital tax
and customs services; the Aurora OpenShift Platform.

This document will describe what the Aurora OpenShift Platform is and how it came to be. It will also cover how we 
are organized for developing software and our development process - including how we use the platform to build and 
deploy our applications, and our guidelines and requirements to applications using the platform.


## A Short History 

The work on what would become the Aurora OpenShift Platform started late in 2014 in the wake of a major project to 
modernize the handling of the foundational data for most tax calculations in Norway - MAG. The MAG project transitioned
the Norwegian Tax Administration from using Oracle Forms, PL/SQL and Java EE into using simpler Java SE technologies
with embedded containers. It also identified the need for a common platform for running and handling the vast 
amount of micro services across development, test and reference environments that the project produced, and that would 
be produced by even larger projects down the line. 

A technology evaluation phase revealed the Red Hat OpenShift Container Platform (then Red Hat OpenShift Origin 3.0) to
be the most appropriate technology to base a new common platform on. Most of 2015 was spent on getting the first few 
applications successfully running on OpenShift and device a suitable strategy for running, configuring and managing all 
the applications that would, in time, be running on the platform.

As time passed it became clear that Kubernetes based
OpenShift was an extremely capable platform for running our applications. We also realized that handling integration
with the other infrastructure components (like database servers, logging/metrics/monitoring tools, and security
infrastructure), and handling the sheer amount of application instances and application runtime environments with all
their associated application versions, configurations and delivery schedules required more than OpenShift alone could
provide.

In the course of getting the first few applications running on OpenShift we had already done quite a bit of
automation and integration work. As the tools and services that provided those features matured and got structured,
they became collectively known as the Aurora OpenShift Platform by the end of 2016.

In December 2016 it was decided that as much possible of the Aurora OpenShift Platform should be open sourced under
an APL 2.0 compatible license.


## Requirements for the Aurora OpenShift Platform

The Norwegian Tax Administration maintains millions of lines of code and employs X people in software development,
IT operations, management and supporting roles. When the need for new software arises - to either replace existing
systems or to handle new or changed responsibilities - a project is created for that purpose. The projects are often
heavily staffed with external consultants, and when complete they are transferred to the Line. The Line will
maintain and support the software over time, usually decades, and will initially be staffed with personnel from both the 
project and from the Line itself.

TODO: Maybe some infrastructure stuff in here?

Even before we started exploring OpenShift we knew that we would have to automate integration with our existing
infrastructure regardless of the platform we would end up using - so doing some sort of common initiative to provide
automation and integration services was given from the get-go. As we familiarized ourselves with OpenShift we saw
that there were several other areas that would also benefit from a central, common effort.

It was important to us that supporting applications on the platform developed by different teams, often with personnel 
unfamiliar with developing software for the NTA, should be pretty much the same regardless of who developed it. For
instance, rolling out a Java Virtual Machine patch should be exactly the same for all applications, and it should be
possible for us to roll out such a patch for all applications across all environments in one go. Similarly, the
process of upgrading an application developed by one team deployed to a specific environment should be exactly the
same for any other application developed by a completely different team. Additionally, we wanted configuration to be
handled the same for all applications across all environments.

One of the things that struck us after getting some experience with OpenShift was how flexible it was to work with.
Although the platform certainly comes with its fair share of requirements and idiosyncrasies, we found that there was 
hardly anything we were not able to do. However, we also very quickly realized that with all this flexibility comes a
quite steep learning curve on how "to do things right", and allowing individual teams to freely develop their
own strategies on how to use OpenShift would bear with it a high cost of entry, slower internal adoption and
difficulties streamlining maintenance and deployment activities across teams. Also, there were few obvious upsides to
letting teams freely use OpenShift their own way. For instance, we were hard press to find good answers to questions
like; 
 * What would be the benefit for teams to develop and maintain their own base image for their Java applications instead 
 of collaborating on one?
 * What would be the benefit for teams to device their own Docker Image build processes instead of adopting a common one?
 * What would be the benefit for teams to develop and maintain their own scripts and tools for handing deployment and 
 configuration across different environments?

Going back and forth, we almost always ended up at "not a whole lot". It became pretty clear that a common effort to
coordinate how the teams should use the platform and maintain reusable components and services was a good idea.


## What is the Aurora OpenShift Platform?

TODO: Needs work

The Aurora OpenShift Platform is everything The Norwegian Tax Administration has developed to support infrastructure
automation, support application configuration, deployment and management, common application base images
and common build and versioning mechanism for application archives and docker images.

The main user facing components implementing these mechanisms are:

 * Aurora Console: The most prominent feature of the Aurora OpenShift platform is the Aurora Console. The Aurora Console is a custom
built web application that we use in conjunction with the platform provided OpenShift Console to better handle
applications across teams and environments.
 * AOC: The Aurora OpenShift Command Line Client. Our custom built CLI that works with a set of high level configuration
 files that helps managing applications and configuration across environments.
 * The Aurora API: The API that implements all our custom services. Used by the Aurora Console and AOC.
 * Architect: A Docker image that implements the image build process for all our supported runtime technologies
 * Base Images: A set of Alpine Linux based Docker Images that all our applications are built from

Coming sections will describe these components in more detail.


## How we Develop and Build our Applications

### Coding and Application Build Requirements

Coding an application targeted at the Aurora OpenShift Platform follows closely the principles of the 
[Twelve-Factor App](https://12factor.net/) from Heroku. Additionally the following requirements must be met;

 * The application must run on the JVM and, currently, must be written exclusively in Java. Node is supported when 
 developing [Backends for Frontends](http://samnewman.io/patterns/architectural/bff/), but then only with very limited
 functionality.
 * The application must currently be built using Apache Maven.
 * The delivery mechanism is an assembly bundle zip file uploaded to our internal Nexus registry. We call this zip file
 a Leveransepakke (Delivery Bundle)
   * The Delivery Bundle must contain a lib folder with all the jars for the application.
   * The Delivery Bundle must contain a metadata/openshift.json file to provide build time metadata to the application
   image building process (Architect). This includes among other things information used to generate a start script for
   the application and metadata used to label the Docker image.
 * We support both versioned releases and snapshots, but versioned releases must follow the 
 [Semantic Versioning](http://semver.org/) system.
 * The application must implement our proprietary management interface. This interface is described in more detail
 later.
 
Additionally we prefer that the applications are built via Jenkins and that the source repository of the application
contains a Jenkinsfile that describes the build process. Ideally, the Jenkinsfile uses our common Jenkins pipeline
scripts (making the Jenkinsfile less than 10 lines of code).

We provide a [Tailored Service Template](https://www.thoughtworks.com/radar/techniques/tailored-service-template) for a
standard application that fulfill all these demands that teams building new applications can use to get started.


### The Image Build Process

Once the Delivery Bundle has been uploaded to Nexus we trigger (or create) an OpenShift BuildConfig for that specific
artifact (identified by the groupId and artifactId that was used when uploading to Nexus). We also provide the version
of the artifact that was just built as a parameter to the build.

The BuildConfig is configured to use the 
OpenShift Container Platform’s [Custom build strategy](https://docs.openshift.com/container-platform/latest/dev_guide/builds/build_strategies.html#custom-strategy-options)
and we have created our own Docker image, dubbed Architect, to build an application Docker image from the Delivery
Bundle based on our Java base image, dubbed Wingnut. 

In addition to building the application image, Architect is also responsible for tagging the produced image with 
several application and platform specific version tags that is the underpinning of our flexible deployment strategy. 

Architect, Wingnut, our versioning strategy and our deployment strategy are described in more detail below.

In addition to passing parameters to Architect, the BuildConfig is also configured with two 
[ImageChange triggers](https://docs.openshift.com/container-platform/latest/dev_guide/builds/index.html#image-change-triggers).
One for Architect and one for Wingnut. This allows us to use OpenShift to automatically trigger a build of the most
recent version of any application when we release a new version of either Architect or Wingnut.


### The Application Image Builder: Architect

Architect is a Docker image built upon Alpine Linux that is responsible for building all our application images. It is
designed to work as an OpenShift 
[CustomBuilder](https://docs.openshift.com/container-platform/3.4/creating_images/custom.html) and is mostly triggered
from BuildConfigs. It will download a prebuilt artifact from Nexus based on the groupId, artifactId and version (GAV)
provided as parameters, inspect the metadata/openshift.json-file to determine the technology used by the application.
Based on the technology used, a suitable base image will be selected and the build process determined.

For our Java applications Architect supports generating a start script based on a few parameters set in the 
metadata/openshift.json-file. Though not a requirement at this time (applications may provide their own start script),
providing a mechanism for automatically generating a start script has a few major benefits;
 * Getting a start script for Java right on OpenShift is actually quite hard. The script must make sure that the main 
 process is properly backgrounded, while still handling signals like SIGTERM. It must also trap the Java process to 
 rewrite application exit codes; for instance, Java exits with 143, while OpenShift expects properly terminated 
 applications to return 0. Additionally -Xmx must be set in relation to the available cgroup memory. Our generated start
 script handles all these issues.
 * We require that the Jolokia agent is enabled for all Java processes to enable the hawt.io integration in OpenShift.
 Our generated start script enables this by default.

Additionally, the generated start script will create a deterministic class path from the lib folder, set JVM_OPTS and
APPLICATION_ARGS from the openshift.json file and optionally enable remote debugging.

The start script we use is heavily inspired by several other similar solutions, chief among them 
[run-java-sh](https://github.com/fabric8io-images/run-java-sh).

Provide a run script that loads configuration.

TODO: These are just things we need to remember to write about:
 * Based on the technology used in the Delivery Bundle (Java or Node) an appropriate base image is selected
 * The application along with a few commonly used resources (logging config, etc) is added to the image
 * The image is pushed to a central Docker Registry
 * The image is tagged with the version of the builder image, the version of the base image and the application version,
 collectively called the Aurora Version. Several other tags are also pushed (major, minor, patch).
 * When an image is pushed, OpenShift ImageStreams may trigger application redeploys from ImageChange triggers.
 * BuildConfigs may be triggered to rebuild images for specific application versions from ImageChange triggers from both
 the Builder Image (Architect) and the base images. Rebuilding application images may in turn result in automatic 
 redeploys from ImageChange triggers. Commonly, when releasing for instance a new base image for Java with a new Java
 Runtime Environment version, hundreds of application images are automatically rebuilt - and in some cases automatically
 redeployed by the platform.


The flow of the build logic is as follows:
  - validate input
  - find actual version of base image for AuroraVersion
  - fetch delivery bundle from Nexus
  - generate a start script 
  - create a Dockerfile based upon one of our base docker image
  - build docker image
  - tag and push relevant tags

More details will be revealed when it is open sourced.


### The Java Base Image: Wingnut

TODO: Add something meaningful here


### Image Versioning Strategy

When creating docker images with a dedicated builderImage and different base image we feel it is necessary to version 
these in a way that makes it clear how to recreate the image at a later stage. The version scheme that is used can be seen 
in the following diagram

![AuroraVersion](auroraVersion.png)

The full version number contains information about the application code, the version of the build logic, the name of the base image and the version of the base image.
It is now possible to know exactly the parts needed to rebuild it, or where to look for bugs.

When tagging a built DockerImage the rules are different based upon the kind of release
## Semantic release
![Versioning](versioning.png)


## Snapshot release
When building a snapshot release you often want to just follow the latest snapshot built. In some cases you might want 
to pin it to an exact build number

The following tags are created:
 - SNAPSHOT-**APPNAME**-**buildnumber**, build number is fetched from Nexus
 -  **APPNAME**-SNAPSHOT




### The architecture behind Aurora OpenShift

 * How to [code](coding.html) an application
 * How the application is [built](building.html) on Jenkins
 * How to [configure and setup](setup.html) the infrastructure on OpenShift to deploy your application
 * How to [monitor](monitoring.html) your application to make sure it performs as expected
 
 
## Suggested todo list

 * Develop a proper outline (table of contents) in the index file. With the current structure some of the pages are only accessible via other sub pages - which can make them hard to find and get a complete overview of the documentation.
 
 * Suggestion: A two section document;
   - Section One: What is the Aurora OpenShift platform, what features are in it, why are they there and how do they work?
   - Section Two: Implementation details; Boober, the Database Hotel API, SKAP, Marjory, Wembley etc.
 
 * A significantly more meaty introduction that includes more background information (not necessarily in the same order);
   - What is the responsibility of The Norwegian Tax Authority?
   - A short history of the Aurora OpenShift platform (even before it was called the Aurora OpenShift platform)
   - What are the problems the Aurora OpenShift platform set out to solve? Is it solving them right now? Why did we decide to build our own set of tools and abstractions on top of OpenShift and other relevant infrastructure components. Common Components, Business Applications, Application "clusters" - Environments (dozens of test and development envronments)
   - What kinds of applications are/will be running on the Aurora OpenShift platform?
   - An overview of the technology stack in use by the applications developed by the Norwegian Tax Authority.
 * An overview of the main features of the Aurora OpenShift platform (What is the Aurora OpenShift platform).
 * An overview of the main user facing Aurora OpenShift abstractions;
   - Leveransepakke (Delivery Bundle)
   - Base Image
   - Architect
   - Aurora Console
   - AOC and the AOC configuration files
   - The Aurora API (?) and its main resources. Not an established term, but I think it will serve us better in the long run to view the common collection of endpoints provided by "our" platform as a single uniform API regardless of which underlying component, be it Boober, Sprocket, the Aurora Console or any other component, implements a specific endpoint/resource.
 * The anatomy of an Aurora Application
   - Application requirements (runtime platform, management, etc)
   - Structure of Leveransepakke
   - From Leveransepakke to Application Image
   - Application versioning (The Aurora Version) and tagging
   - Application Configuration
   - Build process (Jenkins)
 * Deploying an application and sets of applications with AOC
 * Managing applications with AOC and the Aurora Console
 
 * Implementation details:
   - A description of the underlying components and their role in the Aurora OpenShift platform


## Bits and Pieces


 - All applications must implement the management interface demands
    - must be served on a port of its own, not accessible to the internet
    - must expose prometheus metrics, healthchecks, env and other info (build metadata, git metadata, dependencies and other links)
    - must log using standard LOGBACK configuration
 - The application must handle SIGTERM gracefully

   * [Fat-jars](http://stackoverflow.com/questions/19150811/what-is-a-fat-jar) are not supported since we do a security
   check on third party dependencies with Nexus IQ.

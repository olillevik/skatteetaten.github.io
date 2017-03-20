---
layout: home
---

# The Aurora OpenShift Platform

## Introduction

Welcome to the Aurora OpenShift Platform of The Norwegian Tax Administration (TNTA?) on GitHub. The Norwegian Tax 
Administration is an agency under the authority of the Ministry of Finance and is responsible for taxation and the 
population register.

These pages on GitHub is the home of the initiative to open source the platform that runs most of the NTA's digital tax
and customs services; the Aurora OpenShift Platform.


## A Short History 

The work on what would become the Aurora OpenShift Platform started late in 2014 in the wake of a major project to 
modernize the handling of the foundational data for most tax calculations in Norway (MAG). The MAG project identified 
the need for a common platform for running and handling the vast amount of micro services across several development, 
test and reference environments that the project produced, and that would be produced by even larger projects down the 
line. 

The Norwegian Tax Administration started out as a typical Oracle shop with lots of Forms and PL/SQL. Before MAG the standard development plattform was based on Java EE in the first version of the Aurora Plattform. MAG introduced plain old Java SE as the runtime plattform in the second installment of the Aurora Plattform.

TODO: Formulate the transition between oracle and javaEE better.

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


## What is the Aurora OpenShift Platform?

Even before we started exploring OpenShift we knew that we would have to automate integration with our existing
infrastructure regardless of the platform we would end up using - so doing some sort of common initiative to provide
automation and integration services was given from the get-go. As we familiarized ourselves with OpenShift we saw
that there were several other areas that would also benefit from a central, common effort.

One of the things that struck us after getting some experience with OpenShift was how flexible it was to work with.
Although the platform certainly comes with its fair share of requirements and idiosyncrasies, we found that there was 
hardly anything we were not able to do. However, we also very quickly realized that with all this flexibility comes a
quite steep learning curve on how "to do things right", and allowing individual teams to freely develop their
own strategies on how to use OpenShift would bear with it a high cost of entry, slower internal adoption and
difficulties streamlining maintenance and deployment activities across teams. Also, there were few obvious upsides to
letting teams freely use OpenShift their own way. For instance, we were hard press to find good answers to questions
like; 
 * What would be the benefit for teams to develop and maintain their own base image for their Java applications instead 
 of collaborate on one?
 * What would be the benefit for teams to device their own Docker Image build processes instead of adopting a common one?
 * What would be the benefit for teams to develop and maintain their own scripts and tools for handing deployment and 
 configuration across different environments?
 
The Aurora OpenShift Platform is everything The Norwegian Tax Administration has developed to support infrastructure
automation, application configuration, deployment and management, common application images for technologies in use
(currently Java and Node) and common build and versioning mechanism for application archives and docker images. The
main user facing components implementing these mechanisms are:

 * Aurora Console: The most prominent feature of the Aurora OpenShift platform is the Aurora Console. The Aurora Console is a custom
built web application that we use in conjunction with the platform provided OpenShift Console to better handle
applications across teams and environments.
 * AOC: The Aurora OpenShift Command Line Client. Our custom built CLI that works with a set of high level configuration
 files that helps managing applications and configuration across environments.
 * The Aurora API: The API that implements all our custom services. Used by the Aurora Console and AOC
 * Architect: A Docker image that implements our image build process for all our supported runtime technologies
 * Base Images: A set of Alpine Linux based Docker Images that all our applications use


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
   - Leveransepakke (Application Delivery?)
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

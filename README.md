# elrr-infrastructure

This directory contains the code to install, configure, and run infrastructure associated with the Enterprise Learner Record Repository project. example

## Intended use

Intended use of this code is that a user could reference the steps below, including the associated scripts within this repository, to build a ELRR application within their own environment. The build for ELRR consists of the following components:

- ELRR Portal
- xAPI Gateway
- ELRR Database
- ELRR Staging Database
- DATASIM (for testing)
- Jenkins

Each component contains its own file and README for installation / configuration within their respective folder in the `src` folder.

## Capabilities and limitations

The code in this repository leverages Docker images for running majority of the core applications. While the code has been tested on AWS EC2 instances, the code should be able to run on any major cloud platform with parameter tweaks, but has not been verified. The current code base utilizes separate virtual machines (VMs) for each of the core components

IN the future, the development team will explore running the ELRR suite using Docker Compose and/or on Kubernetes.

## Directions for use

Navigate to the `src` folder and select the component you wish to configure. At this time, we recommend each component is installed on a separate VM for performance and stability reasons.

## Further resources

For more details on various components for ELRR, please refer to the following documentation:
* [Jenkins - Installing on Docker](https://www.jenkins.io/doc/book/installing/docker/)
* [PostgreSQL - Install on Docker](https://docs.docker.com/engine/examples/postgresql_service/)

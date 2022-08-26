# CITZ IMB Github Actions CI/CD Pipeline Demo Repository

The goal of this repository is to provide in depth documentation into how to
setup a series of github action jobs that are configured to utilize a
dockerfile/docker-compose file to build and tag any type of container image 
to BC Government Openshift. This pipeline should be agnostic to the type of
application being built and only require a Dockerfile/docker-compose file to
be developed.

For the purposes of this demonstration, this project will revolve around a 
nodejs express API that will have one endpoint (GET /api/v1/health) which
will return a http response code of 200.

This CI/CD pipeline into the BC Government Openshift cluster will utilize both
[Github Actions](https://docs.github.com/en/actions) and [Github Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment). 

If the user forking this repository is unfamiliar with either Github Actions or
Github Environments, please take the time to review the provided documentation
above before starting. The documentation provided in order to integrate this 
pipeline into a given project will give the user detail instructions into
configuring both the Github repository and Github Environments before running
the pipeline, but having context into the technologies utilized is key to 
success.

# Assumptions

1. The user will already have git installed on their local development machine
2. The user will have Docker/Docker Desktop installed on their local development
machine in order to test the Dockerfile and docker-compose file before starting
3. The user will have forked this repository before starting configuration of
the CI/CD pipeline into BC Government Openshift
4. The user has contacted @akroon3r (adam.kroon@gov.bc.ca) for required service
account information and URLs to allow for pipeline integration 

# Understanding the Nodejs Express API

The API that accompanies this repository will have one endpoint. A GET to
/api/v1/health that returns a http 200 response code. This endpoint will be used 
in Openshift by both a readiness and livesness probe attached to the API pod.
A healthcheck in the CI/CD pipeline requires the readiness probe to return a
healthy reponse (200) in order to proceed through various steps of the
deployment process in the DEV, TEST and PROD environments in Openshift.

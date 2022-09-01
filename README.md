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

1. The user will already have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed on their local development machine
2. The user will have [Docker/Docker Desktop](https://docs.docker.com/get-docker/) installed on their local development
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

# Docker 

The main dependency for the implementation of this pipeline into a given 
project ecosystem is Docker. This project will utilze a [Dockerfile](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/src/api/Dockerfile) and
a [docker-compose](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/docker-compose.yaml) file to build the container image of the Nodejs Express API.

To test that the container image is being built correctly before starting this 
tutorial, process to the root level of this respository in the CLI of your
choice and enter the following:

`docker-compose up --build express-api`

You should see the following output in CLI:

```
Starting express-api ... done
Attaching to express-api
express-api    | Example app listening on port 3000
```

You then can open any internet browser of your choosing and go to 
http://localhost:3000/api/v1/health and see the following message displayed:

`I am suuuuuuuuuuuuuuuuuuuuuuuper healthy!`

Your demo Nodejs Express API container is healthy and serving traffic on it's
only endpoint!

# Github Flow

This CI/CD pipeline hinges around a specific development workflow: [Github Flow](https://docs.github.com/en/get-started/quickstart/github-flow)

Because Github Actions are heavily event driven, with the implementation of
the Github Flow into your project, this pipeline will be heavily centered
around the creation of a new commit hash on the repository main branch when a 
pull request has passed it's CI jobs, been peer reviewed, approved and finally
merged to the main branch of the project source code.

If you don't have experience with implementing Github Flow in any of your
previous projects, please take the time to review the documentation provided
by Github (link above). This workflow is extremely lightweight, orangized and
reduces repository overhead and maintainence.

# Github Repository Setup

In this section we will start by setting up your forked repository for 
contributions by hypothetical team members via pull request.

## Branch Protection Policies

Start by entering repository settings (1), clicking on the branches tab (2) and
then finally clicking the "Add Rule" button (3).

![branch-protection-policies-1](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/branch-protection-policies-1.PNG)

First, you will start by creating the branch naming pattern:

![branch-protection-policies-2](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/branch-protection-policies-2.PNG)

Second, you will enforce pull requests to be opened before merging code to the 
main branch, as well as requiring approvals from other team members that have 
reviewed the incoming source code changes:

![branch-protection-policies-3](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/branch-protection-policies-3.PNG)

Finally, you will make sure you enable status checks to pass before merging 
incoming source code changes to the main branch. This means things like basic
CI tasks like image building, running unit tests or code quality checks.

![branch-protection-policies-4](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/branch-protection-policies-4.PNG)

Once these tasks are complete, your forked repository is ready to handle
incoming changes to source code via pull request. Pull request submission to 
your project repository is the first major event in the lifecycle of this CI/CD
pipeline.

# Github Environments - Setup and Configuration 

Github Environments utilizes environment scoped environment variables for the
environment stages we will be setting up later. First you will be setting up
the *global scope* environment variables.

| Variable Name                              | Value                                         | Notes                                                                 |
|--------------------------------------------|-----------------------------------------------|-----------------------------------------------------------------------|
| OPENSHIFT_[OBJECT]_PUBLIC_IMAGE_REPOSITORY | request from adam.kroon@gov.bc.ca             | replace [OBJECT] with "EXPRESS_API" for the purposes of this project  |
| OPENSHIT_PUBLIC_IMAGE_REPOSITORY_BASE_URL  | request from adam.kroon@gov.bc.ca             | used to docker login to with service account credentials              |
| OPENSHIFT_SERVER_URL                       | https://api.silver.devops.gov.bc.ca:6443      | used to login to openshift cluster                                    |
| OPENSHIFT_TOOLS_SA_PASSWORD                | JWT value - request from adam.kroon@gov.bc.ca | service account password                                              |
| OPENSHIFT_TOOLS_SA_USERNAME                | request from adam.kroon@gov.bc.ca             | service account username                                              |
| OPENSHIFT_TOOLS_NAMESPACE                  | request from adam.kroon@gov.bc.ca             | license plate and environment (ex// ca94a8-tools)                     |

To setup the global scope environment variables you will first click on the
repository settings tab (1), then click on the secrets tab (2), and then click 
"actions" from dropdown under the secrets tab.

![github-environments-1](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/github-environments-1.PNG)

Next click 

# Openshift Service Accout Setup

In order for your Github Actions/Environments to access specific namespaces,
you will have to setup a Service Account that has the ability to login to the
Openshift cluster to run specific oc binary commands. Due to the use of Ubuntu
as the image used to run the Github Actions/Environments, the oc binary comes 
pre-installed.

You will first setup the github-actions service account for your projects TOOLS 
namespace. First, login to the BC Goverment Openshift cluster via CLI using the
OC binary. Make sure that you have selected your projects TOOLS namespace as 
your current working space by doing:

`oc project [LICENSE_PLATE]-tools` 

Where you will replace [LICENSE_PLATE] with your projects randomized 6 
character hash. Next, navigate to the `openshift/templates/service-account/`
directory of this github repository in CLI. Run the following command to 
create your github-actions service account.

`oc process -f github-actions.yaml -p ENVIRONMENT=[ENVIRONMENT] -p LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

Again, replace the [ENVIRONMENT] and [LICENSE_PLATE] sections of the command
above with your target environment (in this case, *tools*) and your projects
randomized 6 character hash for the license plate.

If you toggle to Administrator view in the Openshift webapp GUI, and navigate
to the *User Management* tab on the left hand side of the page, and select the
*Service Accounts* section, you should see your service account of 
github-actions has been created.

![openshift-service-account-5](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-5.PNG)

Next, toggle the Openshift GUI to developer mode, and select the Secrets tab on
the left hand side of the page. You'll see 3 new secrets that have been
created. A github-actions docker config secret, and two token secrets. The two
token secrets will have a 5 character randomized hash appended to the end of
the name. The secrets that are created for your service account *will be 
different*. Do not be alarmed.

![openshift-service-account-1](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-1.PNG)

Click on the first secret token value, then click on the *Reveal Values* toggle
on the lower right had side of the page.

![openshift-service-account-2](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-2.PNG)

The value that you'll be most be interested in is the token field found at the
extreme bottom of the page. This is the value that will act as the service
account password. Copy the entirity of this value into your local clipboard,
so you can next configure this value as a secret into Github.

![openshift-service-account-3](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-3.PNG)

 Because the first namespace you setup the Github Actions service account in
 was the TOOLS namespace, this username and password will be considered a 
 global scope environment variable. To configure these values, first click on
 the *Settings* tab of your Github repository, then the *Actions* tab on the left 
 side of the page, and finally the *secrets* option out of the dropdown under
 Actions. On the next page, click *New Repository Secret* and enter 
 `OPENSHIFT_TOOLS_SA_USERNAME` as the name and `github-actions` as the value,
 then click *Add secret*. Repeat this process for the `OPENSHIFT_TOOLS_SA_PASSWORD`
 environment variable with the value as the complete JWT copied from the 
 github-actions token secret opened in the previous step.

 Next you will want to complete the above steps for the DEV, TEST and PROD
 namespaces. You'll want to start with creating the service account using the
 provided kubernetes template again, except re-running the oc command replacing
 the ENVIRONMENT value with DEV, TEST and PROD separately.

`oc process -f github-actions.yaml -p ENVIRONMENT=DEV -p LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

`oc process -f github-actions.yaml -p ENVIRONMENT=TEST -p LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

`oc process -f github-actions.yaml -p ENVIRONMENT=PROD -p LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

To set your first username/password values for the DEV environment, you will
first click the *Settings* tab at the top of your Github repository, then the
*Environments* tab on the left side of the page. Select the DEV environment that
you created earlier in this tutorial and then click *Add Secret*. The name for 
the username environment variable will be `OPENSHIFT_SA_USERNAME` with a value
of `github-actions`. The password variable will be named `OPENSHIFT_SA_PASSWORD`
with a value of the token found in the github-actions-token found specifically
in the DEV namespace. You will repeat these steps with the exact same name used
for username and password environment variables for the TEST and PROD 
namespaces, using the specific github-action token JWT values found in the TEST
and PROD namespaces respectfully.

![openshift-service-account-6](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-6.PNG)

![openshift-service-account-7](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-7.PNG)

![openshift-service-account-8](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-service-account-8.PNG)

The remainder of the Github Environment variables that need to be set up can be found in the following table:

| Variable Name                              | Value                                           | Notes                                                                 |
|--------------------------------------------|-------------------------------------------------|-----------------------------------------------------------------------|
| APPLICATION_NAME                           | Name of your target application                 | example: dpia-api                                                     |
| ENVIRONMENT                                | dev/test/prod                                   | case sensitive                                                        |
| LICENSE_PLATE                              | randomized 6 character hash of your namespaces  | example: ec1236                                                       |
| OPENSHIFT_[ENVIRONMENT]_NAMESPACE          | LICENSE_PLATE+ENVIRONMENT                       | example: ec1236-dev  ALSO replace [ENVIRONMENT] with DEV/TEST/PROD in variable name |
| GLOBAL_USER_EMAIL                          | email address of github user                    | this is used as a part of repo config within the actions script       |
| GLOBAL_USER_NAME                           | name of github user                             | this is used as a part of repo config within the actions script       |

Congratulations! You should have successfully created access for your service
account to access the BC Government Openshift cluster namespaces for your
project, as well as your first Github Actions and Github Environments 
environment variables.

# Openshift Network Policy Setup 

In order for your service account to login and connect with your project
namespaces within the BC Government Openshift cluster, you will next need to
configure baseline network policies that management traffic into your 
namespaces from the public external internet, as well as policies that manage
network traffic between your namespaces as well as between services within your
namespaces.

*! IMPORTANT !* These network policies by default are set to be wide open. A 
typical project will then pair back their network policies to allow the traffic
communications required for their project to work and deny all other network
communications. This tutorial will not cover the pairing back of policy
permissions.

To create the required network policy objects in the TOOLS, DEV, TEST and PROD 
namespaces, run the following commands after you have navigated to the 
`openshift/templates/network-policies/` directory in this repository:

`oc process -f .\any-to-any.yaml -p ENVIRONMENT=[ENVIRONMENT] LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

`oc process -f .\any-to-external.yaml -p ENVIRONMENT=[ENVIRONMENT] LICENSE_PLATE=e[LICENSE_PLATE] | oc apply -f -`

`oc process -f .\allow-from-openshift.yaml -p ENVIRONMENT=[ENVIRONMENT] -p LICENSE_PLATE=[LICENSE_PLATE] | oc apply -f -`

Remember to replace `[ENVIRONMENT]` with a value of tools, dev, test or prod
and `[LICENSE_PLATE]` with your namespaces randomized 6 character hash.

To see the network policies that were successfully created, toggle developer
mode on in the Openshift webapp GUI, then click on the *Search* tab on the left
side of the page. Under the *Resources* dropdown, type "network" into the
search bar. You should see an object with a blue NP icon called *Network Policy*.
Select this option.

![openshift-network-policy-setup-1](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/openshift-network-policy-setup-1.PNG)

# Imagestream Setup 

The initial stage on the Github Environments Actions is a build-tag-push stage.
In order to do so, we need somewhere to put the image that is built as result 
of this phase. The Kubernetes imagestream object is what we need to setup first
in order to accomplish this.

Toggle the Openshift webapp to Administrator mode, and then click on *Builds*.
From the dropdown that appears under Builds, select *Imagestreams*.

![imagestream-setup-1](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/imagestream-setup-1.PNG)

Next, you will want to create an imagestream for your image. Do this by
clicking on the blue *Create Imagestream* button in the top right corner of the
page.

![imagestream-setup-2](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/imagestream-setup-2.PNG)

You'll see the inline yaml editor that comes with Openshift on the next page.
This object only requires a name and no other metadata. Simply replace the 
"example" value found in the name field with the name of the application image
you're looking to build and store. When you've changed the value, click the
blue *Create* button found at the bottom of the text form.

![imagestream-setup-3](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/imagestream-setup-3.PNG)

The next page should look something similar to this:

![imagestream-setup-4](https://github.com/bcgov/citz-imb-actions-pipeline-demo/blob/main/assets/images/imagestream-setup-4.PNG)

You are now ready to build, tag and push your Docker image into the Openshift
image repository.

# Docker and Docker Compose

This CI/CD pipeline utilizes both a Dockerfile and a Docker Compose file to
build the example Express API image in used in this tutorial. Please make
you review both files before testing to understand how the image is built.

# Testing the Build/Tag/Push Github Action Stage

In the .github/workflows directory on this repository, you will find the
following file:

`build-tag-push-deploy-express-api.yaml`

This file in it's entirety is used to push a specific feature and/or change in
the source code from your Openshift DEV environment, through TEST and finally 
into PROD. Before you test the entire pipeline, you should first ensure that 
the docker container image is correctly built and pushed into the imagestream 
that you created in the previous step. You will notice that lines 80 to 557 are 
commented out for this specific purpose.

*! IMPORTANT !* Please note that line 68 of the yaml file being used may have
to change based on the name of your forked repository.

*Current value*: docker tag citz-imb-actions-pipeline-demo_express-api

*Your value*: docker tag [YOUR_REPOSITORY_NAME]_express-api

# Openshift Kubernetes Object Setup 

# Tagging Strategy - Development Lifecycle

# Tagging Strategy  - Operational Lifecycle 

# Utilizing the Pipeline

# FIN

Congratulations. If you've gotten this far in the instructions provided, you
have either skipped all the instructions to find out how the story ends, or 
you've successfully integrated this pipeline into your forked Github repository
and are (successfully!) deploying a Nodejs Express API to Openshift DEV, TEST 
and PROD namespaces.

As previously stated, this pipeline should be completely agnostic to your
project and the technologies you choose in order to develop any given
application or applications in your architecture. The only requirement is that
your project utilize Docker, Dockerfiles and a docker-compose file to build
your container images.

Feedback is on this CI/CD pipeline is encouraged. Please utilize the provided 
Issue template to provide feedback to this repository and its maintainers.
Hopefully we've made your life easier by the end of this!

When you have finished testing this pipeline, please contact @akroon3r
(adam.kroon@gov.bc.ca) to cleanup the Openshift namespaces that you've
deployed into. Thanks!

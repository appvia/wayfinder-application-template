
# wayfinder-app

This repository is a template for creating a new application repository for Wayfinder. It contains a workflow that will build and deploy a simple example application to a Kubernetes cluster managed by Wayfinder.

The contents of this repo needs to be tailored to suit your needs, e.g. the source folder needs to be replaced with your application code.

## Prerequisites

If you created this repo using a [create-wf-app workflow](https://github.com/Digital-Garage-ICL/create-wf-app/actions) for students, All of the pre-requisites should've been done for you, except those that I needed within step 6 details about how to use the code can be found as comments on the script.

Here's a list of pre-requisites for this repository to work:

1. A Wayfinder instance. You will find the URL for the Wayfinder portal in the `Resources info` step of the [create-wf-app workflow](https://github.com/Digital-Garage-ICL/create-wf-app/actions) that you've run.

2. A Workspace created. You can also find the name of the Wayfinder workspace in the `Resources info` step of the create-wf-app workflow.

3. A Kubernetes cluster created and available to use in your workspace. A shared student cluster is available for you to use. The [AppEnv](./infra/appenv.yaml) resource points to it.

4. Permission on this repository to add secrets and trigger Github workflows. You should be owner of this repository if you created it using the create-wf-app workflow.

5. Wayfinder CLI. Read more [here](https://docs.appvia.io/wayfinder/getting-started/cli)

6. gh and jq installed. See more in comments section at the beginning of the [setup.sh](./setup.sh) script.

## Setup Repository

***IMPORTANT*** You need to log in the Wayfinder portal and GitHub CLI for the setup to work. You can find the URL for the Wayfinder portal in the `Resources info` step of the create-wf-app workflow.

To check that you have logged in to Wayfinder, run `wf whoami` in the terminal. You should see your username (e-mail address).

Also make sure you log into the github cli using `gh auth login` and follow the instructions.
  
Before you start, you will need run the setup.sh script to setup the repo. This will create the required secrets and Variables for the workflow to work and set up access to the Wayfinder workspace for the CI pipeline.

 1. Clone the repository you've created using the create-wf-app workflow. You can find the URL in the `Resources info` step of the create-wf-app workflow.

 2. Open the terminal and navigate to the root of this repository then
    run the following command:

	```bash
	./setup.sh  ${WORKSPACE_NAME} ${GITHUB_PERSONAL_ACCESS_TOKEN}
	```
	
	Where:
	- `${WORKSPACE_NAME}` is the name of the Wayfinder workspace you've created using the create-wf-app workflow.
	- `${GITHUB_PERSONAL_ACCESS_TOKEN}` is a Github personal access token with `read:packages` scope. Make sure you create a `Classic` token. You can find more information about how to create a token [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic)

Once this is complete the script will generate the relevant secrets, and variables for the workflow to work. Now you can start configuring the repository for your application.

You should now re-run the `deployment` GitHub workflow that got triggered when you created this repository. This will deploy the application to the Kubernetes cluster. In your repo's UI on GitHub, go to Actions. You should see a workflow run with a name of "Initial commit" that has failed. Re-run it by clicking on the "Re-run all jobs" button.

The workflow initially failed because the CI token was not authorised to do the things it needs to do (creating an application, an environment and deploying an application to that environment). You created and authorised the token in the previous step (running `./setup.sh`). The workflow should now succeed.


## Creating your image
  
There is a CI workflow, which can take your Dockerfile and build an image for you. This is the recommended way to build your image.

 - You will need to adjust your Docker file for your specific application.  
 - Then CI workflow will be triggered on a push to main branch.
 -  The image will be created in GitHub registry which can be located here

>   https://github.com/Digital-Garage-ICL/{repo-name-placeholder/}/pkgs/container/{repo-name-placeholder/}


## Configure Deployment

### Introduction wayfinder Concepts

**Applications** are a way to model the elements of your applications (containers, cloud resources, environments, etc) to simplify deployment and provisioning. Applications should consist of things that follow the same software lifecycle and would typically be deployed together, although individual components can be deployed separately.

Applications are linked to components​. There are two components​ types

- `Container components`

- `Cloud resource components`


**Container components​**  is what is used to deploy your container.

**Cloud resource components** A cloud resource component represents a dependency of your application which is served by a cloud service, such as a database, storage bucket or message queue. It requires a cloud resource plan to be specified when defined, and that represents a Terraform module that administrators have made available to your workspace.

Applications are also linked to `Environments`.

  
**Environments** map to namespaces in Kubernetes. Kubernetes namespaces provide a mechanism for isolating groups of resources within a single cluster. You can deploy your application into an environment which uses existing infrastucture.

  
If you created this repo using [create-wf-app workflow](https://github.com/Digital-Garage-ICL/create-wf-app/actions). You will have a demo `Application`, `AppComponent` and `Environments` created which will be setup with the right workspace and application name.

## Change Configure

You can find the `Application` `AppComponent` and `Environments` in the folder infra/.
Documentation on how to amend these files can be found [here](https://docs.appvia.io/wayfinder/crd#app.appvia.io%2fv2beta1) 

For this demo we will need to make chagne to `infra/storage-account.yml` . You will need to replace `NAME` in the below code. This is because a storage account will be created and requires a unique string which is up to 24 alphanumeric lowercase characters.




or you can use the UI to configure the components then get the new configuration by using then copy that into a file with infra/

- `wf get app {app-name-placeholder} -o yaml` 
 - `wf get appcomponent  {app-name-placeholder}-{component-name} -o yaml`

For this work we do not want you to change the `Environments` file.

### Deploy

We have a workflow set up called you ci.yaml, which will be triggered on merge to the main branch or a manual trigger.


## Validating terranetes

Your configuration for your cloud resources can fail or pass for reasons, not controlled by yourself. For example, when you're making a Storage account you need to specify the name. You may provide a reasonable name, however, it is not unique and has already been registered within Azure. So your terraform plan will succeed, but in your apply, it will fail.

Terranetes creates jobs for your terraform plan and apply. The pods that are created are short lived so they will be removed after a certain period of time so it's important that you check your logs within those pods.

For more information about terranetes [here is the documentation](https://terranetes.appvia.io/terranetes-controller/category/developer-docs/)

### Here's how to check your Terraform plan and apply.

- Login to wayfinder
> wf login -a https://api-20-0-245-170.go.wayfinder.run
- Login to the cluster
> wf use workspace workspace-placeholder
wf access env app-name-placeholder dev
- List all the pods
> kubectl get pods
- Take a look at the plan pod logs
> kubectl logs {pod name}
- If this is successful, you can then take a look at the apply pod logs in the same way
> kubectl logs {pod name}
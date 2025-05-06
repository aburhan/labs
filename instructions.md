# Developer Experience

## Overview
Welcome to this lab, where you will learn how to automate infrastructure and application deployment with minimal inputs using Google Cloud. This hands-on experience leverages Google Cloud Build and Google Cloud Deploy to automate the deployment and management of infrastructure on Google Cloud.

As an SRE or Platform Engineer, this lab equips you with essential skills to improve reliability, scalability, and maintainability of your infrastructure based on the needs of developers while aligning with modern DevOps best practices.

As a developer you often have to deal with a lot of things that are not writing code such as infrastructure provisioning, manually deploying your code perform AB testing. This lab is designed to show you how to remove that burden off the shoulders of Developers using GCP. 

This lab works with the following core concepts:

***Platform Engineering***<br>
Platform engineering focuses on creating and managing the underlying infrastructure that enables developers and operations teams to deliver software efficiently, reliably, and at scale. 

***Application Factory***<br>
A system or platform that allows organizations to rapidly build and deploy multiple applications quickly and efficiently

## Lab
In this lab, you will take on two personas: <strong>Platform Engineer</strong> and <strong>Developer</strong>.
This project represents the live, production environment. Platform Engineers will deploy and manage resources within this project, while Developers will have access to monitor and verify the deployed applications and infrastructure.
This project will be used by both Platform Engineer![usr1](img/qwiklabs-icons1-small.drawio.png) who will deploy resources, and Developer ![usr2](img/qwiklabs-icons-2-small.drawio.png) will access the resources.


| <strong>Resources</strong>| <strong>Platform Engineer Tasks</strong> ![usr1](img/qwiklabs-icons1-small.drawio.png)| <strong>Developer Tasks</strong>![usr2](img/qwiklabs-icons-2-small.drawio.png)|
|--------| ----------------- | --------- |
| Git Repository 🐱|  **•** Create a new branch for infrastructure updates. <br> **•** Define environment variables for the deployment (e.g., image tag, replica counts). <br> **•** Modify Kubernetes deployment YAMLs to implement Canary deployment (e.g., using Deployment and Service objects). <br> **•** Commit and push changes to [trigger](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers) the Cloud Build pipeline. | **•** Validate, Read, Write and Push code changes to the repository. |
|Application Code (Online Boutique) | **•** No direct modification of application code. | **•** Make code changes to the Online Boutique application. <br>**•** Build and test the changes locally. <br>**•** Commit and push code changes to a separate application repository (this may trigger a different build/deploy pipeline for staging).<br>**•** Observe the impact of code changes on the production Canary (e.g., through logs, metrics, and user feedback). |
| Cloud Build | **•** Configure [Cloud Build](https://cloud.google.com/build/docs/overview) triggers to automatically build and deploy application updates to the GKE cluster. <br>**•** Utilize environment variables within Cloud Build to control deployment parameters. |  No direct interaction with Cloud Build. | 
|GKE Cluster | **•**  Deploy the updated application to the [GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview) cluster using a Canary deployment strategy.<br>**•** Monitor the Canary deployment rollout and application health using Google Cloud Console or kubectl.<br>**•** Observe traffic distribution between the two versions of the application. |  No direct access to the GKE cluster. |



## Objectives

As a Platform Engineer ![usr1](img/qwiklabs-icons1-small.drawio.png), you are tasked with building automated CI/CD pipelines with Cloud Build and Terraform, deploying applications to various environments using [Cloud Deploy](https://cloud.google.com/deploy/docs/overview) and environment variables, and performing and monitoring canary deployments on GKE. 

As a Developer ![usr2](img/qwiklabs-icons-2-small.drawio.png), you are tasked to modify code and analyze the application deployed in the current environment.

### Architecture
![ach](img/arc-update.png)

### <strong>Before you click the Start Lab button: </strong>

This is an Advanced lab it is designed for Site Reliability Engineers (SREs) / Platform Engineers / Infrastructure Engineers, this lab assumes users have:

1. Basic knowledge of Terraform syntax and configuration files.
2. Familiarity with CI/CD concepts.
3. An understanding of Google Cloud Platform (GCP) fundamentals, including IAM, networking, Kubernetes and Compute Engine.

## How to start your lab and sign in to the Google Cloud Console
Read these instructions. Lab environments are time-limited and cannot be paused. The timer starts when you begin the lab and shows the remaining time for accessing the <strong>Google Cloud resources</strong>.

This hands-on lab allows you to complete activities in a real cloud environment, not in a simulation or demo environment. It does so by giving you new, temporary credentials you use to sign in and access Google Cloud for the duration of the lab.

<strong>To complete this lab, you need:</strong>
   * Access to a standard internet browser (Chrome browser recommended).
   * Ensure you have enough time to complete the lab, as it cannot be paused once started.

<strong>How to start your lab and sign in to the Google Cloud console</strong>
1. Click the <strong>Start Lab</strong> button. If you need to pay for the lab, a dialog opens for you to select your payment method. On the left is the Lab Details pane with the following:

   <img src="img/startlab1.png" alt="startlab" width="179.00">

2. Add <strong>Platform Engineer</strong> and <strong>Developer users</strong> in the Chrome profile.

*  Follow the steps to add the Chrome users: 

1. Click on the three vertical dots in the top-right corner of Chrome.

   <img src="img/Profile1.png" alt="Profile" width="600.00">


2. Navigate to the <strong>Add New Profile</strong> and click it.

   <img src="img/Profile2.png" alt="Profile" width="441.00">


3. Click on the <strong>Continue without an account</strong>.

   <img src="img/Profile3.png" alt="Profile" width="441.00">



4. Add Platform Engineer in the <strong>Welcome Text Editor box</strong> and Click on Done. 

   <img src="img/Profile4.png" alt="Profile" width="400.00">



* Follow Steps 1 to 3 and add the Developer user Profile.
5. Add Developer in the<strong>Welcome Text Editor box</strong> and Click on Done. 

   <img src="img/Profile5.png" alt="Profile" width="400.00">



6. Click on the user icon to verify the profile switch. 

   <img src="img/Profile6.png" alt="Profile" width="400.00">



<ql-infobox>
<strong>Tip:</strong>  Create two Chrome profiles for Platform Engineer and Developer personas to easily switch between them
</ql-infobox>

3. As a Platform Engineer ![usr1](img/qwiklabs-icons1-small.drawio.png) Copy this URL <i><strong>"console.cloud.google.com"</strong></i> and paste it in the previously created Platform Engineer Chrome profile, The lab will initialize resources and display <strong>Sign-in</strong> in page.

<ql-infobox>
<strong>Tip:</strong> Open the tabs in separate windows, side-by-side.
</ql-infobox>

4. ![usr1](img/qwiklabs-icons1-small.drawio.png) In the <strong>Sign in</strong> page, paste the <strong>Platform Engineer username</strong> that you copied from the left panel. Then copy and paste the password.
<strong>Important:</strong> You must use the credentials from the left panel. Do not use your Google Cloud Training credentials. If you have your own Google Cloud account, do not use it for this lab (avoids incurring charges).

5. Click <strong>Next</strong>.
6. Proceed through the following steps:
    1. Accept the terms and conditions.
    2. Do not add recovery options or two-factor authentication (because this is a temporary account).
    3. Do not sign up for free trials.

After a few moments, the <strong>Google Cloud console</strong> opens in this tab.

The lab spins up resources, and then opens another tab that shows the <strong>Sign in</strong> page.


<ql-infobox>
<strong>Tip:</strong> Arrange the tabs in separate windows, side-by-side.
</ql-infobox>

After a few moments, the Cloud Console opens in this tab.

<ql-infobox>
<strong>Note:</strong> You can view the menu with a list of Google Cloud Products and Services by clicking the Navigation menu at the top-left. 
</ql-infobox>

   <img src="img/startlabs4.png" alt="startlab" width="500.00">

7. As a Developer ![usr2](img/qwiklabs-icons-2-small.drawio.png) Copy this URL <i><strong>"console.cloud.google.com"</strong></i> and paste it in the previously created Developer Chrome profile, The lab will initialize resources and display <strong>[Sign-in](console.cloud.google.com)</strong> in page.

<ql-infobox>
<strong>Tip:</strong> Open the tabs in separate windows, side-by-side.
</ql-infobox>

8. ![usr2](img/qwiklabs-icons-2-small.drawio.png) In the <strong>Sign in</strong> page, paste the <strong>Developer username</strong> that you copied from the left panel. Then copy and paste the password.
<strong>Important:</strong> You must use the credentials from the left panel. Do not use your Google Cloud Training credentials. If you have your own Google Cloud account, do not use it for this lab (avoids incurring charges).

9. Click <strong>Next</strong>.
10. Proceed through the following steps:
    1. Accept the terms and conditions.
    2. Do not add recovery options or two-factor authentication (because this is a temporary account).
    3. Do not sign up for free trials.

After a few moments, the <strong>Google Cloud console</strong> opens in this tab.

The lab spins up resources, and then opens another tab that shows the <strong>Sign in</strong> page.


<ql-infobox>
<strong>Tip:</strong> Arrange the tabs in separate windows, side-by-side.
</ql-infobox>

After a few moments, the Cloud Console opens in this tab.

<ql-infobox>
<strong>Note:</strong> You can view the menu with a list of Google Cloud Products and Services by clicking the Navigation menu at the top-left. 
</ql-infobox>

   <img src="img/startlabs4.png" alt="startlab" width="500.00"> 

## Task 1. Pre-requisite ![usr1](img/qwiklabs-icons1-small.drawio.png)

Once you login as a user in the Platform Engineer Chrome profile ![usr1](img/qwiklabs-icons1-small.drawio.png), you will be able view your project, you can find the project ID in the left sidebar of the lab interface.

### Steps:
**Before you start:** You'll need a Cloud Source Repository to store the application code and configuration files.

## Update the repository.  ![usr1](img/qwiklabs-icons1-small.drawio.png)

Update the Repository to perform the next task in the Lab. <strong>Run</strong> the Following command in <strong>host project Cloud Shell</strong>. 

<strong>Note: </strong>
 - Platform Engineer needs to execute the  following command to proceed ahead and replace the `project-id`, `user_email` and `user_name` mentioned in the lab instructions tab. <br>
 Example : <br>
 <strong>user_email :</strong> `student-01-f25be5766c74@qwiklabs.net`<br>
 <strong>user_name  :</strong> `student-01-f25be5766c74`

```
 export PROJECT_ID=project-id
 export USER_EMAIL=user_email
 export USER=user_name
```
```
gcloud source repos clone qwiklab-developer-experience --project=$PROJECT_ID
gcloud source repos clone developer-experience --project=qwiklabs-resources
```
```
cd qwiklab-developer-experience
cp -rvf  ../developer-experience/* .
git add -A
git config user.email "$USER_EMAIL"
git config user.name "$USER"
git commit -m "initial commit" -a
git push -o nokeycheck origin main
```

## Explore the resource in the Project ![usr1](img/qwiklabs-icons1-small.drawio.png)
Once you login as a user in the Platform Engineer Chrome profile![usr1](img/qwiklabs-icons1-small.drawio.png), You will be able view your project, you can find the project ID in the left sidebar of the lab interface.

In this Section you are going to explore the pre-created resources that require in the next steps.
Now, that you have logged as a <strong>Platform Engineer</strong> in the project.

 1. Go to the IAM section. 
 2. Find a service account named <strong>prod-sa</strong>.
   * <ql-infobox>Note that This service account will have Storage Admin and logging admin Access.</ql-infobox>

 3. **Navigate to Cloud storage:** Click on the <strong>Navigation menu (☰)</strong> on the left side of the console to locate <strong>"Cloud Storage"</strong>. It's under <strong>"[Buckets](https://cloud.google.com/storage/docs/buckets)"</strong>.

 4. **Select Your Buckets:** Copy the bucket name; you will use it in the Cloud Build environment variable.

## Task 2. Setting up the environment ![usr1](img/qwiklabs-icons1-small.drawio.png)

Now that you have logged in to Project. On the top right corner, you can see an icon of <strong>Cloud shell</strong>, Click on the Cloud shell icon, GCP provision the <strong>Cloud shell instance</strong> and provide the prompt.

<img src="img/cloudshell.png" alt="IAM2" width="500">

Once the shell is active let's then run following command in cloud shell to create terraform plan trigger and terraform apply trigger.

Note: 
 - Platform Engineer needs to execute the  following command to proceed ahead and replace the `project-id` mentioned in the lab instructions tab. 
 - Click on <strong>Navigation menu</strong> and select <strong>Cloud storage</strong>. Locate the <strong>bucket</strong>, copy its name, and replace the `bucket-name`.

```
 export PROJECT_ID=project-id
 export TF_BUCKET_NAME=bucket-name
```
   
1. To create terraform infra trigger. Copy and paste the below command into the shell and execute.
```
  gcloud builds triggers create manual --name="terraform-infra-trigger" \
   --description="terraform plan cloud build trigger." \
   --service-account="projects/$PROJECT_ID/serviceAccounts/prod-sa@$PROJECT_ID.iam.gserviceaccount.com" \
   --repo="https://source.developers.google.com/p/$PROJECT_ID/r/qwiklab-developer-experience" \
   --repo-type=CLOUD_SOURCE_REPOSITORIES  \
   --region="us-central1" \
   --branch="main" \
   --build-config="qwiklabs-tf/terraform/env/cloudbuild-infra.yaml" \
   --substitutions=_ENV=prod,_PROJECT_ID=$PROJECT_ID,_TF_BUCKET_NAME=$TF_BUCKET_NAME
 ```  

 2. Run the below command in cloud shell to excute infra trigger.

```
   gcloud beta builds triggers run terraform-infra-trigger --project=$PROJECT_ID --branch=main --region=us-central1
 ```
3. Navigate to the <strong>cloud build</strong> using <strong>Navigate Menu</strong> .
4. click on <strong>history</strong>.
5. You can find the <strong>terraform-infra-trigger</strong> logs. 

**Trigger: terraform-infra-trigger**

   * The trigger gets activated when you trigger it manually, this trigger executes the `qwiklabs-tf\terraform\env\cloudbuild-infra.yaml` file.
   * Terraform scripts (main.tf, terraform.tfvars, variables.tf) define the infrastructure.
   * Terraform uses a tfstate file to track the current state of the infrastructure.
   * The cloudbuild-infra.yaml configures Terraform to store the tfstate file in a Google Cloud Storage (GCS) bucket, ensuring state persistence and collaboration.
   ```
   - name: 'gcr.io/cloud-builders/gcloud'
    id: Generate manifest for GCP resource deployment
    dir: qwiklabs-tf/terraform/env
    entrypoint: /bin/sh
    args:
      - '-c'
      - | 
        sed -e "s/TF_BUCKET_NAME/${_TF_BUCKET_NAME}/g"   -e "s/ENV/${_ENV}/g"  backend.tf.tpl > backend.tf;
        sed -e "s/PROJECT_ID/${_PROJECT_ID}/g"  -e "s/ENV/${_ENV}/g" terraform.tfvars.tpl > terraform.tfvars;
        sed -e "s/PROJECT_ID/${_PROJECT_ID}/g"   providers.tf.tpl > providers.tf;
   ```
   * Terraform init: Downloads necessary provider plugins and initializes the Terraform working directory.
   ```
   - name: 'hashicorp/terraform:1.6.1'
   dir: qwiklabs-tf/terraform/env
   args:
      - '-c'
      - |
      terraform init
   id: terraform init
   entrypoint: sh
   ```
   * Terraform validate: Checks the syntax and internal consistency of the Terraform configuration.
   ```
   - name: 'hashicorp/terraform:1.6.1'
    dir: qwiklabs-tf/terraform/env
    args:
      - '-c'
      - |
        terraform fmt
    id: terraform fmt
    entrypoint: sh
   - name: 'hashicorp/terraform:1.6.1'
    dir: qwiklabs-tf/terraform/env
    args:
      - '-c'
      - |
        terraform validate
    id: terraform validate
    entrypoint: sh
   ```
   * Terraform plan: Generates an execution plan, displaying the changes Terraform will make.
   ```
   - name: 'hashicorp/terraform:1.6.1'
    dir: qwiklabs-tf/terraform/env
    args:
      - '-c'
      - |
        terraform plan 
    id: terraform plan
    entrypoint: sh
   ```
   * Terraform apply: Executes the plan, creating the infrastructure.
   ```
   - name: 'hashicorp/terraform:1.6.1'
    dir: qwiklabs-tf/terraform/env
    args:
      - '-c'
      - |
        terraform apply -input=false -auto-approve
    id: terraform apply
    entrypoint: sh
   ```
   * Following resources are created in this step using terraform scripts.
      * Terraform creates essential cloud resources:
         * API enablement.
         * Service accounts and IAM roles.
         * Virtual Private Cloud ([VPC](https://cloud.google.com/vpc/docs/vpc)), [subnets](https://cloud.google.com/vpc/docs/subnets), [NAT](https://cloud.google.com/nat/docs/overview), [routers](https://cloud.google.com/network-connectivity/docs/router/concepts/overview), and [elastic IPs](https://cloud.google.com/compute/docs/ip-addresses/configure-static-external-ip-address).
         * [Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images).
         * Virtual Machines ([VMs](https://cloud.google.com/compute/docs/instances)) and Google Kubernetes Engine (GKE) clusters.
         * Secrets in [Secrets Manager](https://cloud.google.com/secret-manager/docs/creating-and-accessing-secrets).
         * [Firewall](https://cloud.google.com/firewall/docs/firewalls) Rules.
   * The overall goal is to establish the foundational infrastructure required for application deployment, with terraform managing the state of the infrastructure, all executed by cloud build.

## Task 3. Update the GKE Cluster ![usr1](img/qwiklabs-icons1-small.drawio.png)

1. In the <strong>Google Cloud console</strong>, Use the navigation menu on the left side of the console to locate <strong>Kubernetes Engine</strong> It's under <strong>Cluster</strong>
2. Click on  <strong>Cluster Name</strong> and scroll down at the bottom.
3. You can see the <strong>Service Mesh</strong> under <strong>Features</strong> Click on edit icon. 

   <img src="img/service-mesh-edit.png" alt="service mesh" width="600.00">   

4. Click on <strong>Make Changes</strong>.
5. Do not perform any other step until the [Service mesh](https://cloud.google.com/service-mesh/docs/overview) is completely provisioned by GKE.
6. Once Service mesh available you will be able to see the Dashboard link.

   <img src="img/service-mesh.png" alt="service mesh" width="600.00">

## Task 4. Create Application Deployment Cloud build Trigger Resource ![usr1](img/qwiklabs-icons1-small.drawio.png)
In this step let's create one cloud build trigger that will create 3 Cloud build trigger, to deploy the online boutique application.

Execute the below command in cloud shell. 

```
  gcloud builds triggers create manual --name="terraform-cloud-build-trigger" \
   --description="create three cloud build trigger." \
   --service-account="projects/$PROJECT_ID/serviceAccounts/prod-sa@$PROJECT_ID.iam.gserviceaccount.com" \
   --repo="https://source.developers.google.com/p/$PROJECT_ID/r/qwiklab-developer-experience" \
   --repo-type=CLOUD_SOURCE_REPOSITORIES  \
   --region="us-central1" \
   --branch="main" \
   --build-config="qwiklabs-tf/terraform/env/cloud-build/cloud-build.yaml" \
   --substitutions=_ENV=prod,_REPO_NAME=qwiklab-developer-experience,_PROJECT_ID=$PROJECT_ID,_TF_BUCKET_NAME=$TF_BUCKET_NAME
 ```  

```
   gcloud beta builds triggers run terraform-cloud-build-trigger --project=$PROJECT_ID --branch=main --region=us-central1
 ```

3. Navigate to the <strong>cloud build</strong> using <strong>Navigate Menu</strong> .
4. click on <strong>history</strong>.
5. You can find the <strong>terraform-cloud-build-trigger</strong> logs. 

**Trigger: terraform-cloud-build-trigger**
   * Once The terraform-cloud-build-trigger is initiated.
   * Cloud Build executes the steps defined in `qwiklabs-tf/terraform/env/cloud-build/cloud-build.yaml`.
   * The terraform code and the `cloudbuild.yaml` file are in the same directory.
   * Similar to the previous trigger, the `cloud-build.yaml` file likely includes steps for:
      * terraform init: To initialize the Terraform working directory.
      * terraform apply: To create the three Cloud Build triggers.
      * The yaml file will contain the steps to execute the terraform commands.
   * The cloud-build.yaml file contains Terraform commands that define the creation of three Cloud Build triggers:
      1. **cloud-deploy-trigger**
      2. **application-deployment-trigger**
      3. **canary-deployment-trigger**
   * Terraform successfully creates the three specified Cloud Build triggers.
   * All the of terraform scripts that are used to create three trigger are present on this directory `qwiklabs-tf/terraform/env/cloud-build/ `, In those scripts you can view in detail how the triggers are created using terraform.
   * Below is the values file of terraform script which was created for trigger creation, In below code block we have provided values for first trigger (cloud-deploy-trigger) likewise other two triggers are created.
      
      ```
      cloud_build_main_branch_cloud_deploy_trigger = {
      name        = "cloud-deploy-trigger"
      description = "cloud build trigger for Cloud Deploy"
      uri         = "CODE_REPOSITORY_NAME"
      location    = "us-central1"
      branch      = "main"
      path        = "qwiklabs-tf/terraform/env/cloud-deploy/cloud-deploy.yaml"
      service_account = "projects/PROJECT_ID_/serviceAccounts/prod-sa@PROJECT_ID_.iam.gserviceaccount.com"
      disabled        = true
      substitutions   = {   "_TF_BUCKET_NAME"="_TF_BUCKET_NAME_",
                              "_PROJECT_ID"="PROJECT_ID_"
                           "_ENV"="_ENV_"
                        }
      }
      ```
   * Immediately after creating the triggers, the <strong>cloud-deploy-trigger</strong> is automatically activated. This is likely configured within the Terraform code or the Cloud Build trigger definition itself.


## Check Deployed Resource ![usr1](img/qwiklabs-icons1-small.drawio.png)
In this step you will check the deployed resources.
1. Use <strong>Platform Engineer</strong> Chrome profile.
2. In the <strong>Google Cloud console</strong>, Use the navigation menu on the left side of the console to locate <strong>Kubernetes Engine</strong> It's under <strong>Cluster</strong>
3. Select  <strong>Cluster Name</strong> to view the Cluster configuration running in your environment.
4. Explore the networking configuration in the  <strong>VPC network</strong> section of the Google Cloud console. Understand how the VM instance(s) are connected to the network and firewall rules in place
5. Navigate to the <strong>Cloud Build</strong> section and click on <strong>triggers</strong>.
6. Select the region <strong>us-central1</strong> to your triggers.
7. You can see the <strong>cloud-deploy-trigger, application-deployment-trigger, and canary-deployment-trigger</strong> along with other build pipeline. 
8. Search for <strong>Cloud deploy</strong> in search bar and click on <strong>Cloud deploy</strong>. You can see the delivery pipeline, which is used to deploy the application on the GKE cluster.

**Triggers: cloud-deploy-trigger**

   * The cloud-deploy-trigger is activated, likely immediately following the successful creation of the triggers by the terraform-cloud-build-trigger.
   * Cloud Build begins executing the steps defined in `qwiklabs-tf/terraform/env/cloud-deploy/cloud-deploy.yaml.`
   * The cloud-deploy.yaml file contains Terraform code that defines the creation of Deploy delivery pipeline and GKE target.
   * This setup configures Cloud Deploy to manage the deployment of the Online Boutique application to the specified GKE cluster.
   *  All the of terraform scripts that are used to create Deploy delivery pipeline and GKE target are present on this directory `qwiklabs-tf/terraform/env/cloud-build/` , In those scripts you can view in detail how the triggers are created using terraform.
   * Immediately after the Cloud Deploy infrastructure is created, the application-deployment-trigger is automatically activated. This initiates the actual deployment of the Online Boutique application.


**Triggers: application-deployment-trigger**
   * Once the application-deployment-trigger is activated, Cloud Build begins executing the steps defined in `online-boutique-repository/cloud-build.yaml`.
   * In the cloud-build.yaml of cloud-deploy-trigger a step is defined to create images, push them to Artifact Registry.
   * Once the Images are pushed into Artifact Registry, In Next step deployments start using skaffold file, which is present in same directory.
   * Below is the code snippet used to pushed images on AR and trigger of the Cloud deploy pipeline by using skaffold.yaml.
   ```
   - id: 'Deploy application to cluster'
   name: 'gcr.io/k8s-skaffold/skaffold:v2.13.2'
   entrypoint: 'bash'
   args:
   - '-c'
   - |
      cd online-boutique-repository
      ls
      services=("emailservice" "productcatalogservice" "recommendationservice" "shippingservice" "checkoutservice" "paymentservice" "currencyservice" "frontend" "adservice" "cartservice" "loadgenerator")

      # Iterate over the services to build and push each image
      for service in "${services[@]}"
         do
         echo "Building and pushing $service..."
         # replacing project id in the image.
         sed -e "s/PROJECT_ID/${_PROJECT_ID}/g" kubernetes-manifests/$service.yaml.tpl > kubernetes-manifests/$service.yaml;
         # removing .tpl file
         rm kubernetes-manifests/$service.yaml.tpl
         done
      gcloud deploy releases create "labs-release-$(date '+%Y%m%d%H%M%S')" --project=${_PROJECT_ID} --region=us-central1 --skaffold-file=skaffold.yaml --delivery-pipeline=qwiklabs-delivery-pipeline --skaffold-version=skaffold_preview
   ```
   * In skaffold.yaml, Deployment is done by [skaffold](https://cloud.google.com/deploy/docs/using-skaffold) using kubectl commands, all the [manifest files](https://cloud.google.com/deploy/docs/using-skaffold/managing-manifests) path is mentioned in below snippet.
   ```
   manifests:
   kustomize:
      paths:
      - kubernetes-manifests
   deploy:
   kubectl: {}
  ```
  * This trigger will complete application deployment of Version 1.


## Task 4. Update Secret in Secret Manager ![usr1](img/qwiklabs-icons1-small.drawio.png)
1. Search for <strong>Secret Manager</strong> on the GCP Console <strong>Search Box</strong>.
2. Navigate to the <strong>Secrets</strong>, you can find a secret named as <strong>"secret"</strong> already present in secret manager.
3. Select the <strong>Secret</strong>, and Add a new version by Clicking on <strong>"New Version"</strong> on the secret.

<img src="img/secret-version.png" alt="version" width="400.00">

4. Enter <i><strong>"version-2"</strong></i> on the new version and select the <strong>"Disable all past versions"</strong> checkbox. Click on <strong>ADD NEW VERSION</strong>

<img src="img/value.png" alt="version" width="300.00">

## Task 5. Configure environment variable in canary deployment Cloud build Trigger ![usr1](img/qwiklabs-icons1-small.drawio.png)
1. Navigate to the <strong>Cloud Build</strong> section and click on <strong>triggers</strong>.
2. Select the region <strong>"us-central1"</strong> for your triggers.
3. Click on the trigger name which is created initially step<strong>(canary-deployment-trigger)</strong>.
4. In Event Choose <strong>Push to branch</strong>.
5. Under <strong>Advance section</strong> and <strong>Substitution variables</strong>, click on add variables and fill the below details:

   | <strong>Variables</strong>| <strong>Value</strong>| 
   |---------------------------| --------------------- | 
   | _PROJECT_ID               | your-project-id       | 
   | _V1_TRAFFIC               | number                | 
   | _V2_TRAFFIC               | number                | 

   <ql-infobox>
      <strong>_V2_TRAFFIC </strong> : 25 <br> 
      <strong>_V1_TRAFFIC </strong> : 75 <br> 
      <strong>Your-project-id</strong> : Project ID present in lab Instruction <br> 
   </ql-infobox>

6. Click on <strong>Save</strong>.

**Trigger: canary-deployment-trigger**
   * This Trigger gets automatically triggered when codes are pushed to the repository.
   * First step of this trigger is to build and push new productcatalogservice to Artifact Registry.
   * Once images are pushed to Artifact Registry we will add a label version=v1 to the productcatalog deployment.
   * The Service Mesh which was enabled in previous steps will be used here to split traffic.
   * This trigger uses kubectl commands to apply manifest file required to split traffic, you can view all the manifest files under `online-boutique-repository/traffic-split-manifest/`.
   * `destination-vs-v1.yaml` will deploy your [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/) and [DestinationRule](https://istio.io/latest/docs/reference/config/networking/destination-rule/) for v1 of productcatalog.
   * `productcatalog-v2.yaml.tpl` has productcatalogservice-v2 which introduces a new service with 3-second latency into requests on the EXTRA_LATENCY field.
   * `destination-v1-v2.yaml` uses DestinationRule to specify the subsets of a service. here we have a subset for v1 productcatalogservice and then a separate subset for v2 of productcatalogservice.
   * `vs-split-traffic.yaml.tpl` uses VirtualService to define a small percentage of the traffic to direct to v2 of the productcatalogservice.
   * This completes the implemention a canary deployment strategy using a service mesh.

## Task 5. Make Changes in Application Factory ![usr2](img/qwiklabs-icons-2-small.drawio.png)

   <ql-infobox>
   <strong>Note:</strong> Before performing this step it is recommended to switch the Chrome profile of a Developer
   </ql-infobox>
<br>
In this step you will modify the application code and push to repository.

1. On the Top right corner you can see a icon of <strong>Cloud shell</strong>, Click on the Cloud shell icon, GCP provision a <strong>Cloud shell instance</strong> which would have all the necessary packages pre installed.

2. Once you connect to <strong>Cloud shell instance</strong>, you can see a Terminal, on top of terminal you can find a <strong>Open Editor</strong> button, press that and you can see a <strong>Cloud shell editor</strong> similar to VS Code.

3. Once you are connected you can clone the  <strong>Application Repository </strong> using git clone command, you can copy this command and paste it on the  <strong>cloud shell terminal </strong>.  
<strong><i>gcloud source repos clone online-boutique-repository --project=(Enter your project ID)</i> </strong>.

4. Checkout in main branch and left panel you able to view all the codes on the editor, you need to copy the whole folder <i>src/productcatalogservice</i> and copy on the same directory, and rename the folder as <i>productcatalogservice-v2</i>.

5. Goto this file  <strong><i>src/productcatalogservice-v2/products.json</i></strong> and edit the name of "Tank Top for Women","Watch for men" add descrition  <strong>"SALE!!! SALE!!! SALE!!!." </strong> .

6. Once edit is done you can do the CSR.
   <strong>"git add ."
   "git commit -m "code added" "
   "git push origin main" </strong>

7. Automatic Canary Deployment cloud build triggered and Deploy the latest changes. Navigate to the <strong>Cloud build</strong> and validate the cloud build logs under <strong>history</strong> tab. 

## Task 6. Evaluate Canary deployment ![usr1](img/qwiklabs-icons1-small.drawio.png)

 1. **Navigate to Kubernetes Engine:** Click on the <strong>Navigation menu (☰)</strong> on the left side of the console to locate <strong>"GKE Cluster"</strong>. It's under <strong>"Kubernetes Engine"</strong>.
 2. **Select Your Cluster:** Locate and select the same cluster associated with this application.
 <br><br>
 <img src="img/service-mesh.png" alt="service mesh" width="400.00">
 <br><br>

 3. You can find Service mesh Dashboard endpoint on the bottom of Cluster Details.

 4. You will be able to see visuals of Service Mesh topology of Current application on this section, you can also view the traffic splitting.

 <img src="img/topology.png" alt="topology" width="600.00">


## Task 7. Monitor Traffic change between the two deployments ![usr2](img/qwiklabs-icons-2-small.drawio.png)

1. **Navigate to Kubernetes Engine:** Click on the <strong>Navigation menu (☰)</strong> on the left side of the console to locate <strong>"Kubernetes Engine"</strong>. It's under <strong>"Clusters"</strong>.

2. **Select Your Cluster:** click on the GKE cluster name associated with this application.

3. On the Left Pane, Click on the <strong>Gateway,Services & Ingress</strong> Once you clicked you can see <strong>Gateway, Routes, Services and Ingress</strong>.

4. You need to click on <strong>"Services"</strong> and details of all the running services for the application is available with IP's, you can find the <strong>frontend</strong> service  Ip address here.

   <img src="img/GKE-Servicess.png" alt="gce-dropdown" width="400.00">
   <br></br>

5. Paste the Ip address on your Chrome Browser and you can verify the traffic split.

   OR 

   You can use the below command under service project cloud shell.
   ```      
   export SERVICE_PROJECT_ID=service-project-id
   gcloud container clusters get-credentials qwiklabs-developer-1-cluster --region us-central1 --project $SERVICE_PROJECT_ID
   kubectl get services frontend-external
   ```

6. When you refresh the page, as we have set a <strong>75/25 Split</strong>, you can see <strong>v2 deployment</strong> after every <strong>4th</strong> page refresh.

7. For the First Three Refresh, you will see the below webpage image.

   <img src="img/Service-v1.png" alt="V1 Service" width="600.00">

8. On The Fourth Refresh you will see the change in the Description as shown below image.

   <img src="img/Service-v2.png" alt="V1 Service" width="600.00">


## Congratulations!
You have completed the lab and have a solid understanding of Terraform, GKE and GitOps in Google Cloud.

## Key Takeaways
1. GitOps Workflow: Streamline infrastructure updates and rollbacks using Git as the single source of truth.
2. Terraform + Cloud Build: Automate environment provisioning and improve reliability with CI/CD pipelines.
3. Cloud Deploy + Istio: Automate canary deployment by spliting the service in two.
3. Collaboration and Auditing: Enable team collaboration with version control and maintain a detailed history of infrastructure changes.
4. Scalability and Repeatability: Design modular, reusable Terraform configurations to scale your platform efficiently.

## Lab Resources
* Terraform Code: Pre-configured with reusable modules and variables.
* Google Cloud Build Pipeline: Pre-defined CI pipeline template.
* Google Cloud Deploy: Pre-defined CD pipeline template.
* Documentation: Detailed instructions and references for Terraform, Cloud Build, and GitOps practices.

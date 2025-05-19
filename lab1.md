# Building a Platform with IaC + GitOps

## Overview
Welcome to this lab, where you'll learn to build and manage enterprise-grade cloud environments using Infrastructure as Code (IaC) and GitOps methodology. Through hands-on experience with Terraform and Google Cloud Build, you'll automate infrastructure deployment and management on Google Cloud. This lab is designed for Infrastructure Admins and Platform Engineers, this lab provides essential skills to enhance infrastructure reliability, scalability, and maintainability while following modern DevOps best practices. As enterprise infrastructures grow in complexity, Platform Engineering and GitOps are becoming key approaches to streamlining the development and operations lifecycle, resulting in improved developer experience and feature velocity.

**Platform Engineering**

Platform engineering streamlines software delivery by building and managing self-service infrastructure platforms by providing users of the platform "golden paths". Golden paths abstract away infrastructure complexities and processes, empowering developers with the tools and flexibility they need to concentrate on feature development. For example, in this lab the platform provides a golden path allowing the developer to build and deploy application changes without needing to interact with the underlying infrastructure.

**GitOps**

GitOps manages infrastructure and applications declaratively, using Git as the central source of truth. It leverages Git's features like pull requests and workflows to automate and control deployments.

## Lab
This unique lab requires students to assume the roles (personas)  of both Platform Engineer and Developer, working across multiple GCP projects.  Three distinct GCP Projects are pre-created at the start of the Lab (detailed in the table below)

| <strong>Project Type</strong>|    <strong>Project Name</strong> |     <strong>Purpose</strong> |
|:--------:| :-----------------: | :--------- |
| Seed 🌱 | qwiklabs-gcp**. | This project grants exclusive access to the Platform Engineer for the administration of IaC Pipelines. This project has the IaC pipelines to manage the non-prod environment, prod environment and spin up new environments. This project will be used by the Platform Engineer to manage the infrastructure in the Non-Prod  and Prod  projects |
| Non-Prod ⚙️ | qwiklabs-gcp**. | This project serves as a development environment for the purpose of testing and verifying application modifications prior to their deployment into the production environment. Please note:  Once you start the Lab all required resources for the Non-Prod environment are predeployed including Applications |
| Prod 🏭 | qwiklabs-gcp** | This project, similar to the Non-Prod project, provides resources for developers to deploy their applications. However, this project is specifically used for handling production traffic. |

<ql-infobox>
<strong>Note:</strong> ** The project names are randomly generated, please check them once you start the lab.
</ql-infobox>

## Personas

| <strong>Role/Persona</strong>| <strong>Tasks</strong> |
|--------| ----------------- |
| Platform Engineer (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png) | The platform engineer will leverage existing IaC automation scripts to deploy new production environments, implementing automated CI/CD pipelines using Cloud Build triggers to adhere to GitOps principles and simplify the production platform setup. |
| Developer ![usr2](img/qwiklabs-icons-2-small.drawio.png) | The Developer will confirm the deployed web application's functionality via a web browser and analyze its performance metrics using the monitoring dashboard. |

## Objectives

As a Platform Engineer (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png), your task is to evaluate the existing infrastructure in the Non-Prod Project ⚙️ and then use the Seed project's 🌱 automated pipelines to provision similar resources in the Prod environment 🏭. 

As a Developer ![usr2](img/qwiklabs-icons-2-small.drawio.png), your responsibility is to evaluate the deployed application's performance and functionality in both the Production 🏭 and Non-Production environments ⚙️.

### Architecture
![Architecture](img/qwiklabs-icons-Lab1.drawio.png)

### Before you click the Start Lab button

   This is an advanced lab, it is designed for Platform Engineers / Infrastructure Engineers / Site Reliability Engineers (SREs), this lab assumes you have:

   1. Basic knowledge of Terraform syntax and configuration files.
   2. Familiarity with CI/CD concepts.
   3. An understanding of Google Cloud Platform (GCP) fundamentals, including IAM, networking, and Compute Engine.

## Task 1. Prepare your environment (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task, you create separate Chrome profiles for each persona and start the lab.

This lab includes navigating between multiple projects and multiple user personas. To simplify navigating between projects setup environment variables

1. Sign in using the Platform Engineer username {{{ user_1.username }}} and password {{{ user_1.password }}}

2. On the Google Cloud console title bar, click **Activate Cloud shell**
   ![Activate Cloud Shell](img/cloudshell.png)

## Task 2. Configure your project in Cloud Shell (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task, you set environment variables and clone the sample repositories.

1. In Cloudshell, set environment variables

   <ql-code-block language="bash" templated>
   export SEED_PROJECT_ID={{{project_0.project_id|'[SEED_PROJECT_ID]'}}}
   export REGION={{{project_0.default_region|'[REGION]'}}}
   export USER_EMAIL={{{user_1.username|['PLATFORM_ENG_PERSONA_EMAIL]'}}}
   export USER={{{user_1.local_username|['PLATFORM_ENG_PERSONA_USERNAME]'}}}
   export DEST_REPO=app-repo
   </ql-code-block>

2. Clone the infrastructure and application repos:

   ```sh
   gcloud source repos clone building-platform-with-iac --project=qwiklabs-resources
   gcloud source repos create "$DEST_REPO" --project="$SEED_PROJECT_ID"
   cd building-platform-with-iac && git checkout main && cd ../
   gcloud source repos clone "$DEST_REPO" --project="$SEED_PROJECT_ID" && cd $DEST_REPO
   ```

3. Copy the starter files and push your initial commit:

   ```sh
    git config user.email "$USER_EMAIL"
    git config user.name "$USER"
    ../building-platform-with-iac/repo-setup.sh
   ```

## Task 3. Grant Access and Configure Cloud build Trigger for Non-Prod Environment
Once you login as a Platform Engineer (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png), You will be able to choose between 4 projects. You have to identify the 🌱 seed project, by using the 🌱Seed project id mentioned in the left sidebar of the lab

   1. Now that you have logged as a <strong>Platform Engineer</strong> in and selected the seed project.
   2. Go to the IAM section.
   3. Find a service account named <strong>seed-sa</strong> and copy the email of the service account we will use it later.
      * Note that This service account will have Storage Admin and Logging Admin access.

      <img src="img/IAM2.png" alt="IAM" width="600.00">

   4. Go to the <strong>Cloud Storage</strong> section, you will a bucket there, copy the name of the bucket we will use this later while deploying resources.
   5. Now search for <strong>Cloud Build</strong> in the <strong>search bar</strong> on the top. 
   
   <img src="img/CBT1.png" alt="cloudbuild" width="500.00">
   
   
   6. You are going to create 2 Cloud Build triggers; one trigger for terraform plan and another for terraform apply. Create them using the following steps and details:
         1. Trigger Name (name: terraform-plan-trigger)
         2. Chose region as <strong>us-central1</strong>
         3. Add a description (example: This trigger will run terraform plan command on the terraform code and give the output in the logs)
         4. In Event choose <strong>Manual invocation</strong>
         5. In Source choose the <strong>repo name</strong> provided in the lab and choose <strong>1st Gen</strong> and in Branch filter put <strong>non-prod</strong>
         6. For Configuration choose Cloud Build configuration file and in the location put the path the file <strong>```qwiklabs-tf/terraform/env/cloudbuild-plan.yaml```</strong>
         7. Under Advanced -> Substitution Variables add the following and substituting in values as appropiate:

       | <strong>Variables</strong> | <strong>Value</strong>| |
       |:--------:|:-----------------:|:---------:| 
       | _ENV    | add-your-environment | |
       | _TF_BUCKET_NAME | Bucket-name | |   
       | _PROJECT_ID | your-project-id | |

         9. Choose the <strong>Service Account</strong> we saved in the steps above
         10. Click on <strong>Create</strong>.
   
      <img src="img/CBT2.png" alt="cloudbuild" width="400.00">

   * <strong>Repeat the above steps again to create another <strong>(Name: terraform-apply-trigger)</strong> cloud build trigger to create resources. Under <strong>Cloud build configuration file location</strong> put the path the file <strong>```qwiklabs-tf/terraform/env/cloudbuild-apply.yaml```</strong> </strong>

## Task 4. Access and Assess Non Prod Project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   In this step you are going to access the Non-Prod Project as a <strong>Platform Engineer</strong> and then assess the resources that were pre-deployed.
   1. In the <strong>Google Cloud Console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Non-Prod Project</strong>. You'll need the project ID, which was provided in the labs details when you started the lab.
   2. Use the <strong>navigation</strong> on the left side of the console to locate <strong>"Compute Engine"</strong>, under the <strong>"Compute"</strong>
   3. Select <strong>"VM Instances"</strong>. Now you can view the virtual machines running in your Non-Prod environment. Identify the relevant VM instance(s) that are part of the application you are assessing.
   4. Connect to the VM instance(s) using SSH (for Linux-based instances). (Optional)
   5. Explore the networking configuration in the <strong>"VPC Network"</strong> section of the Google Cloud Console. Understand how the VM instance(s) are connected to the network and firewall rules in place.

## Task 5. Grant Access and Configure Cloud build Trigger for Non-Prod Environment (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step as a Platform Engineer, you are going to deploy the resources in the Non-Prod Environment.

1. In the <strong>Google CloudShell</strong> grant access to the </strong>Seed Project service account</strong>.

   ```sh
   SERVICE_ACCOUNT="seed-sa@$SEED_PROJECT_ID.iam.gserviceaccount.com"
   ROLES=(
   "roles/compute.admin"
   "roles/logging.admin"
   "roles/logging.viewer"
   "roles/iam.roleAdmin"
   "roles/iam.serviceAccountAdmin"
   "roles/iam.serviceAccountUser"
   "roles/serviceusage.serviceUsageAdmin"
   "roles/storage.admin"
   "roles/cloudbuild.builds.editor"
   )

   for ROLE in "${ROLES[@]}"; do
   echo "Adding role ${ROLE} to ${SERVICE_ACCOUNT} on project ${PROJECT_ID}"
   gcloud projects add-iam-policy-binding "${PROJECT_ID}" \
      --member="serviceAccount:${SERVICE_ACCOUNT}" \
      --role="${ROLE}" \
      --condition=None # Optional: explicitly set no condition
   done

   echo "All roles have been processed."
   ```

   6. Click on <strong>Save</strong>.
   7. Navigate to the <strong>Cloud Build</strong> section and click on <strong>Triggers</strong> in the <strong>Seed Project</strong>.
   8. Select the region <strong>"us-central1"</strong> to the your triggers.
   9. Click on the trigger name which is created intially <strong>(terraform-plan-trigger)</strong>.
   10. Click on <strong>Edit</strong>.
   11. Under Advanced -> Substitution Variables add the following and substituting in values as appropiate:

   | <strong>Variables</strong>| <strong>Value</strong>| |
   |--------| ----------------- | --------- |
   | _ENV | add-your-environment | |
   | _PROJECT_ID | your-project-id | |
   | _TF_BUCKET_NAME | Bucket-name | |
   | _BRANCH_NAME | branch-name | |

   <ql-infobox>
       add-your-environment : <strong>non-prod</strong> <br>
       Bucket-name : <strong>Present in the seed project</strong> <br>
       your-project-id : <strong>Non-Prod Project ID present in lab Instruction</strong> <br>
       branch-name : <strong>non-prod</strong> <br>
   </ql-infobox>

   12. Click on <strong>Save</strong>.

   * <strong>Repeat the above steps 7 to 12 again to update the other<strong>(terraform apply trigger)</strong> Cloud Build trigger. </strong>

## Task 6. Deploy Resources in Non-Prod Environment (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step you are going to deploy the resources in the Non-Prod Environment as a <strong>Platform Engineer</strong>.
   1. Navigate to the <strong>Cloud Build</strong> section and click on <strong>triggers</strong> in the <strong>Seed Project</strong>.
   2. Select the region <strong>"us-central1"</strong> to the your triggers.
   3. You find the <strong>terraform-plan-trigger</strong>, Click on <strong>Run</strong>. 

   * <strong> Above step will create you Non-Prod Environment via 2 Could build trigger one trigger plan your Resource and next trigger will create the resources. You can find the log in <strong>history</strong> Tab </strong>  

## Task 7. Check Deployed Resource in Non-Prod Project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step you will check the deployed resources in the Non-Prod Project.
   1. Login as a <strong>Platform Engineer</strong>.
   2. In the <strong>Google Cloud console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Non-Prod Project</strong>. You'll need the project ID, which should be provided in the lab instructions.
   3. Use the navigation menu on the left side of the console to locate <strong>"Compute Engine"</strong> It's under <strong>"Compute"</strong>
   4. Select "VM Instances" to view the virtual machines running in your Prod environment.
   5. Explore the networking configuration in the "VPC network" section of the Google Cloud console. Understand how the VM instance(s) are connected to the network and firewall rules in place

## Task 8. Create branch in the repository to deploy resource on the prod project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   
In this step you are going to create a prod branch and Deploy the resources in the Production Environment as a <strong>Platform Engineer</strong>.
   1. In the <strong>Google Cloud console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Seed Project</strong>. You'll need the project ID, which should be provided in the <strong>lab instructions</strong>.
   2. Click on <strong>activate clould shell</strong> in the Google Cloud console.
   3. Clone the GIT repository. <strong>gcloud source repos clone lab1-testing-tf-code --project=qwiklabs-gcp-01-2fc5a420bf16</strong>.
   4. Navigate to the cloned repository. cd lab1-testing-tf-code
   5. Checkout to a new branch using git checkout -b <strong>prod</strong>.
   6. Commit the changes in the git repository.
   7. Push the changes in the git repository.

## Task 9. Grant Access and Configure Cloud build Trigger for Prod Environment (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step you are going to deploy the resources in the prod Environment as a <strong>Platform Engineer</strong>.
   1. In the <strong>Google Cloud Console</strong> navigate to the IAM section of the </strong>Seed Project</strong> and Copy the Service Account which is already created for Cloud Build <strong>"seed-sa"</strong>.
   2. Switch the project to the Prod project and navigate to the IAM section of the <strong>Prod Project</strong>.
   3. Click on Grant access and add the service account in the <strong>new principal</strong> box.
   4. Select the roles mentioned below:
   <ql-infobox>
      roles/compute.admin.
      roles/logging.admin.
      roles/logging.viewer.
      roles/iam.roleAdmin.
      roles/iam.serviceAccountAdmin.
      roles/iam.serviceAccountUser.
      roles/serviceusage.serviceUsageAdmin.
      roles/storage.admin.
      roles/cloudbuild.builds.editor.
    </ql-infobox>
   5. Click on <strong>Save</strong>. 
   6. Navigate to the <strong>Cloud Build</strong> section and click on <strong>triggers</strong> in the <strong>Seed Project</strong>.
   7. Select the region <strong>"us-central1"</strong> to the your triggers.
   8. Click on the trigger name which is created intially <strong>(terraform-plan-trigger)</strong>
   9. Click on <strong>Edit</strong>.
   10. In source Choose the <strong>repo name</strong> provided in the lab and choose <strong>1st Gen</strong> and in Branch filter put <strong>prod</strong>
   11. Under <strong>Advance section</strong> and <strong>Substitution variables</strong>, click on add variables and fill the below details:

   | <strong>Variables</strong>| <strong>Value</strong>| |
   |--------| ----------------- | --------- |
   | _ENV | add-your-environment | |
   | _PROJECT_ID | your-project-id | |
   | _TF_BUCKET_NAME | Bucket-name | |
   | _BRANCH_NAME | branch-name | |

   <ql-infobox>
       add-your-environment : <strong>prod</strong> <br> 
       Bucket-name : <strong>Present in the seed project</strong> <br> 
       your-project-id : <strong>Prod Project ID present in lab Instruction</strong> <br> 
       branch-name : <strong>prod</strong> <br>
   </ql-infobox>

   12. Click on <strong>Save</strong>.

   * <strong>Repeat the above steps 6 to 12 again to update the another<strong>(terraform apply trigger)</strong> cloud build trigger. </strong>

## Task 10. Deploy Resources in Prod Environment (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   In this step you are going to deploy the resources in the Prod Environment as a <strong>Platform Engineer</strong>.
   1. Navigate to the <strong>Cloud Build</strong> section and click on <strong>triggers</strong> in the <strong>Seed Project</strong>.
   2. Select the region <strong>"us-central1"</strong> to the your triggers.
   3. You find the <strong>terraform-plan-trigger</strong>, Click on <strong>Run</strong>. 

   * <strong> Above step will create  you Prod Environment via 2 Could build trigger one trigger plan your resource and next trigger will create the resources. You can find the log in <strong>history</strong> Tab </strong>  

## Task 11. Check Deployed Resource in Prod Project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   In this step you will check the deployed resources in the Non-Prod Project.
   1. Login as a <strong>Platform Engineer</strong>.
   2. In the <strong>Google Cloud console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Prod Project</strong>. You'll need the project ID, which should be provided in the lab instructions.
   3. Use the navigation menu on the left side of the console to locate <strong>"Compute Engine"</strong> It's under <strong>"Compute"</strong>
   4. Select "VM Instances" to view the virtual machines running in your Prod environment.
   5. Explore the networking configuration in the "VPC network" section of the Google Cloud console. Understand how the VM instance(s) are connected to the network and firewall rules in place

## Task 12. Grant access to the Developer User Non-Prod Project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   In this step we will grant the developer user access to the Non-Prod Project using the <strong>Platform Engineer</strong> account.
   1. In the <strong>Google Cloud console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Non-Prod Project</strong>. You'll need the project ID, which should be provided in the lab instructions.
   2. Navigate to the IAM section of the Prod Project.
   
      <img src="img/IAM1.png" alt="IAM" width="300.00">

   3. Click on <strong>Grant access</strong> and add the <strong>dev-user</strong> account in the <strong>new principal</strong> box.
   4. Select the roles mentioned below:
      1. roles/viewer
      2. roles/iap.tunnelResourceAccessor
      3. roles/iam.serviceAccountUser
   5. Click on <strong>Save</strong>

      <img src="img/iam3.png" alt="IAM" width="500.00">

## Task 13. Grant access to the Developer User Prod Project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)
   In this step we will grant the developer user access to the Prod Project using the <strong>Platform Engineer</strong> account.
   1. In the <strong>Google Cloud console</strong>, use the project selector dropdown at the top of the screen to switch to your <strong>Prod Project</strong>. You'll need the project ID, which should be provided in the lab instructions.
   2. Navigate to the IAM section of the Prod Project.
   
      <img src="img/IAM1.png" alt="IAM" width="300.00">

   3. Click on <strong>Grant access</strong> and add the <strong>dev-user</strong> account in the <strong>new principal</strong> box.
   4. Select the roles mentioned below:
      1. roles/viewer
      2. roles/iap.tunnelResourceAccessor
      3. roles/iam.serviceAccountUser
   5. Click on <strong>Save</strong>

      <img src="img/iam3.png" alt="IAM" width="500.00">

## Task 14. Check app deployed on Non Prod ![usr2](img/qwiklabs-icons-2-small.drawio.png)

   <ql-infobox>
   <strong>Note:</strong> Before performing this step it is recommeded to switch the Chrome profile of a Developer
   </ql-infobox>

   In this step we will access the application that is deployed in the Non Prod Project's VM Instance.
   1. In the Google Cloud console, use the project selector dropdown at the top of the screen to switch to your Non-Prod Project. You'll need the project ID, which should be provided in the lab instructions.
   2. Use the navigation menu on the left side of the console to locate <strong>"Compute Engine"</strong> It's under <strong>"Compute"</strong>
   3. Copy the instance <strong>public ip</strong> and paste it on the new tab.
   4. Use the navigation menu on the left side of the console to locate <strong>"Dashboard"</strong>, it's under<strong>"Observability Monitoring"</strong>. <br>
   If you use the search bar to find this page, then select the result whose subheading is Monitoring.<br>
   <img src="img/mon_dashboard.png" alt="place holder for monitoring dashboard" width="400px"> <br>
   There will be a pre-created Dashboard present for the application logs on the VM Instance or in the search box you may search for Dashboard

## Task 15. Check app deployed on Prod ![usr2](img/qwiklabs-icons-2-small.drawio.png)
   In this step we will access the application that is deployed in the Prod Project's VM Instance.
   1. In the Google Cloud console, use the project selector dropdown at the top of the screen to switch to your Prod Project. You'll need the project ID, which should be provided in the lab instructions.
   2. Use the navigation menu on the left side of the console to locate <strong>"Compute Engine"</strong> It's under <strong>"Compute"</strong>
   3. Copy the instance <strong>public ip</strong> and paste it on the new tab.
   4. Use the navigation menu on the left side of the console to locate <strong>"Dashboard"</strong>, it's under<strong>"Observability Monitoring"</strong>. <br>
   If you use the search bar to find this page, then select the result whose subheading is Monitoring.<br>
   <img src="img/mon_dashboard.png" alt="place holder for monitoring dashboard" width="400px"> <br>
   There will be a pre-created Dashboard present for the application logs on the VM Instance or in the search box you may search for Dashboard


   ## Congratulations!
   You have completed the lab and have a solid understanding of Terraform and GitOps in Google Cloud.
   ## Key Takeaways
   1. GitOps Workflow: Streamline infrastructure updates and rollbacks using Git as the single source of truth.
   2. Terraform + Cloud Build: Automate environment provisioning and improve reliability with CI/CD pipelines.
   3. Collaboration and Auditing: Enable team collaboration with version control and maintain a detailed history of infrastructure changes.
   4. Scalability and Repeatability: Design modular, reusable Terraform configurations to scale your platform efficiently.
   ## Lab Resources
   * Terraform Code: Pre-configured with reusable modules and variables.
   * Google Cloud Build Pipeline: Pre-defined CI/CD pipeline template.
   * Documentation: Detailed instructions and references for Terraform, Cloud Build, and GitOps practices.
   ## Next Steps
   After completing this lab, you are encouraged to:
   * Extend the provided Terraform modules to include additional services like databases or serverless resources.
   * Integrate with third-party DevOps tools for enhanced observability and security checks.
   * Experiment with advanced GitOps workflows, such as canary deployments or blue/green strategies.

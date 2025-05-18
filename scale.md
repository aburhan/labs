# Lab: Scalable Runtime Management, SLI, SLOs

## Overview

Welcome to this lab, this lab guides you through setting up a scalable runtime environment using [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine?hl=en#manage-multi-cluster-infrastructure), GKE Enterprise Fleet, and Cloud Service Mesh (CSM). You will deploy a microservices application (Online Boutique) across multiple GKE clusters, manage them using Fleet, and secure and observe inter-service communication with CSM.

## Lab

In this lab, you will take on two personas: <strong>Platform Engineer</strong> and <strong>Developer</strong>.
This project represents the live, production environment. Platform Engineers will deploy and manage resources within this project, while Developers will have access to monitor and verify the deployed applications and infrastructure.
This project will be used by both Platform Engineer![usr1](img/qwiklabs-icons1-small.drawio.png) who will deploy resources, and Developer ![usr2](img/qwiklabs-icons2-small.drawio.png) will access the resources.

| <strong>Resources</strong>                                                                                | <strong>Platform Engineer Tasks</strong> ![usr1](img/qwiklabs-icons1-small.drawio.png)                                                                                                                                                                                                                                                                                                                                                                                                    | <strong>Developer Tasks</strong> ![usr2](img/qwiklabs-icons2-small.drawio.png)                                                                                                                                                                                                                                                                                              |
| --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Git Repository 🐱                                                                                         | **•** Create a new branch for infrastructure updates. <br> **•** Define environment variables for the deployment (e.g., image tag, replica counts). <br> **•** Modify Kubernetes deployment YAMLs to implement Canary deployment (e.g., using Deployment and Service objects). <br> **•** Commit and push changes to trigger the Cloud Build pipeline.                                                                                                                                    | **•** Validate, Read, Write and Push code changes to the repository.                                                                                                                                                                                                                                                                                                        |
| Application Code ([Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo/tree/main)) | **•** No direct modification of application code.                                                                                                                                                                                                                                                                                                                                                                                                                                         | **•** Make code changes to the [Online Boutique application](https://github.com/GoogleCloudPlatform/microservices-demo/tree/main). <br>**•** Build and test the changes locally. <br>**•** Commit and push code changes to a separate application repository (this may trigger a different build/deploy pipeline for staging).<br>**•** Observe the impact of code changes. |
| Cloud Build                                                                                               | **•** Configure Cloud Build triggers to automatically build and deploy application updates to the GKE cluster. <br>**•** Utilize environment variables within Cloud Build to control deployment parameters.                                                                                                                                                                                                                                                                               | No direct interaction with Cloud Build.                                                                                                                                                                                                                                                                                                                                     |
| GKE Cluster                                                                                               | **•** Deploy the updated application to the GKE cluster using a Canary deployment strategy.<br>**•** Monitor the Canary deployment rollout and application health using Google Cloud Console or kubectl.<br>**•** Observe traffic distribution between the two versions of the application.                                                                                                                                                                                               | No direct access to the GKE cluster.                                                                                                                                                                                                                                                                                                                                        |
| Service Level Indicators                                                                                  | **•** ([SLIs](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/api/identifying-custom-sli)) are quantitative measures of a service's performance.<br>**•** (SLIs) are quantitative measures of a service's performance.<br>**•** Essentially, SLIs tell us how well our service is performing.                                                                                                                                                                          | No direct access to the SLIs.                                                                                                                                                                                                                                                                                                                                               |
| Service Level Objectives                                                                                  | **•** [SLO](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/ui/create-slo) are target values or ranges for those SLIs.<br>**•** They define the desired level of reliability and performance for our application.<br>**•** For example, we might set an SLO of 99.9% availability, meaning we aim for our service to be operational 99.9% of the time. SLOs provide a clear benchmark for evaluating our service's health and ensuring it meets our reliability goals. | No direct access to the SLOs.                                                                                                                                                                                                                                                                                                                                               |

The lab is divided into sections catering to the Platform Engineer ![usr1](img/qwiklabs-icons1-small.drawio.png) and Developer ![usr2](img/qwiklabs-icons2-small.drawio.png) roles. <br>
The lab emphasizes the use of Cloud Build for CI/CD and provides hands-on experience with service monitoring and SLO creation. Each role has specific tasks and objectives, working together to achieve the overall lab goals. <br>
The lab utilizes a multi-project setup with a host project and service projects. The Online Boutique application serves as the test application for deployment and observation.

## Target Audience

This lab is intended for:

- ![usr1](img/qwiklabs-icons1-small.drawio.png) Platform Engineers: The Platform Engineer will set up the infrastructure, including GKE clusters, Fleet configuration, team management, and service mesh.
- ![usr2](img/qwiklabs-icons2-small.drawio.png) Developers: The Developer will then focus on application development and deployment within this managed environment, making code changes, pushing updates, and monitoring their specific services.

## Before starting this lab, ensure you have

- Basic knowledge of Terraform syntax and configuration files.
- Familiarity with CI/CD concepts.
- An understanding of Google Cloud Platform (GCP) fundamentals, including IAM and networking
- A working knowledge of the Google Kubernetes Engine (GKE) is beneficial

## Lab Objectives for Developer ![usr2](img/qwiklabs-icons2-small.drawio.png)

- Login to the assigned Developer project.
- Modify the product catalog microservices.
- Push code changes to the designated repository.
- Verify service updates in the assigned namespace.

## Lab Objectives for Platform Engineer. ![usr1](img/qwiklabs-icons1-small.drawio.png)

- Interact with IAM to grant required access in the GCP projects
- Register GKE clusters to GKE Enterprise Fleet.
- Enable Fleet features, specifically Service Mesh.
- Create teams within GKE Enterprise (Dev Team 1 and Dev Team 2).
- Grant users access to specific teams.
- Create workload-specific dashboards for Dev Team 1 and Dev Team 2
- Configure Service Mesh via Fleet.
- Deploy the Online Boutique application using Cloud Build triggers.
- Use Service Monitoring to create an SLO and associate an alert.
- Verify inter-service communication using Cloud Service Mesh.

<!-- Standard Solutions Engineering Setup and Requirements section -->

![[/fragments/startqwiklab]]
![[/fragments/gcpconsole]]
![[/fragments/cloudshell]]

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
   export HOST_PROJECT_ID={{{project_1.project_id|'[HOST_PROJECT_ID]'}}}
   export REGION={{{project_1.default_region|'[REGION]'}}}
   export USER_EMAIL={{{user_1.username|['PLATFORM_ENG_PERSONA_EMAIL]'}}}
   export USER={{{user_1.local_username|['PLATFORM_ENG_PERSONA_USERNAME]'}}}
   export DEST_REPO=app-repo
   </ql-code-block>

2. Clone the infrastructure and application repos:

   ```sh
   gcloud source repos clone scalable-runtime-management --project=qwiklabs-resources
   gcloud source repos create "$DEST_REPO" --project="$HOST_PROJECT_ID"
   cd scalable-runtime-management && git checkout main && cd ../
   gcloud source repos clone "$DEST_REPO" --project="$HOST_PROJECT_ID" && cd $DEST_REPO
   ```

3. Copy the starter files and push your initial commit:

   ```sh
    git config user.email "$USER_EMAIL"
    git config user.name "$USER"
    ../scalable-runtime-management/repo-setup.sh
   ```

   The `repo-setup.sh` script creates two branch `app` and `infra`. As the platform engineer, you will use the `infra` branch to setup the infrastructure resources.

## Task 3. Grant access to the Developer user in the service project (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task, you will grant access to the Developer to update the application in the`SERVICE PROJECT` and you will also grant access to the `HOST PROJECT` to create resources in the `SERVICE PROJECT.

1. In the `Host Project`, set environment variables

   <ql-code-block language="bash" templated>
   export SERVICE_PROJECT_ID={{{project_2.project_id|'[SERVICE PROJECT_ID]'}}}
   export DEVELOPER_EMAIL={{{user_2.username|['DEVELOPER_EMAIL]'}}}
   </ql-code-block>

2. Grant the following roles to the Developer in the service project to allow the developer to update the repository and cluster and run operations as the service account:

   - `roles/source.writer`
   - `roles/iam.serviceAccountUser`
   - `roles/container.developer`

   ```sh
   gcloud projects add-iam-policy-binding $SERVICE_PROJECT_ID --member="user:$DEVELOPER_EMAIL" --role=roles/source.writer
   gcloud projects add-iam-policy-binding $SERVICE_PROJECT_ID --member="user:$DEVELOPER_EMAIL" --role=roles/iam.serviceAccountUser
   gcloud projects add-iam-policy-binding $SERVICE_PROJECT_ID --member="user:$DEVELOPER_EMAIL" --role=roles/container.developer
   ```

3. Add `HOST PROJECT` service account to create resources in the `SERVICE PROJECT` and grant access role`roles/owner`

   ```sh
   gcloud projects add-iam-policy-binding $SERVICE_PROJECT_ID \
   --member="serviceAccount:prod-sa@$HOST_PROJECT_ID.iam.gserviceaccount.com" \
   --role=roles/owner
   ```

## Task 4. Create Terraform Cloud Build trigger (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task, you configure a Cloud Build trigger in the `Host Project` to create the infrastructure necessary to deploy the application.

1. In Cloud Shell, set your bucket variable

   ```sh
   export TF_BUCKET_NAME=tf-${HOST_PROJECT_ID}
   ```

2. Create a Cloud Build triggers:

   ```sh
   gcloud builds triggers create manual --name="cloud-build-trigger" \
   --description="Initial cloud build trigger." \
   --service-account="projects/$HOST_PROJECT_ID/serviceAccounts/prod-sa@$HOST_PROJECT_ID.iam.gserviceaccount.com" \
   --repo="https://source.developers.google.com/p/$HOST_PROJECT_ID/r/app-repo" \
   --repo-type=CLOUD_SOURCE_REPOSITORIES \
   --region=$REGION \
   --branch="infra" \
   --build-config="infra/terraform/env/cloud-build/cloud-build.yaml" \
   --substitutions=_ENV=prod,_HOST_PROJECT_ID=$HOST_PROJECT_ID,_REPO_NAME=app-repo,_SERVICE_PROJECT_ID=$SERVICE_PROJECT_ID,_TF_BUCKET_NAME=$TF_BUCKET_NAME
   ```

3. Run the below command to create the inital triggers.

   ```sh
   gcloud builds triggers run cloud-build-trigger --project=$HOST_PROJECT_ID --branch=infra --region=$REGION
   ```

   This trigger creates the following Cloud Build triggers:

   | Trigger                                  | Description                                                                    |
   | ---------------------------------------- | ------------------------------------------------------------------------------ |
   | cloud-deploy-trigger                     | Creates the releases and pipelines for continuous delivery.                    |
   | docker-build-push-trigger                | Builds & pushes the container image to the Artifact Registry                   |
   | host-application-deployment-trigger      | Deploys the application in host project's cluster qwiklabs-database-1-cluster" |
   | infra-deploy-trigger                     | Deploys the infrastructure for the application.                                |
   | productcatalog-cicd-trigger              | Run the CI/CD process for the product catalog service                          |
   | service-1-application-deployment-trigger | Deploys the application to  cluster qwiklabs-developer-1-cluster"              |
   | service-2-application-deployment-trigger | Deploys the application to  cluster qwiklabs-developer-2-cluster"              |

4. Run the `infra-deploy-trigger` to create base infrastructure

   ```sh
   gcloud builds triggers run infra-deploy-trigger --branch=infra --region=$REGION
   ```

   This Terraform script will creates resources as a base infrastructure for the `Service Project` the developer will utilize to deploy the sample application. Some shared or supporting resources will also be configured in the `Host Project` to facilitate this environment. The following key components will be provisioned:

   - Activation of essential Google Cloud services and APIs across both host and service projects.
   - A dedicated and secure Virtual Private Cloud (VPC) network within the Service Project, complete with subnets, firewall rules, and a Cloud NAT gateway.
   - Specific service accounts and custom IAM roles with appropriate permissions established in both the host and service projects.
   - A private Google Kubernetes Engine (GKE) cluster within the Service Project to host applications.
   - Artifact Registry repositories in the Host Project for storing and managing software packages.

5. Navigate to the <strong>[Cloud Build](https://cloud.google.com/build/docs/overview)</strong> using <strong>Navigate Menu</strong> .
6. Click on <strong>history</strong>.
7. You can find the <strong>cloud-build-trigger</strong> logs.

<img src="img/cloudbuildtriggerlogs.png" alt="LOGS" width="500"> <br>

<ql-infobox>
<strong>Note:</strong> This step creates the remaining [Cloud Build](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers) triggers responsible for setting up the infrastructure, build and push docker image for Online Boutique Application, and deploying it.
</ql-infobox>

## Task 5. Verify Deployed Resources (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task you will verify all resources that where deployed in the `Service Project`

- The `infra-deploy-trigger` Cloud Build trigger has successfully provisioned all necessary infrastructure resources in both the host and service projects.

1. Verify provisioned GKE clusters in the service project.

   ```sh
   gcloud container clusters list --project=${SERVICE_PROJECT_ID}
   ```

   Example output:

   ```sh
   NAME: qwiklabs-developer-1-cluster
   LOCATION: us-central1
   MASTER_VERSION: 1.32.3-gke.1785003
   MASTER_IP: 34.31.200.200
   MACHINE_TYPE: e2-standard-2
   NODE_VERSION: 1.32.3-gke.1785003
   NUM_NODES: 3
   STATUS: RUNNING

   NAME: qwiklabs-developer-2-cluster
   LOCATION: us-central1
   MASTER_VERSION: 1.32.3-gke.1785003
   MASTER_IP: 104.155.180.175
   MACHINE_TYPE: e2-standard-2
   NODE_VERSION: 1.32.3-gke.1785003
   NUM_NODES: 3
   STATUS: RUNNING
   ```

2. Verify the Cloud Deploy pipeline created by the `cloud-deploy-trigger` in the `Service Project`.

   ```sh
   gcloud deploy delivery-pipelines describe qwiklabs-delivery-service-pipeline \
   --project=$SERVICE_PROJECT_ID \
   --region=$REGION
   ```

   Example output similar to:

   ```sh
   Delivery Pipeline:
      description: main application pipeline
      ...
      serialPipeline:
         stages:
         - targetId: target-1
         - targetId: target-2
      ...
      Targets:
      - Target: projects/qwiklabs-gcp-00-bed84b592c08/locations/us-central1/targets/target-1
      - Target: projects/qwiklabs-gcp-00-bed84b592c08/locations/us-central1/targets/target-2
   ```

   The pipeline deploy the application code to the two targets `target-1` and `target-2`.

3. Verify the targets `target-1` and `target-2` point to the gke cluster verified in the previous steps

   ```sh
   gcloud deploy targets describe target-1 \
      --region=$REGION --project=$SERVICE_PROJECT_ID  \
      --format='table(Target.targetId, Target.description, Target.gke.cluster)' 

   gcloud deploy targets describe target-2 \
      --region=$REGION --project=$SERVICE_PROJECT_ID  \
      --format='table(Target.targetId, Target.description, Target.gke.cluster)'
   ```

   Example out will be similar to:

   ```sh
   TARGET_ID: target-1
   DESCRIPTION: development 1 cluster
   CLUSTER: projects/qwiklabs-gcp-00-bed84b592c08/locations/us-central1/clusters/qwiklabs-developer-1-cluster

   TARGET_ID: target-2
   DESCRIPTION: development cluster 2
   CLUSTER: projects/qwiklabs-gcp-00-bed84b592c08/locations/us-central1/clusters/qwiklabs-developer-2-cluster
   ```

The Cloud Deploy pipeline whill push application code to both of the GKE clusters. Next, you will setup multiclusters and service mesh

## Task 6. Setting Up GKE Fleet Management and Service Mesh (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

As a Platform Engineer, establishing a well-configured Google Kubernetes Engine (GKE) environment with fleet management and a service mesh is key to enhancing developer productivity and streamlining operations. The following gcloud commands will guide you through setting up the necessary APIs, configuring a fleet, and enabling a service mesh. This foundational work is vital as it provides a standardized and resilient platform, abstracting complexities of multi-cluster management and inter-service communication, which ultimately makes developers' jobs significantly easier.

1. Enable Required APIs necessary for GKE, fleet management, and service mesh in the `Service Project`

   ```sh
   gcloud services enable \
    multiclusterservicediscovery.googleapis.com \
    gkehub.googleapis.com \
    container.googleapis.com \
    cloudresourcemanager.googleapis.com \
    trafficdirector.googleapis.com \
    dns.googleapis.com \
    --project=$SERVICE_PROJECT_ID
   ```

2. Enable Fleet Mesh Feature in the `Service Project`

   ```sh
   gcloud container fleet mesh enable --project=$SERVICE_PROJECT_ID
   ```

3. Enable Multi-Cluster Services (MCS) in the `Service Project` to allow service discovery and communication across your GKE clusters within the fleet

   ```sh
   gcloud container fleet multi-cluster-services enable \
    --project $SERVICE_PROJECT_ID

   ```

4. Register first GKE Cluster `qwiklabs-developer-1-cluster` to the fleet and enable workload idenity.

   ```sh
   gcloud container fleet memberships register qwiklabs-developer-1-cluster \
    --gke-cluster $REGION/qwiklabs-developer-1-cluster \
    --enable-workload-identity \
    --project $SERVICE_PROJECT_ID
   ```

5. Register second GKE Cluster `qwiklabs-developer-2-cluster` to the fleet and enable workload idenity.

   ```sh
   gcloud container fleet memberships register qwiklabs-developer-2-cluster \
    --gke-cluster $REGION/qwiklabs-developer-2-cluster \
    --enable-workload-identity \
    --project $SERVICE_PROJECT_ID
   ```

6. Grant Network Viewer Role to MCS Importer to allow the Multi-Cluster Services importer service account the necessary network viewing permissions within your `Service Project`:

   ```sh
   gcloud projects add-iam-policy-binding $SERVICE_PROJECT_ID \
    --member "serviceAccount:$SERVICE_PROJECT_ID.svc.id.goog[gke-mcs/gke-mcs-importer]" \
    --role "roles/compute.networkViewer"
   ```

7. Configure mesh management to be automatic for the first registered cluster membership

   ```sh
   gcloud container fleet mesh update --management automatic --memberships qwiklabs-developer-1-cluster --project=$SERVICE_PROJECT_ID
   ```

8. Configure mesh management to be automatic for the second registered cluster membership

   ```sh
   gcloud container fleet mesh update --management automatic --memberships qwiklabs-developer-2-cluster --project=$SERVICE_PROJECT_ID
   ```

<ql-infobox>
<strong>Note:</strong>Wait for any mesh provisioning activities triggered by the above steps to complete successfully.
</ql-infobox>

With the core GKE fleet and service mesh infrastructure now configured by the Platform Engineer, the clusters are part of a managed collective. The next crucial phase, also typically handled by the Platform Engineer, involves preparing these individual GKE clusters for direct developer use. This includes setting up command-line access and creating Istio-enabled namespaces, ensuring developers have secure and ready-to-use environments for their applications.

## Task 7. Prepare Developer GKE Clusters (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

With the GKE fleet and service mesh established, the Platform Engineer now finalizes the individual GKE clusters for developer deployments. This task involves accessing each cluster and setting up application namespaces with Istio sidecar injection enabled, which simplifies service mesh adoption for developers.

1. Configure kubectl to interact with your first developer GKE cluster (e.g., qwiklabs-developer-1-cluster in us-central1). Adjust the cluster name and region as needed:

   ```sh
   gcloud container clusters get-credentials qwiklabs-developer-1-cluster --region $REGION --project $SERVICE_PROJECT_ID
   ```

2. Create a dedicated namespace (e.g., dev-1) for applications in the first cluster:

   ```sh
   kubectl create ns dev-1
   ```

3. Enable automatic Istio sidecar injection for deployed workloads within that namespace:

   ```sh
   kubectl label namespace dev-1 istio-injection=enabled
   ```

4. Configure kubectl to interact with your second developer GKE cluster (e.g., qwiklabs-developer-2-cluster in us-central1). Adjust the cluster name and region as needed:

   ```sh
   gcloud container clusters get-credentials qwiklabs-developer-2-cluster --region $REGION --project $SERVICE_PROJECT_ID
   ```

5. Create a dedicated namespace (e.g., dev-1) for applications in the second cluster. You can use the same namespace name as in Cluster 1, as namespaces are distinct per cluster:

   ```sh
   kubectl create ns dev-1
   ```

6. Label the namespace in the second cluster to enable automatic Istio sidecar injection:

   ```sh
   kubectl label namespace dev-1 istio-injection=enabled
   ```

## Task 8. Docker Build & Push to Artifact Registry (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step, you will build and push the Docker image to artifact registry and configuring GKE for service mesh.

```sh
gcloud builds triggers run docker-build-push-trigger --project=$HOST_PROJECT_ID --branch=app --region=$REGION
```

| Trigger                     | Description                                                                                                                                                              | Duration      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------- |
| `docker-build-push-trigger` | Builds the online boutique application code and pushes the resulting Docker image to Artifact Registry, Configuring service mesh and adding labels in default namespace. | 17 min 30 sec |

The `docker-build-push-trigger` trigger has built and pushed the container images to Artifact Registry.

## Task 9. Online Boutique: Multi-Cluster Deployment via Cloud Build & Deploy (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

You will deploy the online boutique application across multiple GKE clusters using Cloud Build and Cloud Deploy.

1. Run the below command in cloud shell to excute service-1-application-deployment-trigger.

   ```sh
   gcloud builds triggers run service-1-application-deployment-trigger --project=$HOST_PROJECT_ID --branch=app --region=$REGION
   ```

   After Completetion of this trigger <strong>navigate</strong> to the <strong>Cloud Deploy</strong> and check the <strong>release logs</strong>. It suppose to be failed with render Error <strong>(The rollout failed with deployFailureCause: RELEASE_FAILED. (Release render operation ended in failure.))</strong>
   Follow the step to navigate to the logs.

2. Navigate to the [Cloud Deploy pipelines](https://console.cloud.google.com/deploy/delivery-pipelines/us-central1/qwiklabs-delivery-service-pipeline)

3. In the <strong>Delivery pipeline details</strong> section, select <strong>Rollouts</strong> then click on the hyperlinked Name of the failed release.

4. Here you can check the <strong>ERROR</strong>.

   <img src="img/Error.png" alt="SEARCH_BAR" width="500"> <br>

   <strong>(The rollout failed with deployFailureCause: RELEASE_FAILED. (Release render operation ended in failure.))</strong>

5. Select the <strong> Summary </strong>, then click on the <strong>Render logs</strong>

6. Scroll through the <strong>Build details</strong> to view the error which caused this rollout to fail. Example output

   ```sh
    - stderr: "Error: must build at directory: not a valid directory: evalsymlink failure on '/workspace/source/kubernetes-manifests-service-1XXXXX' : lstat /workspace/source/kubernetes-manifests-service-1XXXXX: no such file or directory\n"
   ```

   In the next task, you will troubleshoot this error and ensure the application is deployed to both clusters.

## Task 10. Troubleshooting Steps (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

In this task you will resolve the error with the release from the previous step by update the 

1. In the terminal, ensure you are in the `app-repo` directory, switch to the app branch

   ```sh
   git checkout app
   ```

2. Open the Cloudshell editor

   ```sh
   vim app/skaffold-service-1.yaml
   ```

3. In the terminal type `i` to update the manifests.kustomize.paths replace the path value form `kubernetes-manifests-service-1xxxxx` to `kubernetes-manifests-service-1`.

4. To save the file `ESC` + `:wq`

5. Commit the changes using below command.

   ```sh
   git add .
   git commit -m "fix the error"
   git push origin app
   ```

6. Now, Navigate to the Cloud build -> Trigger. Execute the <strong>service-1-application-deployment-trigger</strong>.

   Run the below command in cloud shell to excute service-1-application-deployment-trigger.

   ```sh
   gcloud builds triggers run service-1-application-deployment-trigger --project=$HOST_PROJECT_ID --branch=app --region=$REGION
   ```

7. After compeletion of trigger, validate the cloud deploy release again.

   <ql-infobox>
   <strong>Note</strong> After the <strong>service-1-application-deployment-trigger</strong> completes, the <strong>service-2-application-deployment-trigger</strong> automatically initiates, deploying microservices to the second cluster.
   </ql-infobox>

   | Trigger                                    | Description                                                                                           | Duration    |
   | ------------------------------------------ | ----------------------------------------------------------------------------------------------------- | ----------- |
   | `service-1-application-deployment-trigger` | Deploys specific application components to the `qwiklabs-developer-1-cluster` in the service project. | 2 min 5 sec |

   | Trigger                                    | Description                                                                                        | Duration    |
   | ------------------------------------------ | -------------------------------------------------------------------------------------------------- | ----------- |
   | `service-2-application-deployment-trigger` | Deploys other application components to the `qwiklabs-developer-2-cluster` in the service project. | 2 min 5 sec |

8. After completion of `service-2-application-deployment-trigger` you can access the frontend of Online Boutique Application.

   In order to access the website, navigate to the Gateways, Ingress and Services where under service section the `frontend-external` IP address will be available.
   Copy the ip address present under IP Addresses and paste it in the browser.

   ```sh
   gcloud container clusters get-credentials qwiklabs-developer-1-cluster --region $REGION --project $SERVICE_PROJECT_ID
   export EX_IP="http://$(kubectl get service frontend-external -n dev-1 -o jsonpath='{.status.loadBalancer.ingress[0].ip}')" && echo $EX_IP
   ```

9. Enable the `productcatalog-cicd-trigger` cloud build trigger in the host project.

   Navigate to the cloud build -> Trigger.

   Click the menu (vertical ellipses) located at the right end of `productcatalog-cicd-trigger` in row.
   Select Enable.

   <img src="img/productcatalog-enable.png" alt="ENABLE-TRIGGER" width="500"> <br>

## Task 11. Make Changes in Product Catalog Microservice (Developer) ![usr2](img/qwiklabs-icons2-small.drawio.png)

In this task, you will sign into the console as the Developer persona, modify the application code and push to repository.

1. In the Google Cloud Console title bar, **click** on the Profile Icon and click **Add account**

2. Sign in using the Developer username {{{ user_2.username }}} and password {{{ user_2.password }}}

3. In the CloudShell terminal, clone the Application Repository using git clone command.

   <ql-code-block language="bash" templated>
   export USER_EMAIL={{{user_2.username|['DEVELOPER_PERSONA_EMAIL]'}}}
   export USER={{{user_2.local_username|['DEVELOPER_PERSONA_USERNAME]'}}}
   export PROJECT_ID={{{project_1.project_id|'[PROJECT_ID]'}}}
   </ql-code-block>

4. Clone the application repo:

   ```sh
   gcloud source repos clone app-repo --project=$PROJECT_ID
   cd app-repo
   git checkout app
   ```

5. Update the `git config` credentials to use the Developer person

   ```sh
   git config user.email "$USER_EMAIL"
   git config user.name "$USER"
   ```

6. In the Editor, open the `src/productcatalogservice-v2/products.json` file.

   ```sh
   cloudshell edit app/src/productcatalogservice-v2/products.json
   ```

7. Replace the old names values and description with the new names values and description as mentioned in the below table.

   | old name value | old description                                                                     |
   | -------------- | ----------------------------------------------------------------------------------- |
   | "Tank Top"     | " SALE!!!."                                                                         |
   | "Watch"        | "SALE!!! This gold-tone stainless steel watch will work with most of your outfits." |

   | new name value       | new description                                                                                      |
   | -------------------- | ---------------------------------------------------------------------------------------------------- |
   | "Tank Top for Women" | "SALE!!! SALE!!! SALE!!!."                                                                           |
   | "Watch for men"      | "SALE!!! SALE!!! SALE!!!. This gold-tone stainless steel watch will work with most of your outfits." |

8. Commit the changes to the repository.

   ```sh
   git add .
   git commit -m "code added"
   git push origin app
   ```

   <ql-infobox>
   <strong>Note</strong>
   The `productcatalog-cicd-trigger` trigger will automatically execute in the Cloud Build trigger tab.
   </ql-infobox>

9. Navigate to the <strong>Cloud Build</strong> and validate the cloud build logs under <strong>History</strong> tab.

   | Trigger                       | Description                                         | Duration |
   | ----------------------------- | --------------------------------------------------- | -------- |
   | `productcatalog-cicd-trigger` | Deploying the V2 version of Product Catalog Service | 4 min    |

10. After Completetion of productcatalog-cicd-trigger , to see the changes on application's frontend. Refresh the browser after few minutes later .
    <img src="img/Service-v2.png" alt="V1 Service" width="600.00">

## Task 12. Create dev-1-team and dev-2-team within the GKE (Platform Engineer)![usr1](img/qwiklabs-icons1-small.drawio.png)

In this step, you will [set up teams for your GKE fleet](https://cloud.google.com/kubernetes-engine/fleet-management/docs/setup-teams).

1. In the <strong>Google Cloud Console</strong>, select the <strong>service project ID</strong>.
2. In the search bar type <strong>Kubernetes Engine</strong> and under <strong>Resource Management</strong> click on <strong>Teams</strong>.

   <img src="img/team1.png" alt="Team1" width="500">

3. Under <strong>Team basics</strong>, enter the name as <strong>dev-team-1</strong>.
4. Under <strong>View team scope role definition details</strong> <strong>type 1</strong> select <strong>user</strong>.
5. In the <strong>User or Group 1</strong>field, enter the email address of the <strong>Developer user</strong> which is located on the lab instruction tab.
6. Select <strong>Role 1</strong> as <strong>Scope Admin</strong> amd click on Continue.

   <img src="img/team2.png" alt="Team2" width="500">

7. In the <strong>Cluter</strong> dropdown box select the <strong>qwiklabs-developer-1-cluster</strong> and and click on </strong>OK</strong>.

   <img src="img/team3.png" alt="Team3" width="500">

8. Click on Create Team Scope.
9. Under <strong>Team basics</strong>, enter the name as <strong>dev-team-2</strong>.
10. Under <strong>View team scope role definition details</strong> <strong>type 1</strong> select <strong>user</strong>.
11. In the <strong>User or Group 1</strong>field, enter the email address of the <strong>Developer user</strong> which is located on the SSO panel of the lab.
12. Select <strong>Role 1</strong> as <strong>Scope Admin</strong> amd click on Continue.
13. In the <strong>Cluter</strong> dropdown box select the <strong>qwiklabs-developer-2-cluster</strong> and click on </strong>OK</strong>.
    <img src="img/team5.png" alt="Team5" width="500">

14. Click on <strong>Create Team Scope</strong>.

[You are able to monitor a particular cluster within the team scope](https://cloud.google.com/kubernetes-engine/fleet-management/docs/team-management-overview)

## Task 13. Create an SLOs for the Service (Platform Engineer) ![usr1](img/qwiklabs-icons1-small.drawio.png)

<ql-infobox>
<strong>Note:</strong> To ensure optimal performance for our application, we'll establish <strong>Service Level Indicators (SLIs)</strong> and <strong>Objectives (SLOs)</strong> for better <strong>Observability</strong>. Specifically, we'll monitor availability via <strong>request-based metrics</strong>, aiming for a <strong>99.9% uptime SLO</strong>. Automated <strong>email alerts</strong> will trigger upon SLO breaches, enabling swift issue resolution. This proactive approach to reliability enhances user experience and facilitates data-driven service management.
</ql-infobox>

In this Section we will Create a <strong>Service level Objective</strong> for our <strong>Frontend</strong> service to have better observability in service project.

1. In The Console search Box, Search For [SLOs](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/ui/create-slo), It can also be found under the <strong>Observability</strong> section.

2. Once you are in the SLO section, you first need to [Define a Service](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring/ui/define-svc), Click on <strong>+ Define</strong> service.

<img src="img/define-service.png" alt="Profile" width="600.00">

3. Select the service as <strong>"frontend"</strong> service, after Service selection a json script will be created, Click on <strong>"Submit"</strong>.

<img src="img/step2.png" alt="Profile" width="600.00">

4. Once Service is defined, You can proceed with <strong>creation of SLO</strong>, Click on the <strong>service</strong> created.

<img src="img/step-3-create-sto.png" alt="Profile" width="600.00">

5. Click on<strong> "+ Create SLO"</strong>, Set your <strong>service-level indicator</strong> [(SLI)](https://cloud.google.com/stackdriver/docs/solutions/slo-monitoring#defn-sli), choose <strong>"Availability"</strong> as a metric and choose method of evaluation as <strong>"Request-based"</strong>, then select <strong>"Continue"</strong>

<img src="img/step-4-SLI.png" alt="Profile" width="600.00">

6. You will get a preview of the <strong>SLI details</strong> on the next page. Click on <strong>"Continue"</strong>.

<img src="img/step-5-review.png" alt="Profile" width="600.00">

7. On the next page, you will </strong>set your SLO</strong>. Select your period length as <strong>"Calender day"</strong>, and Performance goal at <strong>"99.9%"</strong>.

<img src="img/step-6-set-slo.png" alt="Profile" width="600.00">

8. Once done you can Review the json and Click on <strong>"Create SLO"</strong>, this will <strong>create SLO</strong>.

<img src="img/step-7-sto-final.png" alt="Profile" width="600.00">

9. Select the SLO, Now we will create an <strong>Alert for SLO</strong>, Click on the <strong>"Create SLO alert"</strong>.

<img src="img/step8-create-alert.png" alt="Profile" width="400.00">

10. Now we can <strong>Set SLO alert conditions</strong> Review it and click <strong>Next</strong>.

    <img src="img/step9-slo.png" alt="Profile" width="400.00">

11. On Next page Click on <strong>Manage Notification Channels</strong>, goto <strong>Email</strong> and click on <strong>"Add new"</strong> button.

    <img src="img/step-10-manage-noti.png" alt="Profile" width="400.00">

12. Enter student <strong>email address</strong> Created above, and Set Display name as <strong>"Plaform Engineer"</strong>.

    <img src="img/step11-email.png" alt="Profile" width="300.00">

13. Select <strong>Platform Engineer</strong> as Notification channel, then Click <strong>Next</strong>, Click <strong>Save</strong>.

    <img src="img/step12-select-email.png" alt="Profile" width="300.00">

14. This will <strong>create a Alert</strong> for your Service, Whenever the Service goes down the event will be <strong>notified</strong> on the mail.

    <img src="img/step13-final.png" alt="Profile" width="400.00">

## Congratulations!

You have completed the lab and have a solid understanding of Terraform, GKE , Service Mesh , Fleet and SLO/SLI in Google Cloud.

## Key Takeaways

- Practical experience with GKE, Fleet, and CSM.
- Understanding of microservices architecture and deployment strategies.
- Knowledge of CI/CD pipelines using Cloud Build.
- Hands-on experience with service monitoring and SLO/SLI implementation.
- Collaboration between Platform Engineers and Developers in a cloud-native environment.

## Lab Resources

- Terraform Code: Pre-configured with reusable modules and variables.
- Google Cloud Build/Deploy Pipeline: Pre-defined CI/CD pipeline template.
- GKE Cluster: To deploy the application.
- Documentation: Detailed instructions and references for Terraform, Cloud Build, Cloud Deploy and GitOps practices

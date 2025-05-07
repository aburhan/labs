Provision a managed Cloud Service Mesh control plane on GKE

bookmark_border
Note: This guide only supports Cloud Service Mesh with Istio APIs and does not support Google Cloud APIs. For more information see, Cloud Service Mesh overview.
Cloud Service Mesh is a Google-managed service mesh that you simply enable. Google handles reliability, upgrades, scaling, and security for you.

This page shows you how to use the fleet API to set up managed Cloud Service Mesh using Istio APIs.

Prerequisites
As a starting point, this guide assumes that you have:

A Cloud project
A Cloud Billing account
Obtained the required permissions to provision Cloud Service Mesh
Enabled Workload Identity on your clusters and node pools.
Requirements
One or more clusters with a supported version of GKE, in one of the supported regions.
Note that managed Cloud Service Mesh uses GKE release channels to balance between stability and upgrade speed. New changes to Cloud Service Mesh in-cluster components (including CNI, MDPC, proxies, and Istio CRDs) will rollout to clusters that subscribe GKE rapid channel first. They will then be promoted to GKE regular channel, and finally the GKE stable channel, once they demonstrate enough stability.

Managed Cloud Service Mesh doesn't support changing GKE release channels safely.
If you do change the GKE release channel, Cloud Service Mesh automatically upgrades/downgrades the in-cluster components (CNI, MDPC, default injected proxy version and Istio CRDs) to align with the current GKE release channel.
Ensure that your cluster has enough capacity for the required components that managed Cloud Service Mesh installs in the cluster.

The mdp-controller deployment in kube-system namespace requests cpu: 50m, memory: 128Mi.
The istio-cni-node daemonset in kube-system namespace requests cpu: 100m, memory: 100Mi on each node.
Ensure that the organizational policy constraints/compute.disableInternetNetworkEndpointGroup is disabled. If the policy is enabled, ServiceEntry may not work.

Ensure that the client machine that you provision managed Cloud Service Mesh from has network connectivity to the API server.

Your clusters must be registered to a fleet. This is included in the instructions, or can be done separately prior to the provision.

Your project must have the Service Mesh fleet feature enabled. This is included in the instructions or can be done separately.

GKE Autopilot is only supported with GKE version 1.21.3+.

Cloud Service Mesh can use multiple GKE clusters in a single-project single-network environment or a multi-project single-network environment.

If you join clusters that are not in the same project, they must be registered to the same fleet host project, and the clusters must be in a shared VPC configuration together on the same network.
For a single-project multi-cluster environment, the fleet project can be the same as the cluster project. For more information about fleets, see Fleets Overview.
For a multi-project environment, we recommend that you host the fleet in a separate project from the cluster projects. If your organizational policies and existing configuration allow it, we recommend that you use the shared VPC project as the fleet host project. For more information, see Setting up clusters with Shared VPC.
Roles required to install Cloud Service Mesh
The following table describes the roles that are required to install managed Cloud Service Mesh.

Role name	Role ID	Grant location	Description
GKE Hub Admin	roles/gkehub.admin	Fleet project	Full access to GKE Hubs and related resources.
Service Usage Admin	roles/serviceusage.serviceUsageAdmin	Fleet project	Ability to enable, disable, and inspect service states, inspect operations, and consume quota and billing for a consumer project. (Note 1)
CA Service Admin Beta	roles/privateca.admin	Fleet project	Full access to all CA Service resources. (Note 2)
Roles required to run Cloud Service Mesh
The following table describes the roles that are required by the service account to run managed Cloud Service Mesh. If your network or cluster projects differ from the fleet host project, you need to give the service account in the fleet project access to these roles in the other projects.

Role name	Role ID	Grant location	Description
Anthos Service Mesh Service Agent	roles/anthosservicemesh.serviceAgent	Fleet project	
Mesh Managed Control Plane Service Agent (legacy)	roles/meshcontrolplane.serviceAgent	Fleet project	This is a legacy role that was part of older installations of Cloud Service Mesh. If your installation has this service role, you can leave it as is. New installations don't need this role.
Limitations
Caution: Note the following fleet-specific limitations: - All clusters in a particular Cloud Service Mesh must be registered to the same fleet. The clusters cannot be registered to different fleets. - All clusters in a project that includes a Cloud Service Mesh cluster must be registered to the same fleet.
We recommend that you review the list of Cloud Service Mesh supported features and limitations. In particular, note the following:

The IstioOperator API isn't supported since its main purpose is to control in-cluster components.

Note: You can still enable optional features without using the IstioOperator API, see Enabling optional features on managed Cloud Service Mesh. You can also use the migration tool included with asmcli to automatically convert other IstioOperator optional features to be compatible with managed control plane. For more information, see Migrate from IstioOperator.
Using Certificate Authority Service (CA Service) requires configuring Cloud Service Mesh per cluster, and is not supported when using the fleet-default configuration in GKE Enterprise.

For GKE Autopilot clusters, cross-project setup is only supported with GKE 1.23 or later.

For GKE Autopilot clusters, in order to adapt to the GKE Autopilot resource limit, the default proxy resource requests and limits are set to 500m CPU and 512 Mb memory. You can override the default values using custom injection.

During the provisioning process for a managed control plane, Istio CRDs are provisioned in the specified cluster. If there are existing Istio CRDs in the cluster, they will be overwritten.

Istio CNI and Cloud Service Mesh are not compatible with GKE Sandbox. Therefore, managed Cloud Service Mesh with the TRAFFIC_DIRECTOR implementation does not support clusters with GKE Sandbox enabled.

Before you begin
In the Google Cloud console, on the project selector page, select or create a Google Cloud project.

Go to project selector

Make sure that billing is enabled for your Google Cloud project.

Configure gcloud (even if you are using Cloud Shell).
Authenticate with the Google Cloud CLI, where FLEET_PROJECT_ID is the ID of your Fleet Host project. Generally, the FLEET_PROJECT_ID is created by default and has the same name as the project.


       gcloud auth login --project FLEET_PROJECT_ID
Update the components:


       gcloud components update
Enable the required APIs on your fleet host project.


  gcloud services enable mesh.googleapis.com \
      --project=FLEET_PROJECT_ID
Enabling mesh.googleapis.com enables the following APIs:
Note: Some required APIs have transitive dependencies on other APIs.
API	Purpose	Can Be Disabled
meshconfig.googleapis.com	Cloud Service Mesh uses the Mesh Configuration API to relay configuration data from your mesh to Google Cloud. Additionally, enabling the Mesh Configuration API allows you to access the Cloud Service Mesh pages in the Google Cloud console and to use the Cloud Service Mesh certificate authority.	No
meshca.googleapis.com	Related to Cloud Service Mesh certificate authority used by managed Cloud Service Mesh.	No
container.googleapis.com	Required to create Google Kubernetes Engine (GKE) clusters.	No
gkehub.googleapis.com	Required to manage the mesh as a fleet.	No
monitoring.googleapis.com	Required to capture telemetry for mesh workloads.	No
stackdriver.googleapis.com	Required to use the Services UI.	No
opsconfigmonitoring.googleapis.com	Required to use the Services UI for off-Google Cloud clusters.	No
connectgateway.googleapis.com	Required so that the managed Cloud Service Mesh control plane can access mesh workloads.	Yes*
trafficdirector.googleapis.com	Enables a highly available and scalable managed control plane.	Yes*
networkservices.googleapis.com	Enables a highly available and scalable managed control plane.	Yes*
networksecurity.googleapis.com	Enables a highly available and scalable managed control plane.	Yes*
Warning: Disabling connectgateway.googleapis.com, trafficdirector.googleapis.com, networkservices.googleapis.com, and/or networksecurity.googleapis.com causes managed Cloud Service Mesh control plane to stop working. If the fleet does not use managed Cloud Service Mesh on any cluster, then these APIs can be disabled.
Configure managed Cloud Service Mesh
The steps required to provision managed Cloud Service Mesh using the fleet API depend on whether you prefer to enable by default for new fleet clusters or enable it per cluster.

Configure for your fleet
You must have Google Kubernetes Engine (GKE) Enterprise edition enabled to enable managed Cloud Service Mesh as a default configuration for your fleet. This means that every new GKE on Google Cloud cluster registered during cluster creation will have managed Cloud Service Mesh enabled on the cluster. You can find out more about fleet default configuration in Manage fleet-level features.

Enabling managed Cloud Service Mesh as a default configuration for your fleet and registering clusters to the fleet during cluster creation only supports Mesh CA. If you want to use Certificate Authority Service, we recommend that you enable it per cluster.

To enable fleet-level defaults for managed Cloud Service Mesh, complete the following steps:

Console
gcloud
To configure fleet-level defaults using the Google Cloud CLI, you must establish the following settings:

Fleet-level settings

Create a mesh.yaml file that only contains the single line management: automatic:



echo "management: automatic" > mesh.yaml
Enable Cloud Service Mesh for your fleet:



gcloud container fleet mesh enable --project FLEET_PROJECT_ID \
    --fleet-default-member-config mesh.yaml
If you see the following error, then you need to enable GKE Enterprise.



ERROR: (gcloud.container.fleet.mesh.enable) FAILED_PRECONDITION: The
[anthos.googleapis.com] service is required for this operation and is not
enabled for the project [PROJECT_NUMBER]. Please use the Google Developers
Console to enable it.: failed precondition
Network-level settings

If your network's project differs from your fleet host project (for example you are using a shared VPC), you must allow Cloud Service Mesh service accounts in the fleet project to access the network project. You only need to do this once for the network project.

Grant service accounts in the fleet project permission to access the network project:



gcloud projects add-iam-policy-binding "NETWORK_PROJECT_ID"  \
    --member "serviceAccount:service-FLEET_PROJECT_NUMBER@gcp-sa-servicemesh.iam.gserviceaccount.com" \
    --role roles/anthosservicemesh.serviceAgent
Cluster-level settings

When you're ready to create clusters to use with Cloud Service Mesh, create and register them in a single step with Google Cloud CLI to use the default configuration. For example:



gcloud container clusters create-auto CLUSTER_NAME \
    --fleet-project FLEET_PROJECT_ID \
    --location=LOCATION
You can get the project number for your fleet project by running the following command:



gcloud projects list --filter="FLEET_PROJECT_ID" --format="value(PROJECT_ID)"
The --location flag is the compute zone or region (such as us-central1-a or us-central1) for the cluster.

If your cluster's project differs from your fleet host project, you must allow Cloud Service Mesh service accounts in the fleet project to access the cluster project, and enable required APIs on the cluster project. You only need to do this once for each cluster project.

Grant service accounts in the fleet project permission to access the cluster project:



gcloud projects add-iam-policy-binding "CLUSTER_PROJECT_ID"  \
    --member "serviceAccount:service-FLEET_PROJECT_NUMBER@gcp-sa-servicemesh.iam.gserviceaccount.com" \
    --role roles/anthosservicemesh.serviceAgent
Enable the Mesh API on the cluster's project:



gcloud services enable mesh.googleapis.com \
  --project=CLUSTER_PROJECT_ID
Replace CLUSTER_PROJECT_ID with the unique identifier of your cluster project. If you created your cluster in the same project as your fleet, then the CLUSTER_PROJECT_ID is the same as the FLEET_PROJECT_ID.

Proceed to Verify the control plane has been provisioned.

Configure per cluster
Use the following steps to configure managed Cloud Service Mesh for each cluster in your mesh individually.

Enable the Cloud Service Mesh fleet feature
Enable the mesh feature on the fleet project. Note that if you plan to register multiple clusters, enabling the Cloud Service Mesh fleet feature happens at the fleet-level so you only have to run this command once.



gcloud container fleet mesh enable --project FLEET_PROJECT_ID
Register clusters to a fleet
Register a GKE cluster using fleet workload identity. The --location flag is the compute zone or region (such as us-central1-a or us-central1) for the cluster.



gcloud container clusters update CLUSTER_NAME \
  --location CLUSTER_LOCATION \
  --fleet-project FLEET_PROJECT_ID
Verify your cluster is registered:



gcloud container fleet memberships list --project FLEET_PROJECT_ID
Example output:



NAME                 EXTERNAL_ID                           LOCATION
cluster-1            1d8e255d-2b55-4df9-8793-0435461a2cbc  us-central1
Make note of the MEMBERSHIP_NAME, as you will need it when you enable automatic management.

If your cluster's network's project differs from your fleet host project (for example you are using a shared VPC), you must allow Cloud Service Mesh service accounts in the fleet project to access the network project. You only need to do this once for the network project.

Grant service accounts in the fleet project permission to access the network project:



 gcloud projects add-iam-policy-binding "NETWORK_PROJECT_ID"  \
      --member "serviceAccount:service-FLEET_PROJECT_NUMBER@gcp-sa-servicemesh.iam.gserviceaccount.com" \
      --role roles/anthosservicemesh.serviceAgent
If your cluster's project differs from your fleet host project, you must allow Cloud Service Mesh service accounts in the fleet project to access the cluster project, and enable required APIs on the cluster project.

You only need to do this once for each cluster project. If you previously configured managed Cloud Service Mesh for this combination of cluster and fleet projects, then these changes have already been applied and you don't have to run the following commands.

Grant service accounts in the fleet project permission to access the cluster project:



 gcloud projects add-iam-policy-binding "CLUSTER_PROJECT_ID" \
     --member "serviceAccount:service-FLEET_PROJECT_NUMBER@gcp-sa-servicemesh.iam.gserviceaccount.com" \
     --role roles/anthosservicemesh.serviceAgent
Enable the Mesh API on the cluster's project:



 gcloud services enable mesh.googleapis.com \
     --project=CLUSTER_PROJECT_ID
Configure Certificate Authority Service (Optional)
If your service mesh deployment requires Certificate Authority Service (CA Service), then follow Configure Certificate Authority Service for managed Cloud Service Mesh to enable it for your fleet. Make sure to complete all steps before proceeding to the next section.

Enable automatic management
Run the following command to enable automatic management:

Note: You can apply this change to multiple clusters at a time by passing a comma-separated list of membership names as the --memberships parameter.


  gcloud container fleet mesh update \
     --management automatic \
     --memberships MEMBERSHIP_NAME \
     --project FLEET_PROJECT_ID \
     --location MEMBERSHIP_LOCATION
where:

MEMBERSHIP_NAME is the membership name listed when you verified that your cluster was registered to the fleet.
MEMBERSHIP_LOCATION is the location of your membership (either a region, or global).

If you recently created the membership using the command in this guide, this should be the region of your cluster. If you have a zonal cluster, use the region corresponding to the cluster's zone. For example, if you have a zonal cluster in us-central1-c, then use the value us-central1.

This value may be global if you registered prior to May 2023, or if you specified the global location when registering the membership. You can check your membership's location with gcloud container fleet memberships list --project FLEET_PROJECT_ID.

Terraform support
Cloud Service Mesh supports provisioning via Terraform through the GKEHub feature membership module.

Please see the Provision managed service mesh tutorial for detailed instructions.

Verify the control plane has been provisioned
After a few minutes, verify that the control plane status is ACTIVE:



gcloud container fleet mesh describe --project FLEET_PROJECT_ID
The output is similar to:



...
membershipSpecs:
  projects/746296320118/locations/us-central1/memberships/demo-cluster-1:
    mesh:
      management: MANAGEMENT_AUTOMATIC
membershipStates:
  projects/746296320118/locations/us-central1/memberships/demo-cluster-1:
    servicemesh:
      controlPlaneManagement:
        details:
        - code: REVISION_READY
          details: 'Ready: asm-managed'
        state: ACTIVE
        implementation: ISTIOD | TRAFFIC_DIRECTOR
      dataPlaneManagement:
        details:
        - code: OK
          details: Service is running.
        state: ACTIVE
    state:
      code: OK
      description: 'Revision(s) ready for use: asm-managed.'
...
Take note of the control plane displayed in the implementation field, either ISTIOD or TRAFFIC_DIRECTOR. See Cloud Service Mesh supported features for control plane differences and supported configurations, and for how the control plane implementation is selected.

Configure kubectl to point to the cluster
The following sections involve running kubectl commands against each one of your clusters. Before proceeding through the following sections, run the following command for each of your clusters to configure kubectl to point to the cluster.



gcloud container clusters get-credentials CLUSTER_NAME \
      --location CLUSTER_LOCATION \
      --project CLUSTER_PROJECT_ID
Note that an ingress gateway isn't automatically deployed with the control plane. Decoupling the deployment of the ingress gateway and control plane lets you manage your gateways in a production environment. If you want to use an Istio ingress gateway or an egress gateway, see Deploy gateways. If you want to use the Kubernetes Gateway API, see Prepare Gateway for Mesh. To enable other optional features, see Enabling optional features on Cloud Service Mesh.

Managed data plane
If you use managed Cloud Service Mesh, Google fully manages upgrades of your proxies.

With the managed data plane feature enabled, the sidecar proxies and injected gateways are actively and automatically updated in conjunction with the managed control plane by restarting workloads to re-inject new versions of the proxy. This starts after the control plane has been upgraded and normally completes within 2 weeks after starting.

Note that the managed data plane relies on the GKE release channel. If you change the GKE release channel while the managed data plane is enabled, managed Cloud Service Mesh will update the proxies of all existing workloads like an managed data plane rollout.

If disabled, proxy management is done passively - driven by the natural lifecycle of the pods in the cluster and must be manually triggered by the user to control the update rate.

The managed data plane upgrades proxies by evicting pods that are running earlier versions of the proxy. The evictions are done gradually, honoring the pod disruption budget and controlling the rate of change.

Note: If the cluster has a GKE maintenance window defined, pod evictions will commence at the start of the next available maintenance window and continue without pause until all managed pods have been updated (usually 12 hours). Maintenance window is not observed for CVE related rollouts.
The managed data plane doesn't manage the following:

Uninjected pods
Manually injected pods
Jobs
StatefulSets
DaemonSets
Disable the managed data plane (optional)
If you are provisioning managed Cloud Service Mesh on a new cluster, then you can disable the managed data plane completely, or for individual namespaces or pods. The managed data plane will continue to be disabled for existing clusters where it was disabled by default or manually.

To disable the managed data plane at the cluster level and revert back to managing the sidecar proxies yourself, change the annotation:



kubectl annotate --overwrite controlplanerevision -n istio-system \
  mesh.cloud.google.com/proxy='{"managed":"false"}'
To disable the managed data plane for a namespace:



kubectl annotate --overwrite namespace NAMESPACE \
  mesh.cloud.google.com/proxy='{"managed":"false"}'
To disable the managed data plane for a pod:



kubectl annotate --overwrite pod POD_NAME \
  mesh.cloud.google.com/proxy='{"managed":"false"}'
Enable maintenance windows
If you have a GKE maintenance window configured, active upgrades will commence at the start of the next available maintenance window and continue without pause until all managed pods have been updated (usually 12 hours). The maintenance window is not observed for CVE related rollouts.

Cloud Service Mesh uses the GKE maintenance window to align with GKE.

Enable maintenance notifications
You can request to be notified about upcoming managed data plane maintenance up to a week before maintenance is scheduled. Maintenance notifications are not sent by default. You must also Configure a GKE maintenance window before you can receive notifications. When enabled, notifications are sent at least two days before the upgrade operation.

To opt in to managed data plane maintenance notifications:

Go to the Communication page.

Go to the Communication page

In the Cloud Service Mesh Upgrade row, under the Email column, select the radio button to turn maintenance notifications ON.

Each user who wants to receive notifications must opt in separately. If you want to set an email filter for these notifications, the subject line is:

Upcoming upgrade for your Cloud Service Mesh cluster "CLUSTER_LOCATION/CLUSTER_NAME".

The following example shows a typical managed data plane maintenance notification:

Subject Line: Upcoming upgrade for your Cloud Service Mesh cluster "<location/cluster-name>"

Dear Cloud Service Mesh user,

The Cloud Service Mesh components in your cluster ${instance_id} (https://console.cloud.google.com/kubernetes/clusters/details/${instance_id}/details?project=${project_id}) are scheduled to upgrade on ${scheduled_date_human_readable} at ${scheduled_time_human_readable}.

You can check the release notes (https://cloud.google.com/service-mesh/docs/release-notes) to learn about the new update.

In the event that this maintenance gets canceled, you'll receive another email.

Sincerely,

The Cloud Service Mesh Team

(c) 2023 Google LLC 1600 Amphitheater Parkway, Mountain View, CA 94043 You have received this announcement to update you about important changes to Google Cloud Platform or your account. You can opt out of maintenance window notifications by editing your user preferences: https://console.cloud.google.com/user-preferences/communication?project=${project_id}

Configure endpoint discovery (only for multi-cluster installations)
If your mesh has only one cluster, skip these multi-cluster steps and proceed to Deploy applications or Migrate applications.

Before you continue, ensure that Cloud Service Mesh is configured on each cluster.

Note: For more information on endpoint discovery, refer to Endpoint discovery with multiple control planes.
Enabling Cloud Service Mesh with the fleet API will enable endpoint discovery for this cluster. However, you must open firewall ports. To disable endpoint discovery for one or more clusters, see the instructions to disable it in Endpoint discovery between clusters with declarative API.

For an example application with two clusters, see HelloWorld service example.

Note: The HelloWorld service example uses Docker Hub. In a private cluster, the container runtime can pull container images from Artifact Registry by default. The container runtime cannot pull images from any other container image registry on the internet. You can download the image and push to Artifact Registry or use Cloud NAT to provide outbound internet access for certain private nodes. For more information, see Migrate external containers and Creating a private cluster.
Deploy applications
If you have more than one cluster in the fleet using managed Cloud Service Mesh, then ensure endpoint discovery or firewall ports are configured as intended before proceeding and deploying applications.
Enable the namespace for injection. The steps depend on your control plane implementation.

Managed (TD)
Managed (Istiod)
Apply the default injection label to the namespace:


kubectl label namespace NAMESPACE \
    istio.io/rev- istio-injection=enabled --overwrite
Validate that the namespace label is applied correctly using the following command.



  kubectl get namespace -L istio-injection
Example output:


  NAME                 STATUS   AGE     ISTIO-INJECTION
  default              Active   5m9s    enabled
At this point, you have successfully configured managed Cloud Service Mesh. If you have any existing workloads in labeled namespaces, then restart them so they get proxies injected.

If you deploy an application in a multi-cluster setup, replicate the Kubernetes and control plane configuration in all clusters, unless you plan to limit that particular config to a subset of clusters. The configuration applied to a particular cluster is the source of truth for that cluster.

Customize injection (optional)
You can override default values and customize injection settings but this can lead to unforeseen configuration errors and resulting issues with sidecar containers. Before you customize injection, read the information after the sample for notes on particular settings and recommendations.

Per-pod configuration is available to override these options on individual pods. This is done by adding an istio-proxy container to your pod. The sidecar injection will treat any configuration defined here as an override to the default injection template.

For example, the following configuration customizes a variety of settings, including lowering the CPU requests, adding a volume mount, and adding a preStop hook:



apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: hello
    image: alpine
  - name: istio-proxy
    image: auto
    resources:
      requests:
        cpu: "200m"
        memory: "256Mi"
      limits:
        cpu: "200m"
        memory: "256Mi"
    volumeMounts:
    - mountPath: /etc/certs
      name: certs
    lifecycle:
      preStop:
        exec:
          command: ["sleep", "10"]
  volumes:
  - name: certs
    secret:
      secretName: istio-certs
In general, any field in a pod can be set. However, care must be taken for certain fields:

Kubernetes requires the image field to be set before the injection has run. While you can set a specific image to override the default one, we recommend that you set the image to auto, which will cause the sidecar injector to automatically select the image to use.
Some fields in containers are dependent on related settings. For example, must be less than or equal to the CPU limit. If both fields are not properly configured, the pod may fail to start.
Kubernetes lets you set both requests and limits for resources in your Pod spec. GKE Autopilot only considers requests. For more information, see Setting resource limits in Autopilot.
Additionally, certain fields are configurable by annotations on the Pod, although it is recommended to use the above approach to customizing settings. Take additional care for the following annotations:

For GKE Standard, if sidecar.istio.io/proxyCPU is set, make sure to explicitly set sidecar.istio.io/proxyCPULimit. Otherwise the sidecar's CPU limit will be set as unlimited.
For GKE Standard, if sidecar.istio.io/proxyMemory is set, make sure to explicitly set sidecar.istio.io/proxyMemoryLimit. Otherwise the sidecar's memory limit will be set as unlimited.
For GKE Autopilot, configuring resource requests and limits using annotations might overprovision resources. Use the image template approach to avoid. See Resource modification examples in Autopilot.
For example, see the below resources annotation:



spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/proxyCPU: "200m"
        sidecar.istio.io/proxyCPULimit: "200m"
        sidecar.istio.io/proxyMemory: "256Mi"
        sidecar.istio.io/proxyMemoryLimit: "256Mi"
Migrate applications to managed Cloud Service Mesh
Warning: During the migration process, Istio CRDs are installed in the specified cluster. If there are existing Istio CRDs in the cluster, they will be overwritten.
To migrate applications from in-cluster Cloud Service Mesh to managed Cloud Service Mesh, perform the following steps:

Note: Ensure you have provisioned managed Cloud Service Mesh by following all of the preceding sections of this guide.
Replace the current namespace label. The steps depend on your control plane implementation.
Managed (TD)
Managed (Istiod)
Apply the default injection label to the namespace:


kubectl label namespace NAMESPACE \
    istio.io/rev- istio-injection=enabled --overwrite
Perform a rolling upgrade of deployments in the namespace:



kubectl rollout restart deployment -n NAMESPACE
Test your application to verify that the workloads function correctly.

If you have workloads in other namespaces, repeat the previous steps for each namespace.

If you deployed the application in a multi-cluster setup, replicate the Kubernetes and Istio configuration in all clusters, unless there is a desire to limit that configuration to a subset of clusters only. The configuration applied to a particular cluster is the source of truth for that cluster.

If you are satisfied that your application works as expected, you can remove the in-cluster istiod after you switch all namespaces to the managed control plane, or keep them as a backup - istiod will automatically scale down to use fewer resources. To remove, skip to Delete old control plane.

If you encounter problems, you can identify and resolve them by using the information in Resolving managed control plane issues and if necessary, roll back to the previous version.

Delete old control plane
Roll back
After you install and confirm that all namespaces use the Google-managed control plane, you can delete the old control plane.

kubectl delete Service,Deployment,HorizontalPodAutoscaler,PodDisruptionBudget istiod -n istio-system --ignore-not-found=true
If you used istioctl kube-inject instead of automatic injection, or if you installed additional gateways, check the metrics for the control plane, and verify that the number of connected endpoints is zero.

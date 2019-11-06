# Setting up a lab environment for 300 series labs.

The environment for the 300-series labs is depicted on the below diagram.

![Reference topolgy](/images/lab_300.png)

The core services in the environment are:
- AI Platform Notebooks - ML experimentation and development
- AI Platform Training - scalable, serverless model training
- AI Platform Prediction - scalable, serverless model serving
- BigQuery - analytics data warehouse
- Cloud Storage - unified object storage
- Kubeflow Pipelines (KFP) - machine learning worklow orchestration
- Cloud SQL - machine learning metadata  management
    
All required services are provisioned in the same [Google Cloud Project](https://cloud.google.com/storage/docs/projects). Before proceeding make sure that your account has access to the project and is assigned to the **Owner** or **Editor** role.

## Enabling the required cloud services

The following GCP Cloud APIs  must be enabled in your project:
1. Compute Engine
1. Cloud Storage
1. Container Registry
1. BigQuery
1. Cloud Machine Learning Engine
1. IAM
1. Dataflow
1. Kubernetes Engine
1. Cloud SQL
1. Cloud SQL Admin
1. Cloud Build
1. Cloud Resource Manager

Use [GCP Console](https://console.cloud.google.com/) or `gcloud` command line interface in [Cloud Shell](https://cloud.google.com/shell/docs/) to [enable the required services](https://cloud.google.com/service-usage/docs/enable-disable) . 

You can use the `enable_apis.sh` script to enable the required services from **Cloud Shell**.
```
./enable.sh
```

*Make sure that the Cloud Build service account (that was created when you enabled the Cloud Build service) is granted the Kubernetes Engine Developer role.*

## Provisioning an AI Platform Notebook instance
You use an **AI Platform Notebooks** instance based on a custom container as your primary development environment. 

The process of creating the custom container image has been automated with  [Cloud Build](https://cloud.google.com/cloud-build/). To build the image and push it to your project's **Container Registry** run the `./build.sh` script from **Cloud Shell**.

After the build completes, follow the  [instructions in AI Platform Notebooks Documentation](https://cloud.google.com/ai-platform/notebooks/docs/custom-container) to create an **AI Platform Notebook** instance. In the **Docker container image** field, enter the following image name: `gcr.io/[YOUR_PROJECT_NAME/tfx-kfp-dev:latest`.




## Deploying Kubeflow Pipelines 

The below diagrame shows an MVP infrastructure required to support a lightweight deployment of Kubeflow Pipelines:

![KFP Deployment](/images/kfp.png)

- A VPC to host GKE cluster
- A GKE cluster to host KFP services
- A Cloud SQL managed MySQL instance to host KFP and ML Metadata databases
- A Cloud Storage bucket to host artifact repository
- GKE and KFP service accounts. The GKE service account is used by GKE nodes. The KFP service account is used by KFP pipelines.

The provisioning of the MVP infrastructure and installation of Kubeflow Pipelines has been automated with Terraform. The Terraform HCL configurations can be found in the `terraform` folder.

To deploy Kubeflow Pipelines:

1. Open **Cloud Shell**
1. Navigate to the `terraform` folder
1. Configure your environment settings. You have two options. You can modify the settings in the `terraform.tfvars` file or you can provide the settings when applying the configuration using the `terraform apply -var '[NAME1]=[VALUE1]' -var '[NAME2]=[VALUE2]' ` format. The following settings must be configured:
    - `project_id` - your project ID
    - `region` - the region for a Cloud SQL instance
    - `zone` - the zone for a GKE cluster
    - `name_prefix` - the name prefix that will be added to the names of provisioned resources
    - `sql_username` - the name of a Cloud SQL user
    - `sql_password` - the password of a Cloud SQL user
1. From the terraform folder execute
```
terraform init 
terraform apply 
```


## Accessing KFP UI

After the installation completes, you can access the KFP UI from the following URL. You may need to wait a few minutes before the URL is operational.

```
echo "https://"$(kubectl describe configmap inverse-proxy-config -n kubeflow | grep "googleusercontent.com")
```
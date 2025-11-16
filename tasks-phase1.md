IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   group 1

   https://github.com/xcomerni/tbd-workshop-1
   
2. Follow all steps in README.md.

3. From avaialble Github Actions select and run destroy on main branch.
   
4. Create new git branch and:
    1. Modify tasks-phase1.md file.
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. 
    
    ***place the screenshot from GA after succesfull application of release***
   <img width="1081" height="554" alt="image" src="https://github.com/user-attachments/assets/36e6948f-bab1-42e9-82c4-cf6e53c4c5ed" />


6. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    ***describe one selected module and put the output of terraform graph for this module here***

The Composer module provisions a fully managed Cloud Composer (Apache Airflow) environment on Google Cloud that enables you to create, schedule, monitor, and manage workflow pipelines (DAGs) with no infrastructure-management overhead. It automatically creates a GKE cluster for Airflow components (schedulers, workers, triggerers), a Cloud SQL database for Airflow metadata, and an environment-specific Cloud Storage bucket that stores DAGs, plugins, logs, and data. The module configures associated service accounts, IAM roles, networking (including VPC/subnet integration and optional Private IP setups), and monitoring/logging integrations. By using this module, you get a production-ready Airflow environment including compute, storage, networking and orchestration, fully managed and integrated into your Google Cloud project.

**variables:**

env_name, project_name, region, network, subnet_address, subnet_name, image_version, env_size, env_variables
  
**outputs:**
```
output "gcs_bucket" {
  description = "GCS bucket for storing Apache Airflow DAGs"
  value       = module.composer.gcs_bucket
}

output "data_service_account" {
  description = "Apache Airflow service account"
  value       = google_service_account.tbd-composer-sa.email
}

output "gke_cluster" {
  description = "Composer underlying GKE cluster"
  value       = module.composer.gke_cluster
}
```
**graph generated from modules/composer:**
   <img width="2436" height="291" alt="composer-graph" src="https://github.com/user-attachments/assets/c3a6a3f0-818b-4d54-a981-c41c89e44db6" />

   
8. Reach YARN UI
   
   ***place the command you used for setting up the tunnel, the port and the screenshot of YARN UI here***
```
gcloud compute ssh tbd-cluster-m \
  --project=tbd-2025z-318326 \
  --zone=europe-west1-d \
  -- -L 8088:localhost:8088
```
   <img width="1913" height="581" alt="image" src="https://github.com/user-attachments/assets/ae56de32-43c2-4e50-a240-553dbdfb140f" />

   
10. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. Description of the components of service accounts
    2. List of buckets for disposal

        1. tbd-2025z-318326-lab@tbd-2025z-318326.iam.gserviceaccount.com
        
        Lab / workshop service account.
        
        2. tbd-2025z-318326-data@tbd-2025z-318326.iam.gserviceaccount.com
        
        Data pipeline / DAG service account. Used for accessing buckets like: tbd-2025z-318326-data, tbd-2025z-318326-code
        
        3. 1094786992778-compute@developer.gserviceaccount.com
        
        Default Compute Engine SA. GCP creates this automatically. Used when any VM or Dataproc node does not specify a custom service account.
        
        4. tbd-2025z-318326-dataproc-sa@tbd-2025z-318326.iam.gserviceaccount.com
        
        Dataproc cluster service account. Has permissions to: read/write Dataproc staging bucket, read/write Dataproc temp bucket, interact with BigQuery, read data bucket
        
        Buckets are:
        
        tbd-2025z-318326-code
        
        tbd-2025z-318326-data
        
        tbd-2025z-318326-dataproc-staging
        
        tbd-2025z-318326-dataproc-temp
        
        tbd-2025z-318326-state
    
        ***place your diagram here***

       <img width="634" height="623" alt="image" src="https://github.com/user-attachments/assets/75f052f9-1ff7-4c54-92cf-5842c2b34a71" />


10. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

   ***place the expected consumption you entered here***
```
resource_usage:

  module.gcr.google_artifact_registry_repository.registry:
    storage_gb: 50
    monthly_egress_data_transfer_gb:
      europe_west1: 50

  "module.data-pipelines.google_storage_bucket.tbd-data-bucket":
    storage_gb: 100
    monthly_class_a_operations: 10000
    monthly_class_b_operations: 20000
    monthly_data_retrieval_gb: 100
    monthly_egress_data_transfer_gb:
      same_continent: 100
      worldwide: 0
      asia: 0
      china: 0
      australia: 0

  "module.data-pipelines.google_storage_bucket.tbd-code-bucket":
    storage_gb: 20
    monthly_class_a_operations: 2000
    monthly_class_b_operations: 5000
    monthly_data_retrieval_gb: 20
    monthly_egress_data_transfer_gb:
      same_continent: 20
      worldwide: 0
      asia: 0
      china: 0
      australia: 0

  module.dataproc.google_storage_bucket.dataproc_staging:
    storage_gb: 50
    monthly_class_a_operations: 3000
    monthly_class_b_operations: 7000
    monthly_data_retrieval_gb: 30
    monthly_egress_data_transfer_gb:
      same_continent: 30
      worldwide: 0
      asia: 0
      china: 0
      australia: 0

  module.dataproc.google_storage_bucket.dataproc_temp:
    storage_gb: 20
    monthly_class_a_operations: 1000
    monthly_class_b_operations: 3000
    monthly_data_retrieval_gb: 10
    monthly_egress_data_transfer_gb:
      same_continent: 10
      worldwide: 0
      asia: 0
      china: 0
      australia: 0

  google_service_networking_connection.my_connection:
    monthly_egress_data_transfer_gb:
      same_region: 0
      us_or_canada: 0
      europe: 15
      asia: 0
      south_america: 0
      oceania: 0
      worldwide: 0
```

   ***place the screenshot from infracost output here***

   I tried few different Infracost files so thats why the sum is 12$ instead of 3$
<img width="678" height="135" alt="image" src="https://github.com/user-attachments/assets/2065f3de-d5db-415d-8e77-9fb6a43e1ba4" />

11. Create a BigQuery dataset and an external table using SQL
    
    ***place the code and output here***
```
    CREATE SCHEMA `tbd-2025z-318326.tbd_zmp_dataset`
    OPTIONS (
      location = "europe-west1"
    );
```

<img width="647" height="134" alt="image" src="https://github.com/user-attachments/assets/9e7551e5-11e9-4fa0-b861-54e7600f68cc" />

    
```
CREATE OR REPLACE EXTERNAL TABLE `tbd-2025z-318326.tbd_zmp_dataset.external_orc_table`
OPTIONS (
  format = 'ORC',
  uris = ['gs://cloud-samples-data/bigquery/us-states/us-states.orc']
);
```
   <img width="623" height="216" alt="image" src="https://github.com/user-attachments/assets/09cc5ff1-c18c-4cd7-9ecc-100074cf4ee1" />

    ***why does ORC not require a table schema?***

ORC files store the full schema (column names, types, metadata) inside the file footer.
Because the schema is embedded in the ORC file itself, BigQuery can automatically detect it and does not require you to define the table schema manually when creating an external table.

13. Find and correct the error in spark-job.py

    ***describe the cause and how to find the error***
I went to Dataproc → Jobs → Job that failed → Investigated job details and found:

```
py4j.protocol.Py4JJavaError: An error occurred while calling o96.orc.
: com.google.cloud.hadoop.repackaged.gcs.com.google.api.client.googleapis.json.GoogleJsonResponseException: 404 Not Found
POST https://storage.googleapis.com/upload/storage/v1/b/tbd-2025z-9901-data/o?ifGenerationMatch=0&uploadType=multipart
{
  "code": 404,
  "errors": [
    {
      "domain": "global",
      "message": "The specified bucket does not exist.",
      "reason": "notFound"
    }
  ],
  "message": "The specified bucket does not exist."
}
```

Changed data bucket to:

```
DATA_BUCKET = "gs://tbd-2025z-318326-data/data/shakespeare/"
```

rerun job by cloning it and now it worked:

<img width="1230" height="243" alt="image" src="https://github.com/user-attachments/assets/342c5281-3d6b-4ac3-99a1-1992bdd950dc" />


14. Add support for preemptible/spot instances in a Dataproc cluster

    ***place the link to the modified file and inserted terraform code***

https://github.com/xcomerni/tbd-workshop-1/blob/workers/modules/dataproc/main.tf

```
    preemptible_worker_config {
      num_instances  = 3
      preemptibility = "SPOT"

      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
      }

      instance_flexibility_policy {
        instance_selection_list {
          machine_types = [var.machine_type]
          rank          = 1
        }
      }
    }

```
    
15. Triggered Terraform Destroy on Schedule or After PR Merge. Goal: make sure we never forget to clean up resources and burn money.

Add a new GitHub Actions workflow that:
  1. runs terraform destroy -auto-approve
  2. triggers automatically:
   
   a) on a fixed schedule (e.g. every day at 20:00 UTC)
   
   b) when a PR is merged to main containing [CLEANUP] tag in title

Steps:
  1. Create file .github/workflows/auto-destroy.yml
  2. Configure it to authenticate and destroy Terraform resources
  3. Test the trigger (schedule or cleanup-tagged PR)
     
***paste workflow YAML here***
```
name: Auto Terraform Destroy

permissions:
  contents: read
  
on:
  schedule:
    - cron: "0 20 * * *"    # run every day at 20:00 UTC
  push:
    branches:
      - master

jobs:
  destroy:
    if: |
      github.event_name == 'schedule' ||
      (github.event_name == 'push' && contains(github.event.head_commit.message, '[CLEANUP]'))

    runs-on: ubuntu-latest

    permissions:
      contents: read
      id-token: write

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.11.0

      - id: "auth"
        name: "Authenticate to Google Cloud"
        uses: google-github-actions/auth@v1
        with:
          token_format: access_token
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER_NAME }}
          service_account: ${{ secrets.GCP_WORKLOAD_IDENTITY_SA_EMAIL }}

      - name: Init Terraform
        run: terraform init -backend-config=env/backend.tfvars

      - name: Terraform Destroy
        run: terraform destroy -auto-approve -var-file env/project.tfvars
```
***paste screenshot/log snippet confirming the auto-destroy ran***

<img width="915" height="721" alt="image" src="https://github.com/user-attachments/assets/b7b38311-fd1e-4d47-bfdc-8f937f88b949" />

<img width="470" height="212" alt="image" src="https://github.com/user-attachments/assets/c964aaf2-3d4d-4976-b0a1-064ab7b1b16a" />

***write one sentence why scheduling cleanup helps in this workshop***

Scheduled cleanup helps ensure no orphaned cloud resources remain running, preventing unnecessary costs during the workshop.

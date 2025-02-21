# Dataflow security and permissions 

### ✅ Running Dataflow Pipelines

-> Pipelines can run locally (for testing) or on Google Cloud (for scalability) using the Dataflow runner.

-> Dataflow VMs may need upgrades for better performance.

### ✅ Roles & Permissions

-> Dataflow uses IAM roles to control access.

-> Running pipelines requires specific roles and permissions.

-> Accessing pipeline resources (e.g., storage, Pub/Sub, BigQuery) needs proper IAM permissions.

### ✅ Data & Security in Dataflow

-> Handles structured & unstructured data.

-> Ensures data security through encryption, IAM roles, and access controls.

# Upgrade and patch Dataflow VMs

✅ Dataflow VMs run on Container-Optimized OS, which follows strict security processes.

### ✅ Batch Pipelines

-> Use the latest Dataflow image automatically when a new job starts.

-> No manual maintenance required.

### ✅ Streaming Pipelines

-> Google Cloud sends security alerts if immediate patches are needed.

-> Use --update to restart your job with the latest Dataflow image.
        
    📌 Batch jobs update automatically, while streaming jobs may require manual updates for security patches. 🚀

# Security & Permissions for Local Pipelines

### 1️⃣ How Local Pipelines Run

-> When you run an Apache Beam pipeline locally, it uses the Google Cloud account set up in the gcloud CLI.

-> This means your local system and Google Cloud account have access to the same files and resources.

### 2️⃣ How to Check Your Google Cloud Account

Run the command:

```gcloud config list```

-> This will show the Google Cloud account and project currently in use.

### 3️⃣ Where Local Pipelines Can Store Data

-> Locally: Save output files on your machine.

-> Cloud: Write data to Cloud Storage, BigQuery, etc.

### 4️⃣ Permissions for Cloud Storage

-> If writing data to Google Cloud resources, the pipeline uses:

    ->  Your Google Cloud account credentials

    -> The Google Cloud project set as default in gcloud CLI

### key points:

    ✔ Local pipelines use your Google Cloud account for authentication.
    
    ✔ They can store data locally or in Google Cloud.

    ✔ Writing to cloud resources requires authentication via the gcloud CLI. 🚀
# Security & Permissions for Dataflow Pipelines on Google Cloud

1️⃣ Service Accounts Used by Dataflow

### ✅ Dataflow Service Account (also called Dataflow Service Agent)

    -> Used during job creation to check project quota and create worker VMs.

    -> Also manages the job execution.

### ✅ Worker Service Account

    ->  Used by worker instances to access input/output resources.
  
    -> By default, it uses the Compute Engine default service account.

    -> Best practice: Use a user-managed service account instead of the default one.

### 2️⃣ Required IAM Roles & Permissions

🔹 To impersonate the service account, your account needs:

   ``` iam.serviceAccounts.actAs role.```
   
🔹 Additional permissions for running pipelines:

  ```roles/dataflow.developer``` (needed unless you’re a Project Owner/Editor).

### 3️⃣ Best Practices for Security

✅ Use a User-Managed Service Account for workers instead of the default one.

✅ Follow the Principle of Least Privilege

    -> Grant only the minimum required permissions for each task.

    -> Create custom roles with the exact permissions needed.

✅ Limit Scope of Role Assignments
        
    -> Instead of giving bigquery.dataEditor role at the project level, assign it at the table level.
   
✅ Use a Dedicated Staging Bucket

    -> Create a project-owned bucket for staging Dataflow jobs.

    -> The default bucket permissions allow Dataflow to use it automatically.

# Dataflow Service Account - Key Points

### 1️⃣ What is it?

-> A Google-managed service account used by Dataflow to manage resources like VMs.

### 2️⃣ Email Format:

``` service-PROJECT_NUMBER@dataflow-service-producer-prod.iam.gserviceaccount.com ```

### 3️⃣ How is it created?

-> It is automatically created when you first use a Dataflow Job in your project.

### 4️⃣ Role & Permissions:

-> Assigned the Dataflow Service Agent role `(roles/dataflow.serviceAgent)`

-> Has permissions to `start Compute Engine` workers and `manage Dataflow jobs`.

### 5️⃣ Do not modify its permissions!

-> Google Cloud services need full access to run Dataflow jobs.

-> Removing permissions may break Dataflow’s ability to create VMs and manage jobs.

### 6️⃣ How to check its roles?

-> Use the Google Cloud Console or run:

```
gcloud iam roles describe roles/dataflow.serviceAgent

```
### 7️⃣ If permissions are removed?

-> The service account remains in IAM, but Dataflow won't function properly without its required permissions.

# Worker Service Account - Key Points

### 1️⃣ What is it?

-> Used by Compute Engine worker VMs to run Apache Beam SDK operations in the cloud.

-> Acts as the identity for all worker VMs, allowing them to access required Google Cloud resources.

### 2️⃣ Permissions Required:

`roles/dataflow.worker` → To run jobs.

`roles/dataflow.admin` → To create or examine jobs.

-> Additional permissions required based on the resources accessed (e.g.,` roles/bigquery.dataEditor `for BigQuery).

### 3️⃣ Access to Google Cloud Resources:

        -> Cloud Storage buckets
        
        -> BigQuery datasets

        -> Pub/Sub topics and subscriptions

        -> Firestore datasets

### 4️⃣ Default Worker Service Account:

-> Uses the Compute Engine default service account:
```
PROJECT_NUMBER-compute@developer.gserviceaccount.com

```
-> This account is created automatically when you enable the Compute Engine API.

### 5️⃣ Best Practice: Use a Dedicated Worker Service Account

-> Instead of using the default Compute Engine account, create a custom service account with only the required permissions.

### 6️⃣ Security Considerations:

-> By default, the `Compute Engine service account` may have the `"Editor"` role, which grants too many permissions.

-> Disable automatic role grants using:

`iam.automaticIamGrantsForDefaultServiceAccounts` policy constraint.

### 7️⃣ Role Management & Least Privilege Principle:

-> If the default account has `Editor` role, replace it with `specific` roles.

-> Use Policy Simulator to analyze permission impact before making changes.

# User-Managed Worker Service Account - Key Steps

### 1️⃣ Create a User-Managed Service Account

-> Instead of using the default Compute Engine service account, create a dedicated service account for fine-grained access control.

### 2️⃣ Assign Required IAM Roles

`roles/dataflow.worker` → To run Dataflow jobs.

`roles/dataflow.admin` → To create or examine jobs.

-> Additional roles for accessing required resources (e.g., `roles/bigquery.dataViewer` for BigQuery).

### 3️⃣ Ensure Access to Staging & Temporary Locations

-> The user-managed service account must have `read/write` access to `staging and temporary` locations used by Dataflow jobs.

### 4️⃣ Grant Service Account Permissions

-> Allow the service account to be impersonated:`iam.serviceAccounts.actAs` permission.

-> Ensure Google-managed service accounts have these roles:

-> Service Account Token Creator (`iam.serviceAccountTokenCreator`)

-> Service Account User (`iam.serviceAccountUser`)

### 5️⃣ Cross-Project Usage Considerations

-> If the service account and job are in different projects, grant the necessary roles to allow cross-project service account usage.

-> Ensure `iam.disableCrossProjectServiceAccountUsage` constraint is not enforced.

### 6️⃣ Specify the Service Account in Your Pipeline Job

Command Line:

```
--serviceAccount=SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com
```
### For Flex Templates:

```
--service-account-email=SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com
```

### 7️⃣ Assign Roles via gcloud CLI

-> Grant Service Account User role to your user account:

```
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:EMAIL_ADDRESS" \
  --role=roles/iam.serviceAccountUser
```

-> Assign roles to the worker service account:

```
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role=SERVICE_ACCOUNT_ROLE

```
Replace placeholders:

        PROJECT_ID → Your project ID

        EMAIL_ADDRESS → Your user email

        PROJECT_NUMBER → Your project number
        
        SERVICE_ACCOUNT_ROLE → Required roles (e.g.,` roles/dataflow.worker`, `roles/dataflow.admin`)

# Accessing Google Cloud Resources for Apache Beam Pipelines

### 1. Resources That Require Access

        -> Artifact Registry (for storing container images)

        -> Cloud Storage (for reading/writing files)

        -> BigQuery (for querying datasets)
        
        -> Pub/Sub (for message streaming)
        
        -> Firestore (for structured data storage)
        
### 2. Granting Access to the Dataflow Worker Service Account

-> Use the Compute Engine default service account:

```
PROJECT_NUMBER-compute@developer.gserviceaccount.com
```
-> Or configure a user-managed service account.


# 3. Permissions Required for Each Resource

### Artifact Registry → `roles/artifactregistry.writer`
 

-> Grant Artifact Registry Writer role:

```
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/artifactregistry.writer"

```


### Cloud Storage → `roles/storage.objectViewer`, `roles/storage.objectCreator`

-> Grant Storage Object Viewer/Creator roles: (for reading/writing files)

```
gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/storage.objectCreator"
```
-> List existing buckets

```
gcloud storage buckets list --project=PROJECT_ID
```

### BigQuery → `roles/bigquery.dataEditor`, `roles/bigquery.jobUser`

-> Grant the following roles: (for reading/writing datasets)

        roles/bigquery.dataEditor (modify datasets)
        
        roles/bigquery.jobUser (run queries )

```

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/bigquery.jobUser"
```


### Pub/Sub → `roles/pubsub.subscriber`

-> Grant necessary roles based on use case: (for message streaming)

        roles/pubsub.subscriber (consume messages)
        
        roles/pubsub.editor (create subscriptions)
                
        roles/pubsub.viewer (query topic settings)

```
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/pubsub.subscriber"

```

### Firestore → `roles/datastore.viewer`

-> Grant Datastore Viewer role: (Datastore Mode)

```
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
  --role="roles/datastore.viewer"
```

-> Ensure Firestore API is enabled:


```
gcloud services enable firestore.googleapis.com
```

### 4. Additional Considerations

-> Assured Workloads: If using compliance controls (e.g., `EU regions`), all resources must be within an Assured Workloads project or folder.

-> Cross-Project Access: If accessing resources in other projects, grant permissions in both projects.

-> VPC Service Controls: Ensure that org policies or VPC security rules do not block access.


### 5. Best Practices

        ✅ Assign least privilege roles to service accounts.

        ✅ Use IAM roles instead of legacy ACLs.

        ✅ Verify access using: `gcloud projects get-iam-policy PROJECT_ID`

        ✅ Enable required Google Cloud APIs before running pipelines.

# Data Access and Security in Dataflow

### 1. Types of Data in Dataflow

End-User Data: Processed by the pipeline (e.g., input from sources, transformations, output to sinks).

Operational Data: Metadata required to manage the pipeline (e.g., job name, job ID, pipeline options).

### 2. Security Mechanisms Applied to:

        -> Pipeline submission

        -> Pipeline evaluation

        -> Access to logs, telemetry, and metrics

        -> Using Dataflow services (Shuffle, Streaming Engine)

### 3. Data Locality

-> Specify a region for Dataflow jobs to ensure data processing occurs in that region.

-> If no region is specified, `us-central1` is the default.

-> Data processing happens only in the specified region, even if sources/sinks are in different locations.

-> Worker VMs execute pipeline logic in specified zones.

### 4. Security in Different Data Stages

✅ Pipeline Submission

        -> Requires authentication via Google Cloud CLI.
        
        -> IAM roles (Editor/Owner) control access.
        
        -> Submitted via HTTPS for security.
        
✅ Pipeline Evaluation

        -> Temporary data may be stored in Cloud Storage or worker VMs.

        -> Temporary data is encrypted at rest and deleted after the job ends.

        -> Compute Engine VMs and attached Persistent Disks are deleted upon job completion.
        
        -> Intermediate data might be found in --stagingLocation or --tempLocation in Cloud Storage.

✅ Logs and Telemetry

        -> Stored in Cloud Logging.

        -> Dataflow adds only warnings and errors to logs.

        -> Telemetry and metrics are encrypted at rest and access is controlled via IAM.

✅ Data in Dataflow Services

        -> Dataflow Shuffle/Streaming Engine auto-selects the zone when the region is specified.

        -> Data in transit stays within worker VMs and remains in the pipeline’s specified region.

### 5. Best Practices

🔹 Use built-in security mechanisms of storage services (e.g., Cloud Storage, BigQuery).

🔹 Avoid mixing different trust levels in a single project.

🔹 Always specify a region to ensure optimal data processing and security.

🔹 Grant IAM roles with the least privilege needed for the pipeline.

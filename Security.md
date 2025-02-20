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

# 

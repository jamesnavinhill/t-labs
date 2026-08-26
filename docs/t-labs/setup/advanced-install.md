Cloud Storage
Where Does Transformer Lab Store Files
Transformer Lab runs as a central "coordinator" node, but dispatches workloads to different "worker" nodes. All of these nodes (workers and the coordinator) need to have common view of a shared storage directory. This can be stored in the cloud (usually recommended) but could also be on shared storage that is mounted to all nodes in common path (e.g. using NFS)

If you use our s3 or gcs storage option, Transformer Lab will mount the bucket automatically, you don't have to mount any drives yourself. But if you use our localfs storage engine, you map it to a directory that appears like a local path, but is mounted at the operating system level to a shared NFS or other storage engine.

Worker Node 3

Worker Node 2

Worker Node 1

Transformer Lab

Shared Storage

/shared_storage

/shared_storage

/shared_storage

/shared_storage

AWS S3 Storage
To use AWS S3 as remote storage:

Set TFL_REMOTE_STORAGE_ENABLED=true in your .env file.

Configure AWS credentials for the transformerlab-s3 profile.

note
If you already have an existing AWS profile you want to use instead of transformerlab-s3, add this line to your .env file:

AWS_PROFILE=<your-profile-name>

Using AWS CLI (Recommended)
If you have the AWS CLI installed, run:

aws configure --profile transformerlab-s3

Enter your AWS Access Key ID, Secret Access Key, default region, and output format when prompted.

Manual Configuration
Create or edit the AWS credentials file at ~/.aws/credentials and add:

[transformerlab-s3]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY

Ensure the profile has the necessary permissions to create and manage S3 buckets.

Google Cloud Storage (GCS)
To use Google Cloud Storage instead of AWS S3:

Set TFL_REMOTE_STORAGE_ENABLED=true in your .env file.

Set REMOTE_WORKSPACE_HOST=gcp in the same .env file.

Optionally, set GCP_PROJECT to specify the Google Cloud project. If not set, it defaults to transformerlab-workspace.

Configure Google Cloud credentials for the

local API server:

Using gcloud CLI (Recommended)
If you have the Google Cloud CLI installed, authenticate and set the project:

gcloud auth application-default login
gcloud config set project transformerlab-workspace  # or your project name


This writes credentials to ~/.config/gcloud/application_default_credentials.json, which the local API server picks up automatically.

Configure a service account key for remote job launches:

When Transformer Lab dispatches jobs to remote workers (RunPod, SkyPilot on non-GCP clouds, etc.), those machines run non-interactively and cannot complete a browser-based OAuth flow. Instead, Transformer Lab injects a service account key into the worker at launch time.

Create a Service Account Key
Go to Google Cloud Console → IAM & Admin → Service Accounts.
Create or select a service account with Storage Object Admin (or equivalent) permissions on your GCS bucket.
Generate a JSON key and download it to your server.
Configure Transformer Lab
During lab server init, you will be prompted for TFL_GCP_SERVICE_ACCOUNT_JSON_PATH — the path to the service account key file. Transformer Lab will store it and automatically inject the credentials into remote workers at job launch time.

warning
Without a service account key, remote job launches on non-GCP infrastructure will fail to access GCS storage.

Ensure the service account has the necessary permissions for Cloud Storage operations (Storage Admin or equivalent).

Azure Blob Storage
To use Azure Blob Storage instead of AWS S3 or GCS:

Set TFL_REMOTE_STORAGE_ENABLED=true in your .env file.

Set TFL_STORAGE_PROVIDER=azure in the same .env file.

Configure Azure credentials using one of the following approaches:

Option A: Connection String (Simplest)
Set the AZURE_STORAGE_CONNECTION_STRING environment variable in your .env file:

AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=your_account;AccountKey=your_key;EndpointSuffix=core.windows.net"


You can find your connection string in the Azure Portal under Storage account → Access keys.

Option B: Account Name + Key
Set the storage account name and access key separately:

AZURE_STORAGE_ACCOUNT="your_account_name"
AZURE_STORAGE_KEY="your_account_key"

Option C: Account Name + SAS Token
If you prefer to use a Shared Access Signature (SAS) token instead of the full account key:

AZURE_STORAGE_ACCOUNT="your_account_name"
AZURE_STORAGE_SAS_TOKEN="your_sas_token"

Ensure the SAS token has sufficient permissions for read, write, list, and delete operations on containers and blobs.

Local Storage
Instead of using a cloud provider like AWS or GCS, you can configure Transformer Lab to store all artifacts and job data locally. How you set this up depends on your architecture:

Single-Node Setup If your controller and workers run on the exact same machine, configuration is straightforward. You simply define a local file path, and both components will read and write to that exact same location.

Multi-Node Setup (Shared Filesystem) If your controller and workers run on separate machines, you must use a shared network filesystem (such as NFS). You must mount this shared folder to the exact same file path on every single machine. The system expects the TFL_STORAGE_URI to be identical across the controller and all workers so they can seamlessly share files.

Configuration Steps
To enable a local or shared filesystem, update your .env file with the following changes:

Set the storage provider: Add TFL_STORAGE_PROVIDER=localfs
Define the storage path: Add TFL_STORAGE_URI=/path/to/your/shared/folder
Disable remote storage: Delete the line TFL_REMOTE_STORAGE_ENABLED=true (if it is present).
Configure SkyPilot (if applicable): If you are running tasks with SkyPilot, you must configure hostPath volume mounts so your TFL_STORAGE_URI is accessible inside the task pods. See SkyPilot Volume Mounts for localfs for detailed instructions.
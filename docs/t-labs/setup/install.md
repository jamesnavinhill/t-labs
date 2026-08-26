Install Instructions
Prerequisites
Before starting the installation, ensure you have the following:

SSH access and administrative (sudo) privileges on the server hosting Transformer Lab
uv for installation
You can run tasks locally, or on a remote machine using a compute provider:
Install Slurm
Install SkyPilot
Install Runpod
Install dstack
Choosing Between Slurm and SkyPilot
You can store data locally, or use cloud storage (GCP, AWS, or Azure). You'll need an account with permissions to create and manage buckets/containers
Step 1 - Install Transformer Lab Using the CLI
1a. Install the Transformer Lab CLI
uv tool install transformerlab-cli

1b. Install the Server
Run the interactive installer:

lab server install

This will walk you through configuring:

Frontend URL — where users will access the web interface.
Storage Backend — choose between AWS S3, GCP, Azure, or local filesystem. See Cloud Storage Options for details on each provider.
Admin Account — a default admin account (admin@example.com / admin123) is created on first startup. Change the default password immediately after first login.
Compute Provider — optionally configure a default compute provider (you can also add or change providers later via the web UI or with lab provider add).
Email (SMTP) — optionally configure SMTP for sending user invitations and signup confirmations.
Authentication — optionally configure additional auth providers (OAuth/OIDC). Email/password is enabled by default.
The values you enter are saved to \~/.transformerlab/.env and used to configure the server on startup. If you had installed the server before and previously configured the setup, your existing values will be shown as defaults — press Enter to keep them.

If you want to run jobs on remote compute providers, you'll need a shared cloud storage. Configure credentials for your chosen cloud storage option before starting the server:

AWS S3 Setup
GCS Setup
Azure Setup
Note that storage configuration currently cannot be changed via the web UI.

Step 2 - Run Transformer Lab and Log in
Start the server:

cd \~/.transformerlab/src \&\& ./run.sh

Now visit http://localhost:8338 (or the address of your server) and log in with the default admin account:

Login: admin@example.com
Password: admin123
Change the default password immediately.

Step 3 - Configuring a Compute Provider
If you already configured a compute provider during the installer, you can skip this step. Otherwise, add one through the web UI or with lab provider add.

For provider-specific setup instructions and example configs, see:

SkyPilot
Slurm
Runpod
dstack
AWS
Azure
GCP



Update Instructions

To update Transformer Lab's server to the latest version, run the server update command in the cli:



lab server update



You can update the CLI with:



&#x20;uv tool upgrade transformerlab-cli


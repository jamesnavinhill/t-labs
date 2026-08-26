SkyPilot
After installing SkyPilot and starting Transformer Lab, follow these steps to add it as a compute provider.

Add SkyPilot provider

Add SkyPilot in Team Settings
Open Team Settings by clicking your username in the sidebar.
Go to Compute Providers.
Click Add Compute Provider.
In the modal:
Set Type to Skypilot.
Give the provider a name (e.g. skypilot-prov).
Fill in the Server URL, User ID, User name, Docker image (optional), Default region and Zone (optional) fields.
Click Add Compute Provider.
You can also add the provider via the CLI with lab provider add.

Run health check
After the provider is listed in Team Settings:

Find your SkyPilot provider in Compute Providers.
Click the "Check provider status" icon (heartbeat) next to your SkyPilot provider in the status column.
Confirm the provider reports healthy/connected.
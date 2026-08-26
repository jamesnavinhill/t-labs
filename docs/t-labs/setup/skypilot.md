SkyPilot Volume Mounts for localfs
When you use TFL\_STORAGE\_PROVIDER=localfs, Transformer Lab expects a shared network filesystem path (for example NFS) that is visible from the jobs it launches.

If your compute provider is SkyPilot, this means the pod/container started by SkyPilot must mount the same host path that your TFL\_STORAGE\_URI points to.

Why This Is Required
With localfs, data is not fetched from object storage. Your task reads and writes directly to a shared network volume that is mounted as a local filesystem path.

That path must be available in two places:

On the machine running your SkyPilot API/server (host path)
Inside SkyPilot task pods/containers (container mount path)
If these are not aligned, tasks can start successfully but fail when trying to access datasets, checkpoints, or artifacts.

Example Mount Path
In this guide, we use /shared\_storage as an example mount path for a shared network volume.

Then set:

TFL\_STORAGE\_PROVIDER=localfs
TFL\_STORAGE\_URI=/shared\_storage

Configure SkyPilot Host + Pod Mounts
Ensure the shared network volume is mounted on each Kubernetes worker node used by SkyPilot at a consistent host path (e.g. /shared\_storage).

Add a hostPath volume mount in \~/.sky/config.yaml on the SkyPilot head node.

kubernetes:
context\_configs:
default:
pod\_config:
spec:
containers:
- volumeMounts:
- name: tfl-shared-storage
mountPath: /shared\_storage
volumes:
- name: tfl-shared-storage
hostPath:
path: /shared\_storage
type: DirectoryOrCreate

Ensure your Transformer Lab .env uses the same container path.
TFL\_STORAGE\_PROVIDER=localfs
TFL\_STORAGE\_URI=/shared\_storage

Optional: Persist Hugging Face and uv Caches
For faster repeated runs, mount persistent caches so new SkyPilot pods reuse downloaded models and Python packages.

Create cache directories on each worker node.
Replace <LOCAL\_USERNAME> with your Linux username.

sudo mkdir -p /home/<LOCAL\_USERNAME>/huggingface\_cache
sudo mkdir -p /home/<LOCAL\_USERNAME>/uv\_cache
sudo chown -R 1000:1000 /home/<LOCAL\_USERNAME>/huggingface\_cache /home/<LOCAL\_USERNAME>/uv\_cache
sudo chmod -R 777 /home/<LOCAL\_USERNAME>/huggingface\_cache /home/<LOCAL\_USERNAME>/uv\_cache



Append cache env vars and mounts to \~/.sky/config.yaml:
kubernetes:
context\_configs:
default:
pod\_config:
spec:
containers:
- env:
- name: HF\_HOME
value: /home/sky/.cache/huggingface
- name: UV\_CACHE\_DIR
value: /home/sky/.cache/uv
volumeMounts:
- name: hf-vol
mountPath: /home/sky/.cache/huggingface
- name: uv-vol
mountPath: /home/sky/.cache/uv
volumes:
- name: hf-vol
hostPath:
path: /home/<LOCAL\_USERNAME>/huggingface\_cache
type: DirectoryOrCreate
- name: uv-vol
hostPath:
path: /home/<LOCAL\_USERNAME>/uv\_cache
type: DirectoryOrCreate

Validation Checklist
Before running tasks, verify all of the following:

TFL\_STORAGE\_PROVIDER=localfs is set in .env
TFL\_STORAGE\_URI points to the mounted container path (for this example: /shared\_storage)
The same storage is mounted on all nodes that can run tasks
\~/.sky/config.yaml has matching hostPath and mountPath values




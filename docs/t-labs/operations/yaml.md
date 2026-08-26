Task YAML Structure
This guide explains how to format YAML files for creating tasks in Transformer Lab. Tasks define jobs that run on compute providers and can include training scripts, evaluation scripts, or any other computational workloads.

Note: For detailed information about defining task parameters with validation and custom UI, see the Task Parameters guide.

Basic Structure
The basic structure of a task YAML file includes the following sections:

name: task-name
resources:
  compute_provider: provider-name-in-your-transformerlab-workspace
  cpus: 2
  memory: 4
minutes_requested: 60
envs:
  KEY: value
setup: 'command'
run: 'command'
github_repo_url: 'url'
github_repo_dir: 'dir'
parameters: { ... }
sweeps:
  sweep_config: { ... }
  sweep_metric: 'metric'
  lower_is_better: true

Required Fields
name
The task name. This will be sanitized to create a safe filename and name of the cluster on the compute provider.

Type: String

Example:

name: my-training-task

Resources Configuration
The resources section defines the compute resources required for the task.

resources.compute_provider
The name of the compute provider to use. This should match a configured provider name in your workspace.

Type: String

Example:

resources:
  compute_provider: skypilot-provider

Note: If not specified, the system will use the first available provider as a fallback.

resources.cpus
Number of CPUs to allocate.

Type: Integer or String

Example:

resources:
  cpus: 4

resources.memory
Amount of memory to allocate (in GB).

Type: Integer or String

Example:

resources:
  memory: 16

resources.disk_space
Amount of disk space to allocate (in GB).

Type: Integer or String

Example:

resources:
  disk_space: 100

resources.accelerators
Accelerator specification (e.g., GPU type and count). Format depends on the provider. To look at supported formats in Skypilot, refer to their accelerator documentation and for Slurm, refer to their GPU documentation.

Type: String

Example:

resources:
  accelerators: 'H100:8'

resources.num_nodes
Number of nodes for distributed training.

Type: Integer

Example:

resources:
  num_nodes: 2

Complete Resources Example:

resources:
  compute_provider: aws-ec2
  cpus: 8
  memory: 32
  disk_space: 200
  accelerators: '1xA100'
  num_nodes: 1

Commands
setup
Command(s) to run before the main task execution. This is typically used for installing dependencies, setting up the environment, or downloading data.

Type: String

Example:

setup: 'pip install -r requirements.txt'

Multi-line Setup:

setup: |
  pip install -r requirements.txt
  apt-get update
  apt-get install -y git
  python download_data.py

run
The main command to execute for the task. This is the primary script or command that performs the actual work.

Type: String

Example:

run: 'python train.py'

With Arguments:

run: 'python train.py --epochs 10 --batch-size 32'

Multi-line Run:

run: |
  python train.py \
    --epochs 10 \
    --batch-size 32 \
    --learning-rate 2e-5

Environment Variables
envs
Environment variables to set for the task execution. These are passed as key-value pairs.

Type: Dictionary (key-value pairs)

Example:

envs:
  CUDA_VISIBLE_DEVICES: '0'
  WANDB_API_KEY: 'your-api-key'
  HF_TOKEN: 'your-huggingface-token'

Quota Tracking
minutes_requested
Estimated number of minutes the task will run. This is used for quota tracking and resource allocation. When specified, a quota hold is created to reserve the estimated compute time.

Type: Integer

Example:

minutes_requested: 60

Note: This is an optional field but recommended for tasks running on remote compute providers to enable quota tracking and better resource management.

GitHub Integration
github_repo_url
GitHub repository URL to clone before running the task. The repository will be cloned to the working directory.

Type: String

Example:

github_repo_url: 'https://github.com/username/repo.git'

github_repo_dir
Subdirectory within the GitHub repository to use as the working directory. Useful when the repository contains multiple projects.

Type: String

Example:

github_repo_url: 'https://github.com/username/multi-project-repo.git'
github_repo_dir: 'project1'

github_repo_branch
Branch of the GitHub repository to clone. Defaults to the default branch if not specified. Type: String Example:

github_repo_url: 'https://github.com/username/multi-project-repo.git'
github_repo_dir: 'project1'
github_repo_branch: 'main'

Note: The final path where the cloned folder would be available is either: ~/github_repo_dir or ~/github_repo_name (if no directory is specified).

Complete GitHub Example:

github_repo_url: 'https://github.com/transformerlab/examples.git'
github_repo_dir: 'training/llm-finetuning'
setup: 'pip install -r requirements.txt'
run: 'python train.py'

Parameters
parameters
Task parameters (hyperparameters, configuration, etc.) that will be accessible via lab.get_config() in your scripts. These are passed to the job and can be used to configure the training or evaluation process.

Detailed documentation on this field is on its own page

Type: Dictionary (any JSON-serializable values)

Example:

parameters:
  model_name: 'gpt2'
  learning_rate: 2e-5
  batch_size: 8
  num_epochs: 3
  max_seq_length: 512
  warmup_ratio: 0.03
  weight_decay: 0.01

Nested Parameters:

parameters:
  model:
    name: 'gpt2'
    architecture: 'GPT2LMHeadModel'
  training:
    learning_rate: 2e-5
    batch_size: 8
    num_epochs: 3
  data:
    dataset_name: 'wikitext'
    max_seq_length: 512

Note: Parameters can be accessed in your Python scripts using the Lab SDK:

from lab import lab

lab.init()
config = lab.get_config()
learning_rate = config.get("learning_rate")
model_name = config.get("model_name")

📖 For comprehensive parameter documentation, including:

Parameter types (int, float, bool, enum, string, json, model, dataset)
Schema validation (min, max, multipleOf)
UI customization (ui_widget options)
Special model and dataset selectors
Complete examples
See the Task Parameters guide.

Hyperparameter Sweeps
sweeps
Configuration for hyperparameter sweeps. When sweeps are enabled, the system will generate multiple jobs, one for each combination of parameter values.

sweeps.sweep_config
Dictionary mapping parameter names to lists of values to try. The system will generate jobs for all combinations of these values.

Type: Dictionary (parameter name -> list of values)

Example:

sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
    batch_size: ['4', '8', '16']
    lora_rank: ['8', '16', '32']

sweeps.sweep_metric
The metric to optimize during the sweep. This should match a metric name that your script logs (e.g., via wandb or in evaluation results).

Type: String

Example:

sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
  sweep_metric: 'eval/loss'

Common Metrics:

"eval/loss" - Evaluation loss
"train/loss" - Training loss
"eval/accuracy" - Evaluation accuracy
"eval/f1_score" - F1 score
"eval/bleu" - BLEU score
sweeps.lower_is_better
Whether lower values of the sweep metric are better (True) or higher values are better (False).

Type: Boolean

Example:

sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
  sweep_metric: 'eval/loss'
  lower_is_better: true # Lower loss is better

or

sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
  sweep_metric: 'eval/accuracy'
  lower_is_better: false # Higher accuracy is better

Complete Sweeps Example:

name: hyperparameter-sweep
resources:
  compute_provider: aws-ec2
  cpus: 4
  memory: 16
  accelerators: '1xV100'
run: 'python train.py'
parameters:
  model_name: 'gpt2'
  dataset_name: 'wikitext'
sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
    batch_size: ['4', '8']
    lora_rank: ['8', '16']
  sweep_metric: 'eval/loss'
  lower_is_better: true

Complete Examples
Example 1: Simple Training Task
name: simple-training
resources:
  compute_provider: local
  cpus: 4
  memory: 8
minutes_requested: 30
setup: 'pip install transformers torch'
run: 'python train.py'
parameters:
  model_name: 'gpt2'
  learning_rate: 2e-5
  batch_size: 8
  num_epochs: 3

Example 2: Training Task with GitHub Repository
name: finetune-llm
resources:
  compute_provider: skypilot-provider
  cpus: 8
  memory: 32
  accelerators: 'H100:1'
minutes_requested: 120
github_repo_url: 'https://github.com/username/llm-training.git'
github_repo_dir: 'finetuning'
setup: |
  pip install -r requirements.txt
  pip install wandb
envs:
  WANDB_API_KEY: 'your-api-key'
  HF_TOKEN: 'your-huggingface-token'
run: 'python train.py'
parameters:
  model_name: 'meta-llama/Llama-2-7b-hf'
  dataset_name: 'wikitext-2'
  learning_rate: 2e-5
  batch_size: 4
  gradient_accumulation_steps: 8
  num_epochs: 3
  max_seq_length: 512
  warmup_ratio: 0.03
  weight_decay: 0.01

Example 3: Evaluation Task
name: evaluate-model
resources:
  compute_provider: local
  cpus: 2
  memory: 4
setup: 'pip install transformers datasets'
run: 'python evaluate.py'
parameters:
  model_name: 'gpt2'
  dataset_name: 'wikitext'
  batch_size: 16
  max_samples: 1000

Example 4: Hyperparameter Sweep
name: lora-sweep
resources:
  compute_provider: skypilot-provider
  cpus: 4
  memory: 16
  accelerators: 'H100:1'
minutes_requested: 180
github_repo_url: 'https://github.com/username/llm-training.git'
setup: |
  pip install -r requirements.txt
  pip install wandb
envs:
  WANDB_API_KEY: 'your-api-key'
run: 'python train.py'
parameters:
  model_name: 'gpt2'
  dataset_name: 'wikitext'
  num_epochs: 3
sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
    batch_size: ['4', '8']
    lora_rank: ['8', '16', '32']
    lora_alpha: ['16', '32', '64']
  sweep_metric: 'eval/loss'
  lower_is_better: true

Best Practices
Use Descriptive Names: Choose clear, descriptive task names that indicate what the task does.

name: finetune-gpt2-wikitext  # Good
name: task1                    # Bad

Specify Resources Appropriately: Match resources to your workload. Don't request more than you need, but ensure you have enough for the task.

# For small models
resources:
  cpus: 4
  memory: 8

# For large models
resources:
  cpus: 16
  memory: 64
  accelerators: "1xA100"

Use Setup for Dependencies: Install dependencies in the setup command rather than in the run command.

setup: 'pip install -r requirements.txt' # Good
run: 'python train.py'

Store Sensitive Data Securely: Don't hardcode API keys or tokens in YAML files. Use environment variables or secure configuration.

# Good - use environment variables
envs:
  WANDB_API_KEY: "${WANDB_API_KEY}"

# Bad - hardcoded
envs:
  WANDB_API_KEY: "abc123xyz"

Use Parameters for Configuration: Store hyperparameters and configuration in the parameters section so they're accessible via lab.get_config().

parameters:
  learning_rate: 2e-5
  batch_size: 8

Document Complex Sweeps: When using sweeps, document what you're optimizing and why.

sweeps:
  # Testing different learning rates and batch sizes
  sweep_config:
    learning_rate: ['1e-5', '3e-5', '5e-5']
    batch_size: ['4', '8']
  sweep_metric: 'eval/loss'
  lower_is_better: true

Use GitHub for Code: Store your code in a GitHub repository and reference it with github_repo_url, github_repo_dir, and github_repo_branch rather than uploading files manually.

github_repo_url: 'https://github.com/username/my-project.git'
github_repo_dir: 'training'
github_repo_branch: 'main'

Test Locally First: Test your task configuration locally before running on expensive cloud resources.

resources:
  compute_provider: local # Test locally first

Use Multi-line Strings for Long Commands: Use YAML's | or > syntax for multi-line commands.

setup: |
  pip install -r requirements.txt
  python download_data.py
  python preprocess_data.py

Validate YAML Syntax: Ensure your YAML is valid before submitting. Use a YAML validator or linter.

Common Issues and Solutions
Issue: YAML Parsing Errors
Problem: Invalid YAML syntax causes parsing errors.

Solution: Validate your YAML syntax. Common issues:

Missing colons after keys
Incorrect indentation (use spaces, not tabs)
Unquoted strings with special characters
Issue: Parameters Not Accessible
Problem: Parameters defined in YAML are not accessible via lab.get_config().

Solution: Ensure parameters are at the root level under parameters: key:

parameters:
  learning_rate: 2e-5 # Correct

Not:

config:
  parameters:
    learning_rate: 2e-5 # Wrong

Issue: Sweeps Not Running
Problem: Sweeps are defined but not generating multiple jobs.

Solution: Ensure the sweeps section includes all required fields:

sweeps:
  sweep_config:
    learning_rate: ['1e-5', '3e-5']
  sweep_metric: 'eval/loss' # Required
  lower_is_better: true # Required

Issue: Provider Not Found
Problem: compute_provider name doesn't match any configured provider.

Solution: Check the exact provider name in your workspace. The system will use the first available provider as a fallback, but it's better to specify the correct name.
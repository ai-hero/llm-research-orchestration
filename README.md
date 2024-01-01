# Orchestration for LLM Research by AI Hero

Code used to orchestrate the fine-tune and serving services on top of Kubernetes. 

## Setup

### Kubernetes
Currently we are only supporting Coreweave.
- `Coreweave` - We assume that you have downloaded the `kubeconfig` file and can see your pods using `kubectl get pods`.
  
### Requirements
```sh
pip install -r requirements.txt
pip install -r k8s/requirements.txt
pip install -r poc/requirements.txt
```

### Llama 2
Make sure you've signed the T&Cs to be able to access the llama-2 weights on Huggingface.
 
### Environment
You'll also need a `.env` file in the `k8s` folder from where you'll run the jobs.
```
# For reporting experiments to W&B
WANDB_API_KEY=   
WANDB_USERNAME=

# For loading/saving data/models to Huggingface
HF_TOKEN=

# For downloading data to from S3
S3_ENDPOINT=s3.amazonaws.com
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
S3_REGION=us-east-2
S3_SECURE=true
```


## Launching a Fine-Tuning Job
```sh
cd k8s/
```

### Update the Config for the Job
Update [k8s/yamls/config.yaml](k8s/yamls/config.yaml) as needed for your job.

### Launch the Job
```sh
cd k8s/
python train.py launch rparundekar/fine_tune_research:20231209_01 mmlu_peft.yaml
```
You'll see the name of the job. And instructions to see the logs and delete the job.
NOTE: The container is created in the fine-tuning project.

If launching with a distributed config
```sh
cd k8s/
python train.py launch rparundekar/fine_tune_research:20231209_01 distributed_default.yaml -d fsdp_single_worker.yaml
```

#### Deleting a Job
Use the same program to delete the job so that all resources are deleted.
```sh
python train.py delete <job-name>
```

## Serving a Trained Model
We'll use the Huggingface TGI library to serve a model. The program will read details of the output model from the training config and serve in Kubernetes.
Then, you can port forward locally and run the PoC
```sh
cd k8s/
python serve.py launch mmlu_peft.yaml
```

### Running the PoC
The PoC assumes that the model server is running at `http://127.0.0.1:8080/`.

```sh
streamlit run poc.py
```

## Code QA
```sh
python3.9 -m venv venv
source venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
pre-commit install
pre-commit autoupdate
pre-commit run --all-files
```

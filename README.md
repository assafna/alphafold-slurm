# AlphaFold on Slurm

## Introduction

General walktrough for inferencing a sequence using AlphaFold through Slurm and on GPUs. The guide uses [DeepMind's AlphaFold](https://github.com/deepmind/alphafold) recent version.

## First Time Setup

Follow DeepMind's [instructions](https://github.com/deepmind/alphafold#first-time-setup) and download the relevant databases. You can use a general container to download the databases through it.

## Prepare a Container

Slurm uses a [squash](https://en.wikipedia.org/wiki/SquashFS) file to run. Therefore, a container should be created and converted to a squash file first.

1. Git clone AlphaFold's GitHub repository.

    ```bash
    git clone https://github.com/deepmind/alphafold.git
    ```

2. __Optional:__ modify the Dockerfile to include the latest versions.

    Replace [`docker/Dockerfile`](https://github.com/deepmind/alphafold/blob/main/docker/Dockerfile) with the provided modified [Dockerfile](Dockerfile).

    __Note:__ this modified Dockerfile assumes you have CUDA version 11.4 installed with Ubuntu 20.04. If this is not the case, please change the relevant lines.

3. Build a Docker image.

    ```bash
    docker build -f docker/Dockerfile -t alphafold .
    ```

4. Convert the Docker image into a Squash file using [NVIDIA Enroot](https://github.com/NVIDIA/enroot).

    ```bash
    enroot import 'dockerd://alphafold:latest'
    ```

    This will create a local squash file.

## Prepare a Slurm SBATCH Script File

DeepMind's instructions of running AlphaFold includes running [`run_docker.py`](https://github.com/deepmind/alphafold/blob/main/docker/run_docker.py) file from outside of the container, and to have a Python environment for that. To overcome this, the file was converted to the provided Slurm SBATCH [`alphafold.sub`](alphafold.sub) script file.

Change the relevant paths and configuration in the file to fit your settings:

- `CONTAINER_PATH` - path to the container squash file.
- `DATA_DIR` - path to the directory which includes all the downloaded databases.
- `OUTPUT_DIR` - path to the directory where the inferencing outputs will be. __Note:__ a subfolder will be created for every new Slurm job based on the job ID.
- `FASTA_DIR` - path to the directory which includes the FASTA files.
- `FASTA_PATH` - name of the FASTA file to inference.
- `FINAL_RUN_OPTIONS` - modify this argument for using a different model and database presets.

__Note:__ the script assumes a single FASTA file is inferenced at a time, AlphaFold supports inferencing more than a single FASTA file but _NOT_ in parallel, and _NOT_ in multi-GPU manner. Therefore, it is recommended to inference a single FASTA file in a single Slurm job, with a single GPU.

## Submit a Slurm job

```bash
sbatch alphafold.sub
```

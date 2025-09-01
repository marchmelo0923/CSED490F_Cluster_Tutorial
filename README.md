# [W1] Simple tutorial for CSE Edu Cluster

Let we start with very simple example for CSE Edu Cluster.

## What is Slurm?

Slurm is the job scheduler for GPU server.
When you submit your jobs to Slurm, it automatically calculate your job priority.
The higher priority get the node first, and lower will be pending until higher one is over.
Your priority is caculated by
1. How much GPU you use for this job
2. How many resources you use in recent few times.

## How to use Slurm?

### Let's see how our slurm script work first!

Slurm get options from top of shell files notation. Like below.

```
#!/bin/sh

#SBATCH -J  CSED490F_W1           # Job name
#SBATCH -o  slurm_logs/%j.out    # Name of stdout output file (%j expands to %jobId)
```

-J: Slurm job name
-o: log with any info, error.

---
After that, you should set your parition information
```
#SBATCH -p titanxp       # queue  name  or  partiton
#SBATCH   --gres=gpu:1          # gpus per node
```

-p: Set node partition. You can check the partition of your cluster with "sinfo" command.
--gres=gpu:N : Set number of gpu. 

CSE Edu Cluster only have "titanxp" partition.

---
Now, you sholud set the node info
```
#SBATCH   --nodes=1              # the number of nodes 
#SBATCH   --ntasks-per-node=1
#SBATCH   --cpus-per-task=4
```

--nodes=N : Set number of nodes. It is different with 'Number of GPU.' In general cases, you will use N=1.
--ntasks-per-node=N : How many times your node execute the scripts.
--cpus-per-task=N : How many CPU resources you want.

### Docker setup
We already give you all things for docker setup, but let me explain you how our scripts work!

First at all, out cluster server have very important rule
**"Include your POVIS ID in your docker name"** and
**"Set GPU device with UUID, when you use docker"**

So, You should set your personal information for every assignment.
```
YOUR_POVIS_ID="{your_name}" # ex: honggildong
YOUR_HOME_PATH="{your_home_path}" # ex: /home/honggildong
```

For UUID, we already contain the automatic UUID finding scripts
```
echo "[System] UUID GPU List"
UUIDLIST=$(nvidia-smi -L | cut -d '(' -f 2 | awk '{print$2}' | tr -d ")" | paste -s -d, -)
GPULIST=\"device=${UUIDLIST}\"
```

Now, it create the docker container and mount your directory as **/HOME**
```
docker run -itd --gpus ${GPULIST} --name ${DOCKER_NAME} --ipc=host --shm-size=8G \
    -v ${DIRECTORY_PATH}:/HOME \
    ${DOCKER_IMAGE}
```

After the docker container creation, you can execute command like below.
```
docker exec ${DOCKER_NAME} bash -c "cd /HOME && bash scripts/execute_hello_world.sh"
```

When your job is all finished, you should cleanup your docker container
```
## Stop docker container
docker stop ${DOCKER_NAME}

## Remove docker container
docker rm -f ${DOCKER_NAME}
```

## Start

Now, let we start this simple example!

### 1. Clone this repository on your cluster
```
git clone https://github.com/marchmelo0923/CSED490F_Cluster_Tutorial.git
```

### 2. Create folder for logs (In future assignment, we will give you initilization scripts. Mind it only for this example!)
```
cd CSED490F_Cluster_Tutorial
mkdir -m 777 slurm_logs
```

### 3. Set your POVIS ID and cluster directory
```
# open your scripts file with "vi rum_hello_world.sh"
YOUR_POVIS_ID="{your_name}" # ex: honggildong
YOUR_HOME_PATH="{your_home_path}" # ex: /home/honggildong
```

### 4. Submit the script to Slurm
```
sbatch run_hello_world.sh
```

Now you can see your job's log with
```
tail -f slurm_logs/{your_job_id}.out
or
vi slurm_logs/{your_job_id}.out
```

If you want to see your job state (or other person's one)
```
scontrol show job {your_job_id} # It will write your job's state (PENDING or RUNNING, etc)
or
squeue # It will show you all the jobs in Slurm
```

If you want to cancle your RUNNING/PENDING job,
```
scancel {your_job_id}
```

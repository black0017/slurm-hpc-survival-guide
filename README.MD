

# Survival guide for HPC system of HHU

$Authors: Nikolas Adaloglou, Julius Ramarkes$

For the whole guide use your HHU university login name + password. For access to the computers you need to enable openvpn:

```
$ sudo openvpn HHU-VPN.ovpn
```



## Start here


1. Fill out the [form](https://www.zim.hhu.de/forschung/high-performance-computing/antrag) to get access to the computer

2. On a Linux terminal run `$ssh <username>@hpc.rz.uni-duesseldorf.de` . In my case`$ssh adaloglo@hpc.rz.uni-duesseldorf.de`

3. See all the install software/libraries like CUDA versions and python libraries (Tensorflow, Pytorch) with `$ module avail`

4. Modify the bashrc to load the modules you need 

   ``` $ nano ~/.bashrc``` 

   Note that ''#" is used to comment out i.e. if you need to switch between TF versions

   ```
   ## comments start with # like: # Source global definitions
   if [ -f /etc/bashrc ]; then
           . /etc/bashrc
   fi
   if env | grep -q ^JPY_API_TOKEN=
   then
       export MODULES_FOR_JUPYTER=1
       #module load TensorFlow/1.14
       #module load TensorFlow/2.4.0
       module load PyTorch/1.8.0
       #module load Horovod
       module load APEX
       #module load BioPython
       # add you modules here!!!!!
   fi
   ```

You can load modules directly in the terminal like `module load PyTorch/1.8.0` . Then if you type `$  python` you should be able to `import torch`

5. Find you project folder  `$ cd /gpfs/project/<username>` or   `$ cd /gpfs/project/projects/<yourproject>` .

   For example in my case   `$ cd /gpfs/project/adaloglo/ && ls`

6. For savings (checkpoints, statistics, figures etc) use:  `$ /gpfs/scratch/<username>/... `

   

Then, you will need to transfer your code + data in this folder. Before that, take a look at the resource managment:

## General purpose stuff - resource management

One way to allocate resources from the [Jupyter hub of HPC](https://jupyter.hpc.rz.uni-duesseldorf.de/hub/spawn) but it's not recommended.

SLUMR systems work with job launching. 

   - You can see the running jobs `qstat`
   - To see your jobs `qstat -u <yourloginname>`
   - You can cancel your job with `qdel <jobid>`

Then you need to find your subserver i.e. BioJob queue

For instance our BioJob servers are three systems each having 10 GPU's `hilbert302, hilbert309 m hilbert310` are their names. 

- To see the available resources per server type: $ `pbsnodes <servername>` like: `pbsnodes hilbert310`

Outputs:

```hilbert310
     Mom = hilbert310.hilbert.hpc.uni-duesseldorf.de
     ntype = PBS
     state = free
     pcpus = 20
     resources_available.accelerator_model = gtx1080ti
     resources_available.arch = broadwell
     resources_available.host = hilbert310
     resources_available.hpmem = 0b
     resources_available.ibswitch = 0xe41d2d0300450000,all
     resources_available.mem = 256648mb
     resources_available.ncpus = 20
     resources_available.ngpus = 10
     resources_available.Qlist = cuda
     resources_available.vmem = 256648mb
     resources_available.vnode = hilbert310
     resources_assigned.accelerator_memory = 0kb
     resources_assigned.hbmem = 0kb
     resources_assigned.mem = 0kb
     resources_assigned.naccelerators = 0
     resources_assigned.ncpus = 0
     resources_assigned.ngpus = 0
     resources_assigned.vmem = 0kb
     resv_enable = True
     sharing = default_shared
     license = l
     last_state_change_time = Mon Aug 16 10:48:18 2021
     last_used_time = Thu Aug  5 04:09:10 2021``` 
```

As you can see all 10 GPU's are free right now!


## Data transfer using scp

For extensive file transfer it is recommended to use [storage.hpc.rz.uni-duesseldorf.de](http://storage.hpc.rz.uni-duesseldorf.de/), you can copy and paste using scp:

#### Send files using scp

From your local terminal:

`$ scp file.zip adaloglo@storage.hpc.rz.uni-duesseldorf.de:/gpfs/project/<username>/<project_folder>`

#### Receive files using scp

`$ scp adaloglo@storage.hpc.rz.uni-duesseldorf.de:/gpfs/project/<username>/<project_folder>/<file.zip>  ./ `



Alternative methods are Filezilla for Linux.


- German [guide](https://wiki.hhu.de/display/HPC/Filesysteme+mounten)


## Launch jobs

Given everything else is set up you now need to specify a `run_my_code.sh` file for `$ qsub`

`$ qsub run_my_code.sh`

Below is an example of `run_my_code.sh`

```
#!/bin/bash 
#PBS -l select=1:ncpus=4:mem=10gb:ngpus=2:accelerator_model=gtx1080ti # or a100 
#PBS -l walltime=167:59:00 
#PBS -A "DeepGenome" # modify accordingly
#PBS -r n 
#PBS -q "BioJob" # specify your server if you know where to launch your script
#PBS -m e 
#PBS -M <username>@uni-duesseldorf.de 
 
set -e 

module load CUDA/10.0.130 
module load Python/3.6.5 
module load TensorFlow/1.14 
 
python /gpfs/project/<username>/<project_folder>/main.py # modify
```



## Launch interactive sessions on HPC to test your code 

 In the ssh HPC login shell you can also **interactively** ask for more resources, e.g. 2CPUs and 5GB would be: 

```qsub -A DeepGenome -I -l select=1:ncpus=2:mem=5G -l walltime=12:00:00```


## Install your Python packages locally (TODO test)

- Here is a list of **trusted** [python packages](http://pypi.repo.test.hhu.de/simple/) that can be installed. If the package that you need is not available you need to contact the HPC team at `hpc-support [at] uni-duesseldorf.de `

You can **locally** install the packages like this:

```
pip install --upgrade --target=<your-project/user-folder>/site-packages -i http://pypi.repo.test.hhu.de/simple/ --trusted-host pypi.repo.test.hhu.de <your-awesome-package>
```
Here is an example to use  `pytorch-lightning`
```
module load module load PyTorch/1.8.0
pip install --upgrade --user  --target=/gpfs/project/adaloglo/site-packages  -i http://pypi.repo.test.hhu.de/simple/ --trusted-host pypi.repo.test.hhu.de pytorch-lightning
```

#### Install requirements.txt locally (TODO)

...





## NOT for HPC: Develop interactively with Jupyter server (todo test)

ssh `<username>@servername` like: `ssh adaloglo@134.99.222.53`

#### Remote - server side 


- create virtual environment `virtualenv --python=python3.6 ~/test_env` or `virtualenv-3  --python=python3.6 ~/test_env`

- load virtualenv `$ source ~/test_env/bin/activate`

- install project requirements, including jupyter `pip install -r requirements.txt`

- start jupyter server:    

  ```  jupyter notebook --no-browser --port=8916 >~/jupyter_lab_output_8916_$(date +"%Y%m%d").txt 2>&1 &   ```

- Look into the newly generated text file -> web URL for local browser.
  
- To access the notebook, open this file in a browser:
     file:///home/adaloglo/.local/share/jupyter/runtime/nbserver-183700-open.html
   Or copy and paste one of the shown URLs.
  
  

#### Local/client side:
- Port forwarding, here e.g. for port 8916:  `ssh -L 8916:localhost:8916 <username>@<server_id>`
- Local browser URL open: http://127.0.0.1:8916/?token=85fdfa86bf95d2bbd0e4e42af536de1126c55e8bf66e398b
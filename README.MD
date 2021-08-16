# Survival guide for HPC system of HHU

For the whole guide use your HHU university login name + pass

## Start here

1. Fill out the [form](https://www.zim.hhu.de/forschung/high-performance-computing/antrag) to get access to the computer

2. On a Linux terminal run`$ ssh adaloglo@hpc.rz.uni-duesseldorf.de`

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

5. Find you project folder  `$ cd /gpfs/project/<username>` or   `$ cd /gpfs/project/projects/<yourproject>` .

   For example in my case   `$ cd /gpfs/project/adaloglo/ && ls`

6. Then you will need to transfer your code + data in this folder

## General purpose stuff - resource management

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


## Local mounting for data transfer
- German [guide](https://wiki.hhu.de/display/HPC/Filesysteme+mounten)

  

## Install your Python packages locally 

TODO
```
pip install --upgrade --target=<your-project/user-folder>/site-packages -i http://pypi.repo.test.hhu.de/simple/ --trusted-host pypi.repo.test.hhu.de <your-awesome-package>
```
Here is an example to use  scikit-learn
```
pip install --upgrade --user   --target=/gpfs/project/adaloglo/site-packages  -i http://pypi.repo.test.hhu.de/simple/ --trusted-host pypi.repo.test.hhu.de scikit-learn
```


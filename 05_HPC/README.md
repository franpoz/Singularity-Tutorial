
As stated previously, a lot is shared between the container and the original machine (filesystem/network/...).
However if you need to run an mpi run within the cluster, you need to have to install a (compatible) software stack 
within your container.

For lemaitre3, we have provide three container that corresponds to the three foss version available on that cluster.
The current default foss version  is foss/2018b which use OpenMPI-3.1.1.

In this part of the tutorial we will use those basic container in order to create a simple hello-world examples using mpi.

In this case our deffile will be 


```
BootStrap: library
From: omatt/default/mpi:3.1.1

%runscript
    /usr/bin/mytest-mpi

%files
    test.cc /opt/test-mpi.c

%post
    echo "Hello from inside the container"
    mpicc -o /usr/bin/mytest-mpi /opt/test-mpi.c
```

Note that you need to have a file test.cc available in your local directory
You can get this one via the following command
```
scp lemaitre3:/CECI/soft/src/singularity/test.cc .
```

Then the "build" step of the container will compile this file with the mpicc executable installed within the container that we have provided.
Then in order to run it on lemaitre3, you have to use the foss/2018b toolchain and then run it:
```
module load releases/2018b
srun -n 4 -p debug  "singularity run ./mpi-test.sif"
```
However when you do that you will see a Huge error. If you look at the begining of the (long) error message you will have the following:
```
--------------------------------------------------------------------------
A call to mkdir was unable to create the desired directory:

  Directory: /localscratch/omatt
  Error:     Read-only file system

Please check to ensure you have adequate permissions to perform
the desired operation.
```

OpenMPI expect to write to the /localscratch directory and we therefore need to bind that directory to a physical directory.
For each node/job an environment variable $LOCALSCRATCH is defined on the fly. However you can not simply use it like this 
```
module load releases/2018b
srun -n 4 -p debug  singularity run -B $LOCALSCRATCH:/localscratch ./mpi-test.sif
```
since $LOCALSCRATCH is only define at run time (that variable contains your slurm job id)
so you need to postpone the evaluation of the variable to the time of the run being executed:
```
module load releases/2018b
srun -n 4 -p debug  bash -c "singularity run -B \$LOCALSCRATCH:/localscratch ./mpi-test.sif"
```

and you will see:
```
srun: job 68320800 queued and waiting for resources
srun: job 68320800 has been allocated resources
Hello world from processor lm3-w003.cluster, rank 0 out of 4 processors
Hello world from processor lm3-w003.cluster, rank 1 out of 4 processors
Hello world from processor lm3-w003.cluster, rank 2 out of 4 processors
Hello world from processor lm3-w003.cluster, rank 3 out of 4 processors
```


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
Then in order to run it on lemaitre3, you have to use the foss/2018b toolchain
```
module load releases/2018b
srun -n 4 -p debug  "singularity run -B /localscratch:$LOCALSCRATCH/$SLURM_JOB_ID ./mpi-test.sif"
```

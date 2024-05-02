* How to build

```
FC=gfortran BRANCH=master ARMCI_NETWORK=MPI-PR MPI_IMPL=ompi  apptainer build --fakeroot  -F exachem.simg Singularity
```

* How to run on Azure

```
srun --mpi=pmix  \
-u \
 --gpus-per-node 8 --gpu-bind=closest    \
 apptainer exec \
 --nv \
 --bind /anfhome,/mnt,/etc,/sched,/run \
 oras://ghcr.io/ExaChem/exachem-container-images/apptainer.ompi41x:latest \
 /opt/install/exachem/bin/ExaChem \
 ~/exachem_inputs/ubiquitin_dgrtl.json
 ```

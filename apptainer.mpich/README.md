Slurm script for running on OLCF Frontier as of January 28 2025
```
module purge
module load PrgEnv-gnu
module load rocm/"${ROCM_VERSION}"
module  load cray-mpich-abi
module load libfabric/1.15.2.0
module -t list

export OMP_NUM_THREADS=1
export APPTAINERENV_LD_LIBRARY_PATH="$CRAY_MPICH_DIR/lib-abi-mpich:$CRAY_MPICH_ROOTDIR/gtl/lib:/opt/rocm/lib:/opt/rocm/lib64:$CRAY_LD_LIBRARY_PATH:$LD_LIBRARY_PATH:/opt/cray/pe/lib64:"
export APPTAINER_CONTAINLIBS="/usr/lib64/libcxi.so.1,/usr/lib64/libjson-c.so.3,/lib64/libtinfo.so.6,/usr/lib64/libnl-3.so.200,/usr/lib64/libgfortran.so.5,/usr/lib64/libjansson.so.4"
MYFS=$(findmnt -r -T . | tail -1 |cut -d ' ' -f 1)

export all_proxy=socks://proxy.ccs.ornl.gov:3128/
export ftp_proxy=ftp://proxy.ccs.ornl.gov:3128/
export http_proxy=http://proxy.ccs.ornl.gov:3128/
export https_proxy=http://proxy.ccs.ornl.gov:3128/
export no_proxy='localhost,127.0.0.0/8,*.ccs.ornl.gov'
export PERMANENT_DIR=$MEMBERWORK/$SLURM_JOB_ACCOUNT
mkdir -p $PERMANENT_DIR
export ORGDIR=$(pwd)
export BINDS=/opt/amdgpu,/usr/share/libdrm,/var/spool/slurmd,/opt/cray,${MYFS}
export APPTAINER_CACHEDIR=$PERMANENT_DIR/cache
mkdir -p $APPTAINER_CACHEDIR
export APPTAINERENV_PATH=/opt/install/exachem/bin:/opt/mpich/bin:/opt/rocm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export ROMIO_FSTYPE_FORCE=lustre:
export MPICH_MPIIO_CB_ALIGN=3
export MPICH_MPIIO_AGGREGATOR_PLACEMENT_DISPLAY=1

export MPICH_GPU_SUPPORT_ENABLED=0
export MPICH_SMP_SINGLE_COPY_MODE=NONE
export FI_CXI_RX_MATCH_MODE=hybrid
export MPICH_OFI_VERBOSE=1

## modify with name of input file
export TAMM_INP=$ORGDIR/inputs/uracil.json
export TAMM_EXE=/opt/install/exachem/bin/ExaChem

export MYIMG=oras://ghcr.io/Exachem/exachem-container-images/apptainer.mpich3.4.2-main.amd_gfx90a_rocm6.2.jammy:
echo MYIMG is $MYIMG
srun  -N $SLURM_NNODES --tasks-per-node=9  --gpus-per-node=8 \
apptainer exec --bind $BINDS   --rocm  $MYIMG $TAMM_EXE $TAMM_INP  >& uracil.out.$SLURM_JOBID
```
## Intro

Here are a few pointers about ExaChem.

The documentation page is at https://exachem.readthedocs.io/en/latest/

A couple of difference between NWChem and ExaChem that is worth mentioning for your CCSD(T) runs

1. ExaChem makes use of the Cholesky decomposition https://exachem.readthedocs.io/en/latest/user_guide/cholesky_decomposition.html

2. The ExaChem input used the JavaScript Object Notation (JSON) standard for formatting. You can see plenty of examples in

https://github.com/ExaChem/exachem/tree/main/inputs

and following file shows all the available input options

https://github.com/ExaChem/exachem/blob/main/inputs/example.json

The input options are also discussed in the documentation webpage mentioned earlier.

Since getting the JSON formatting right can be an issue, you could check the validity of your json input file with the command
```
cat "filename" | python -m json.tool
```

### Slurm script
```
#!/bin/bash
#SBATCH -C gpu
#SBATCH -t 0:9:59
#SBATCH -q debug
#SBATCH -N 2
#SBATCH -A m3196
#SBATCH --ntasks-per-node=10
#SBATCH --cpus-per-task=12
#SBATCH --gpus-per-node=4
#SBATCH --gpu-bind=none
#SBATCH -J exachem
#SBATCH -o exachem.%j.out
#SBATCH -e exachem.%j.out
module load cray-pmi
module list
export GA_NUM_PROGRESS_RANKS_PER_NODE=2
export FI_CXI_RX_MATCH_MODE=hybrid
export COMEX_EAGER_THRESHOLD=16384
export FI_CXI_RDZV_THRESHOLD=16384
export FI_CXI_OFLOW_BUF_COUNT=6
export FI_CXI_DEFAULT_CQ_SIZE=262144
export MPICH_SMP_SINGLE_COPY_MODE=NONE
export FI_CXI_DEFAULT_CQ_SIZE=13107200
export FI_CXI_REQ_BUF_MIN_POSTED=10
export FI_CXI_REQ_BUF_SIZE=25165824
export MPICH_GPU_SUPPORT_ENABLED=0
MYIMG=ghcr.io/edoapra/exachem-container-images/podman:latest
orgdir=$(pwd)
BIND=--volume=$orgdir:$orgdir:rw
WORKDIR=--workdir=$orgdir
podman-hpc pull --quiet $MYIMG
srun -l -N $SLURM_NNODES   podman-hpc run  $BIND  $WORKDIR --env-host  --mpi --gpu $MYIMG /opt/install/exachem/bin/ExaChem input.json
```

## Input file

```
{
  "geometry": {
    "coordinates": [
	"bqSe   -1.87713183   -1.06487690    0.26121977",
	"bqSe    1.86057570   -1.10244892    0.05512568",
	"Se      0.00070797    2.16291170   -0.27426568",
	"bqH    -1.45330721   -1.12621174   -1.13027871",
	"bqH    -3.25753574   -0.79531985   -0.13108814",
	"bqH     2.14395400   -0.50446811   -1.24161851",
	"bqH     2.57106498   -2.32519639   -0.30911319",
	"H      -0.02404148    1.76303646    1.12549057",
	"H       0.96592777    3.22206654    0.00832991"
    ],
    "units": "angstrom"
  },
  "basis": {
    "basisset": "cc-pVdZ",
    "atom_ecp": {
	"Se" :"cc-pvdz-pp"
    },
    "atom_basis": {
      "bqSe": "cc-pVdZ-PP",
      "Se": "cc-pVdZ-PP"
    }
  },
  "SCF": {
    "restart": false,
    "noscf": false
  },
    "CC": {
      "writet": false,
      "readt": true,
      "tilesize": 140,	
      "writet_iter": 2,
      "freeze": {
          "core": 12
      },
      "CCSD(T)": {
          "cache_size": 4,
          "skip_ccsd": false,
          "ccsdt_tilesize": 40
      }
  },
  "TASK": {
    "scf": false,
    "mp2": false,
    "cd_2e": false,
    "ccsd": false,
    "ccsd_t": true
  }  
}

```
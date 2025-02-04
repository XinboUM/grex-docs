# Ho to run Julia jobs

## Available Julia versions

Presently, binary Julia version 1.3.0 is available. Use ```module spider julia``` to find out other versions.

## Installing packages

We do not maintain centralized versions of Julia packages. Each user should install Julia modules in their home directory. 

The command is (in Julia REPL): 

    Using Pkg; Pkg.Add("My-Package")
    
In case of package/version conflicts, remove the packages directory _~/.julia/_ .

## Using Julia notebooks

It is possible to use IJulia kernels for Jupyter notebooks. A preferrable way of running a Jupyter notebook is SLURM interactive job with **salloc** command.

(More details coming soon).

## Running Julia jobs 

Julia comes with a large variety of packages. Some of them would use threads; and therefore, have to be ran as SMP jobs with _--cpus-per-task_ specified. Moreover, you would want to set _JULIA_NUM_THREADS_ environment variable in your job script to be the same as SLURM's number of threads. 

## Using Julia for GPU programming

It is possible to use Julia with CUDA Array objects to greatly speed up the Julia computations.  For more information, please refer to this link: [julia-gpu-programming](https://nextjournal.com/sdanisch/julia-gpu-programming). However, a suitable "CUDA" module should be loaded during thee installation of the CUDA Julia packages. And you likely want to be on a GPU node when the Julia GPU code is executed.


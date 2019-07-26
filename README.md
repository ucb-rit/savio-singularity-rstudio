# savio-singularity-rstudio
Materials for creating Singularity container for running RStudio server on Savio.

## Overview

This repository provides the materials needed to create a Singularity container that provides an RStudio server session to run on a compute node (or on the viz node) and for the Savio user to connect to that RStudio session from the viz node. It builds on the Rocker Dockerfile for RStudio/RStudio server, modified such that the RStudio server can start when not running as root.

## To use the container on Savio

### Using RStudio without authentication

  1) Start RStudio server on a compute node via `srun` or `sbatch` as follows.  If you're using `srun`, start your interactive session via `srun` and then invoke the Singularity command below. If you're using `sbatch`, invoke Singularity command below within your submission script. 
  
     ```singularity run rstudio-server-0.3.simg```
  2) Note the name of the Savio node, e.g., `n0070.savio2` on which the job started.
  3) Login to the Savio visualization node, start a vncserver session, and connect to a VNC Viewer window (i.e., a remote desktop session) following [these instructions](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc).
  4) From a terminal in the remote desktop session run the following (changing `n0070.savio2` as needed to the node from step 2):
     - `firefox http://n0070.savio2:8787`
  5) When you are done with RStudio, make sure to kill your `srun` or `sbatch` session so you are not charged for time you don't need.
 
Note that it's possible another user could connect to your RStudio session if you don't use authentication. 

### Using RStudio with authentication

  1) Start RStudio server on a compute node via `srun` or `sbatch` as follows.  If you're using `srun`, start your interactive session via `srun` and then invoke the Singularity command below. If you're using `sbatch`, invoke Singularity command below within your submission script. You can set the password (here 'foo') to whatever you desire.
  
     ```PASSWORD=foo singularity run rstudio-server-0.3.simg --auth-pam-helper-path /usr/lib/rstudio-server/bin/pam-helper --auth-none 0```
  2) Note the name of the Savio node, e.g., `n0070.savio2` on which the job started.
  3) Login to the Savio visualization node, start a vncserver session, and connect to a VNC Viewer window (i.e., a remote desktop session) following [these instructions](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc).
  4) From a terminal in the remote desktop session run the following (changing `n0070.savio2 as needed` to the node from step 2):
     - `firefox http://n0070.savio2:8787`
  5) Authenticate with RStudio using your username (your SAvio username) and the password you entered in step 1.
  6) When you are done with RStudio, make sure to kill your `srun` or `sbatch` session so you are not charged for time you don't need.

### Adding R packages to your container

While one could add additional R packages to the container itself, the easiest thing to do as a user is to install additional packages outside the container, which is easy to do because Singularity automatically gives you access to files on the host system.

One possibility is to simply use `install.packages` inside the container. That will likely install into the `R` subdirectory of your home directory on the host system (if `~/R/x86_64-pc-linux-gnu-library/3.6` exists, where 3.6 in this case is the version of R being used). This might be fine but runs the risk of conflicting with R packages that you've installed for use on the host system.

Here's an alternative that isolates the additional packages in a directory. In this example, we install the `assertthat` and `testthat` packages:

```
export SING_R_DIR=~/singularity_R
mkdir ${SING_R_DIR}
SINGULARITYENV_R_LIBS_USER=${SING_R_DIR} singularity exec  \
   rstudio-server-0.3.simg Rscript -e "install.packages(c('assertthat', 'testthat'))"
```

Now when you want to run R inside the container, make sure to set `SINGULARITYENV_R_LIBS_USER` (which sets `R_LIBS_USER` inside the container, such that R knows where to find the packages you've just installed), for example:

```
SINGULARITYENV_R_LIBS_USER=${SING_R_DIR} singularity run rstudio-server-0.3.simg
```

### Using R without RStudio

You can also use R without using the RStudio interface. For interactive use, after invoking `srun`, you just tell the Singularity container to run R, like this:

```
singularity exec rstudio-server-0.3.simg R
```

Similarly for a batch job via `sbatch`, you tell the Singularity container to run `R CMD BATCH`:

```
singularity exec rstudio-server-0.3.simg R CMD BATCH --no-save file.R file.Rout
```

## To build the container

You'll need access to a machine where you have administrative privileges (i.e., 'root' access). 


```
sudo singularity build /tmp/rstudio-server-0.3.simg rstudio-server-0.3.def
```

Alternatively (i.e., without any special privileges), if you're using a newer version of Singularity, you may be able to build the container via Sylabs Cloud Remote Builder, like this:

```
singularity build --remote rstudio-server-0.3.simg rstudio-server-0.3.def
```

You'll need to create an account with Sylabs Cloud. More details are [here](https://www.sylabs.io/guides/3.1/user-guide/singularity_and_docker.html#building-containers-remotely).

Notes:

 - Unlike the Rocker definition on which this is based, don't use `/init` in your container definition as that causes RStudio server to try to write to /var/run, which would only be possible with a writable image in which permissions on /var are modified; use `rserver` instead as the command to start RStudio server.
 - Make sure you have `pam-helper.sh` copied into container (the Dockerfile does this as long as pam-helper.sh is in the build directory. That allows authentication to work by overriding usual pam authentication, which won't work on Savio: see [https://github.com/nickjer/singularity-rstudio/issues/1](https://github.com/nickjer/singularity-rstudio/issues/1).
 - Note that @paciorek can't figure out how to run the container as a service (using `singularity instance.start`). If I try to use `rserver` to start RStudio server inside the container, the user sees a login page, and I'm not sure why it is asking for authentication. If I try to run with ```rserver --auth-pam-helper-path /usr/lib/rstudio-server/bin/pam-helper --auth-none 0```, I don't know how to pass PASSWORD into instance.start.


# savio-singularity-rstudio
Materials for creating Singularity container for running RStudio server on Savio.

## Overview

This repository provides the materials needed to create a Singularity container that provides an RStudio server session to run on a compute node (or on the viz node) and for the Savio user to connect to that RStudio session from the viz node. 

## To use the container

### Using RStudio without authentication

Note that it's possible another user could connect to your RStudio session if you don't use authentication. 

  1) Execute the following via srun or sbatch:
  
     ```singularity run rstudio_server_0.1.img```
  2) Login to the Savio viz node, start a browser (e.g., `/global/scratch/kmuriki/firefox`) within a [VNC remote desktop session](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc), and connect to `<nodename>:8787`, where `<nodename>` is the node that your srun/sbatch job is running on, e.g., `n0072.savio2`.

### Using RStudio with authentication

  1) Execute the following via srun or sbatch, setting the password (here 'foo') to whatever you desire:
  
     ```PASSWORD=foo singularity run rstudio_server_0.1.img --auth-pam-helper-path /usr/lib/rstudio-server/bin/pam-helper --auth-none 0```
  2) Login to the Savio viz node, start a browser (e.g., `/global/scratch/kmuriki/firefox`) within a [VNC remote desktop session](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc), and connect to `<nodename>:8787`, where `nodename` is the node that your srun/sbatch job is running on, e.g., `n0072.savio2`.
  3) Authenticate with RStudio using your Savio username and the password you entered in step 1.

## To build the container

```
sudo singularity build /tmp/rstudio_server_0.1.img rstudio_server_0.1.def
```

Notes:

 - Unlike the Rocker definition on which this is based, don't use `/init` in your container definition as that causes RStudio server to try to write to /var/run, which would only be possible with a writable image in which permissions on /var are modified; use `rserver` instead as the command to start RStudio server.
 - Make sure you have `pam-helper.sh` copied into container (the Dockerfile does this as long as pam-helper.sh is in the build directory. That allows authentication to work by overriding usual pam authentication, which won't work on Savio: see [https://github.com/nickjer/singularity-rstudio/issues/1](https://github.com/nickjer/singularity-rstudio/issues/1).
 - Note that I can't get `singularity instance.start` to work -- either with or without authentication I'm unable to get into the RStudio session. 




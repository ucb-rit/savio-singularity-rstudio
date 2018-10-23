# savio-singularity-rstudio
Materials for creating Singularity container for running RStudio server on Savio.

## Overview

This repository provides the materials needed to create a Singularity container that provides an RStudio server session to run on a compute node (or on the viz node) and for the Savio user to connect to that RStudio session from the viz node. It builds on the Rocker Dockerfile for RStudio/RStudio server, modified such that the RStudio server can start when not running as root.

## To use the container on Savio

### Using RStudio without authentication



  1) Execute the following via `srun` or `sbatch`:
  
     ```singularity run rstudio_server_0.1.img```
  2) Note the name of the Savio node, e.g., `n0070.savio2` on which the job started.
  3) Login to the Savio visualization node, start a vncserver session, and connect to a VNC Viewer window (i.e., a remote desktop session) following [these instructions](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc).
  4) From a terminal in the remote desktop session run the following (changing `n0070.savio2 as needed` to the node from step 2):
     - `/global/scratch/kmuriki/firefox http://n0070.savio2:8787`
  5) When you are done with RStudio, make sure to kill your `srun` or `sbatch` session so you are not charged for time you don't need.
 
Note that it's possible another user could connect to your RStudio session if you don't use authentication. 

### Using RStudio with authentication

  1) Execute the following via `srun` or `sbatch`, setting the password (here 'foo') to whatever you desire:
  
     ```PASSWORD=foo singularity run rstudio_server_0.1.img --auth-pam-helper-path /usr/lib/rstudio-server/bin/pam-helper --auth-none 0```
  2) Note the name of the Savio node, e.g., `n0070.savio2` on which the job started.
  3) Login to the Savio visualization node, start a vncserver session, and connect to a VNC Viewer window (i.e., a remote desktop session) following [these instructions](https://research-it.berkeley.edu/services/high-performance-computing/using-brc-visualization-node-realvnc).
  4) From a terminal in the remote desktop session run the following (changing `n0070.savio2 as needed` to the node from step 2):
     - `/global/scratch/kmuriki/firefox http://n0070.savio2:8787`
  5) Authenticate with RStudio using your Savio username and the password you entered in step 1.
  6) When you are done with RStudio, make sure to kill your `srun` or `sbatch` session so you are not charged for time you don't need.

## To build the container

```
sudo singularity build /tmp/rstudio_server_0.1.img rstudio_server_0.1.def
```

Notes:

 - Unlike the Rocker definition on which this is based, don't use `/init` in your container definition as that causes RStudio server to try to write to /var/run, which would only be possible with a writable image in which permissions on /var are modified; use `rserver` instead as the command to start RStudio server.
 - Make sure you have `pam-helper.sh` copied into container (the Dockerfile does this as long as pam-helper.sh is in the build directory. That allows authentication to work by overriding usual pam authentication, which won't work on Savio: see [https://github.com/nickjer/singularity-rstudio/issues/1](https://github.com/nickjer/singularity-rstudio/issues/1).
 - Note that @paciorek can't figure out how to run the container as a service (using `singularity instance.start`). If I try to use `rserver` to start RStudio server inside the container, the user sees a login page, and I'm not sure why it is asking for authentication. If I try to run with ```rserver --auth-pam-helper-path /usr/lib/rstudio-server/bin/pam-helper --auth-none 0```, I don't know how to pass PASSWORD into instance.start.

# Guide to the HPC cluster at [ISTA](https://ista.ac.at)

## Establishing an SSH connection

Communication with the HPC cluster is performed via `ssh`. To set the connection up:
1. Determine the name of a login node to which you wish to connect. [Here is the full list](https://scicomp.ista.ac.at/slurm/slurm/). Try logging in via username+password combination:
    ```
    ssh <user_name>@<front_node>.hpc.ista.ac.at
    ```
    Log out, when succeeded. 
2. Create an SSH key at `~/.ssh/keys/ISTA_HPC` and the respective entry in `~/.ssh/config`, using [this guide](https://github.com/mkmaslov/guides/blob/main/linux-client/ssh-client.md). Transfer the **public** key to the remote:
    ```
    ssh-copy-id -i ~/.ssh/ISTA_HPC.pub <username>@<frontnode>.hpc.ista.ac.at
    ```


## SLURM commands

To be added...
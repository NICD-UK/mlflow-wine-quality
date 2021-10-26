# uncertainty-estimation

## Configuring remote server

Create resource on Azure.

### Add a new user

`useradd` will provide a walkthrough for creating a new user

Create a `~/.ssh/authorized_keys` file for the new user and give them ownership

```
chown new-user ssh
```

Add the new user to the sudoers group

```
usermod -aG sudo mynewuser
```

On the new user's machine, create a new ssh-key

```
ssh-keygen
```

Update `~/.ssh` config file with details of the host IP address, the new username and identity file.

```
Host dl
    HostName xx.xxx.xx.xx
    User mynewuser
    IdentityFile ~/.ssh/id_rsa
```

Temporarily enable password login by modifying `/etc/ssh/sshd_config` to include the setting `PasswordAuthentication Yes` and run `sudo service ssh restart`

From the new user machine, copy the id to the remote

```
ssh-copy-id dl
```

You will be prompted for your password to login, which will copy the id to the `~/.ssh/authorized_keys` file.

You can now login and modify `/etc/ssh/sshd_config` to include the setting `PasswordAuthentication No` and run `sudo service ssh restart`.

# Running mlflow

To run an `mlflow` project locally and log the metrics to a remote server and log artifacts to Azure Blob Storage. First set two environment variables

```
MLFLOW_TRACKING_URI=http://mlflow.uksouth.cloudapp.azure.com:8080
MLFLOW_TRACKING_USERNAME=xxxxxxxxxxxxxxxxxxx
MLFLOW_TRACKING_PASSWORD=xxxxxxxxxxxxxxxxxxx
AZURE_STORAGE_ACCESS_KEY=xxxxxxxxxxxxxxxxxxx
```

To ensure you are using the Docker daemon on the training VM you can run the following command:

> **_NOTE:_** this will only work if you have SSH access to the training VM

```
export DOCKER_HOST="ssh://azureuser@uq-training.uksouth.cloudapp.azure.com"
```

Build the docker container and tag it with the same name as defined in `MLproject`.

```
docker build . -t mlflow-learn
```

Start the run on MLFlow with a detached docker container, using `-A d`.

```
mlflow run . -A d
```

To submit a run to a particular experiment, e.g. 'ExperimentName'.

```
mlflow run --experiment-name ExperimentName . -A d
```

To select a different entry point in the `MLproject` file, e.g. 'EntryPointName', for a run.

```
mlflow run -e EntryPointName . -A d
```

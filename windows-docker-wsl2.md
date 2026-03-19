# Installing Docker on Windows Using WSL2, NOT Docker Desktop
These instructions help Windows users install Docker inside a WSL2 VM, without using 
Docker Desktop, which can be flaky when updating and has licensing fees.

**Note:** If you already have Docker desktop installed you don't have to 
uninstall but you also need to install docker on wsl2 container if you want to run docker commands in it. 

## Step 0. Optional Terminal install 
Install Windows Terminal application(https://learn.microsoft.com/en-us/windows/terminal/install) This provides a clean customizable interface to connect to the wsl2 container(s) installed.

## Step 1.  Install WSL2
Open powershell as admin and run the following commands. 

```
wsl --install -d Ubuntu-24.04
```
Once installation is complete run:
```
sudo apt update
```

## Step 2.  Install Docker in WSL2 VM
From a WSL terminal, run these commands.

1. Install Docker:
    ```
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

2. Add your user to the Docker group
    ```
    sudo usermod -aG docker $USER
    ```

3. Sanity check that both tools were installed successfully
    ```
    docker --version
    docker compose version
    ```

4. If you have docker desktop installed on your machine, move the default docker config so that docker auth from within wsl2 container does not try to use docker desktop's exe.
   ```
   mv ~/.docker/config ~/.docker/config_backup
   ```

6. Set your Docker daemon to run on starup

    From a WSL terminal, edit `/etc/wsl.conf`
    ```
    sudo nano /etc/wsl.conf
    ```

    Add the following to the end of the file:
    ```
    [boot]
    systemd=true
    ```
7. Restart WSL

    From PowerShell terminal:
    ```
    wsl --shutdown
    wsl
    ```

8. Make sure systemd is running

    From your new WSL terminal, run
    ```
    systemctl list-unit-files --type=service
    ```

    If you see a bunch of services, all is well.  Hit `q` to exit.

9. Make sure Docker is running
    From your new WSL terminal, run
    ```
    ps aux | grep docker
    ```

    You should see that the docker daemon is running.  Something like:
    ``` 
    root         263  0.6  0.1 5284636 139760 ?      Ssl  Sep20 144:46 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    ```

## Step 4a. If you've already got code cloned and want to move your code to WSL
All of your code has to be migrated under the WSL VM in order for you to use it.  
In Explorer, you can see it at the bottom under Linux/Ubuntu

![Image](./windows-explorer-wsl.png)

Note that your WSL username is the one you set up when you installed WSL.  I picked my 
same windows username and password so I would remember.

You should symlink any config files or folders that currently exist in your Windows home folder
to your WLS home folder (e.g., ~/.ssh, ~/.aws, etc.) 

For example, to link your .ssh folder (known hosts and keys), from your WSL terminal, run:
```
ln -s /mnt/c/Users/$USER/.ssh ~/.ssh
```

## Step 4b. Clone code from WSL
VS code will make your life easier when it comes to git commands and credentials so from wsl terminal do:
```
mkdir test
cd test
code .
```
Follow prompts to have WSL extension added to VS Code, you may have to restart Code. Then open bash terminal in vs code to execute the git clone commands, follow prompts and click 'ok' in browser window that will eventually open to allow VS Code permissions to git.

## Step 5. Install tools in WSL
From WSL terminal:

For node and npm use nvm:
https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl

For python and to use virtual envs:
```
sudo apt install python-is-python3
sudo apt install python3.10-venv
```

For aws-cli:
(if using aws cli from within a venv be sure to activate the venv AFTER installing node)

Install or update the latest version of the AWS CLI - AWS Command Line Interface (amazon.com)
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html



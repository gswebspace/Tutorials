# Backtesting with QuantConnect LEAN Engine and Docker

## Introduction
[Docker containers](https://www.docker.com/resources/what-container) are useful to minimize setup times and deployments as an application and its dependencies are packaged into one **docker image**. These docker images can be run as a **docker container** on your machine in an isolated environment. 

QuantConnect LEAN Engine and docker can be used to easily setup a development environment without worrying about installing any required packages and dependencies.

This tutorial starts from scratch on a development host and teaches you how to perform backtesting using LEAN easily.

## Requirements

- A development machine
    - Supported processor architectures : AMD64/X64 with [SLAT](http://en.wikipedia.org/wiki/Second_Level_Address_Translation)
    - Supported OS : Linux, Windows and macOS
    - 4 GB RAM or more
    - BIOS-level hardware [virtualization](https://docs.docker.com/docker-for-windows/troubleshoot/#virtualization-must-be-enabled) support enabled
    - 20GB+ Free space

## Tutorial
To proceed with the tutorial follow the steps below

- Install *Docker* on your host using the instructions provided [here](https://docs.docker.com/get-docker/). To verify the installation run `docker ps` or `docker info` in a terminal. The command should execute without errors.
    - For linux users, you can use docker as a non-root user by adding your user to the *docker* group by executing `sudo usermod -aG docker $USER`. You will need to reboot to start using docker as your user.
    
- Create a new development directory.
  
- Open a command prompt/terminal on your host and navigate to the development directory.

- Clone the LEAN repository in the development directory using `git clone --branch master https://github.com/QuantConnect/Lean.git`.
  
    - **[META] THIS STEP WILL BE REMOVED IN THE FINAL DOCUMENTATION**
      
        Clone the modified Lean repo using `git clone --branch lean_and_docker https://github.com/gswebspace/Lean.git`
  
    - **[META] THIS STEP WILL BE REMOVED IN THE FINAL DOCUMENTATION**
      
        cd into the repo and prepare the *lean-build-run* **docker image** using the following command
        
          `docker build -t lean-build-run -f DockerfileBuildRun .`
      
- To execute the code follow these steps -
    
    - Change directory into the source code repo.
    
    - Create a *Results* folder in the directory.
    
    - Run the helper script using `./build_run_docker.sh`
    
    - The script will ask you to define the config, Data and Results path. You can change the defaults if you are using a different directory, otherwise use the defaults by pressing the Enter key.
    
    - Once the script completes, the output will be available on the terminal as well as the Results folder.
        
          
            $ ./build_run_docker.sh 
            Enter docker image [default: <meta/redacted>:latest]: 
            Enter absolute path to Lean config file [default: /home/<redacted>/Lean/Launcher/config.json]: 
            Enter absolute path to Data folder [default: /home/<redacted>/Lean/Data]: 
            Enter absolute path to store results [default: /home/<redacted>/Lean/Results]: 
            Enter absolute path to the code folder [default: /home/<redacted>/Lean]: 
            Preparing packages...
            MSBuild auto-detection: using msbuild version '14.0' from '/usr/lib/mono/xbuild/14.0/bin'.
            .
            .
            .
            .
            .
            20200713 03:34:04.295 Trace:: BacktestingResultHandler.SendAnalysisResult(): Processed final packet
            20200713 03:34:04.296 Trace:: Engine.Run(): Disposing of setup handler...
            20200713 03:34:04.296 Trace:: Engine.Main(): Analysis Completed and Results Posted.
            Engine.Main(): Analysis Complete. Press any key to continue.
            

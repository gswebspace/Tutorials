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
    
    <video width="320" title="Docker installation" controls>
      <source src="img/lean-docker-install.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
    
- Create a new development directory.

- In the development directory create a new file called `Dockerfile` with the following contents

```
# Base image used is Ubuntu
FROM ubuntu:18.04

# apt update and install useful packages
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    curl git apt-transport-https nano 

# Install Mono + nuget
RUN apt install -y gnupg ca-certificates && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF && echo "deb https://download.mono-project.com/repo/ubuntu stable-bionic main" | tee /etc/apt/sources.list.d/mono-official-stable.list
RUN apt-get update && apt install -y binutils mono-complete ca-certificates-mono mono-vbnc nuget referenceassemblies-pcl fsharp

# Install python and dependencies
RUN apt-get install -y build-essential python3.6 python3-pip  python3-dev
RUN pip3 install cython==0.29.11 pandas==0.25.3 wrapt==1.11.2

# Setup a build and run script
RUN bash -c 'echo -e "cd /Lean\nnuget restore QuantConnect.Lean.sln\nmsbuild QuantConnect.Lean.sln\ncd /Lean/Launcher/bin/Debug\nmono QuantConnect.Lean.Launcher.exe\ncd /Lean"' > /usr/local/bin/lean-build-run && chmod u+x /usr/local/bin/lean-build-run

# Clone the LEAN repo
RUN git clone --branch 8772 --depth 1 https://github.com/QuantConnect/Lean.git

# Open a new shell for the user
CMD ["/bin/bash"]

```
    
- Open a command prompt/terminal on your host and navigate to the development directory.

- Prepare your LEAN environment **docker image** using the following command
  
    `docker build -t lean-env .`
  
- To confirm if the image has been created run `docker images` and you will see an image called *lean-env*.

- Create a docker container from the newly created *lean-env* image

    `docker run -it --name lean-env lean-env`
    
    <video width="320" title="LEAN container setup" controls>
      <source src="img/lean-container-setup.mp4" type="video/mp4">
      Your browser does not support the video tag.
    </video>
    
- Install VS Code to use it for development

    - Install VS Code from instructions [provided here](https://code.visualstudio.com/docs/setup/setup-overview)
      
        <video width="320" title="Installing VS Code" controls>
          <source src="img/lean-vscode-install.mp4" type="video/mp4">
          Your browser does not support the video tag.
        </video>
      
    - Setup VS Code and extensions
      
        - Run VS Code and install the [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) extension pack 
        
        - Connect VS Code to the docker container by opening a *Remote Window* from the bottom left of the editor and choose "Attach to running container".
        
        - Install the [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
        
        - Install the [VS Code .csproj extension](https://marketplace.visualstudio.com/items?itemName=lucasazzola.vscode-csproj). Go into File > Prefererences > Settings > Extensions > csproj configuration  > Item Type > Edit in settings.json and add the following -
          
            ```
            "csproj.itemType": {
                ".cs" : "Compile"
            }
            ```
        
        - Install the [Mono Debug extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.mono-debug).
        
        - Open */LEAN* folder in VS Code.
        
        - Create a new Run/Debug configuration by selecting Run > Create a launch.json file and add the following object in the  *configurations* key -
          
            ```
            {
              "name": "LEAN",
              "type": "mono",
              "request": "launch",
              "program": "/Lean/Launcher/bin/Debug/QuantConnect.Lean.Launcher.exe",
              "cwd": "/Lean/Launcher/bin/Debug"
            }
            ```
        
        - Create a task to build the project from VS Code by pressing ctrl+shift+p and selecting Tasks: Configure Task > Open tasks.json file > Others. Add the following to *tasks* -
            
            ```
            {
              "label": "LEAN Build",
              "type": "shell",
              "command": "nuget restore QuantConnect.Lean.sln && msbuild QuantConnect.Lean.sln",
              "options": {
                  "cwd": "/Lean"
              },
              "problemMatcher": [],
              "group": {
                  "kind": "build",
                  "isDefault": true
              }
            }
            ```
            
        
        - Build the project by pressing `ctrl+shift+b` or by running `ctrl+shift+p` > *Tasks: Run Task* > LEAN Build.
        
        - Goto the Run view and select *LEAN* and hit the play icon to start/debug the program.

		<video width="320" title="VS Code Setup" controls>
			<source src="img/lean-vscode-setup.mp4" type="video/mp4">
			Your browser does not support the video tag.
		</video>

      
- Create and add your Backtesting algorithm

    - Add a new file for your algo under `Algorithm.Csharp` and name it `MyAlgo.cs`.

    - The *VS Code .csproj extension* will ask you to add the file to the project, click Yes.

    - Add the following code to the file (or your own custom implementation) 
    
            
            using QuantConnect.Data.Market;

            namespace QuantConnect.Algorithm.CSharp
            {

                public class MyAlgo : QCAlgorithm
                {
                    decimal myprice = 0;

                    private readonly Symbol _ibm = QuantConnect.Symbol.Create("IBM", SecurityType.Equity, Market.USA);

                    public override void Initialize()
                    {
                        SetStartDate(2013, 01, 01);  //Set Start Date for backtesting
                        SetEndDate(2014, 01, 01);    //Set End Date for backtesting
                        SetCash(100000);             

                        AddSecurity(SecurityType.Equity, "IBM", Resolution.Hour);
                        Securities[_ibm].SetLeverage(1.0m);
                    }

                    public void OnData(TradeBars data)
                    {
                        if (!data.ContainsKey(_ibm)) return;
                        
                        if(!Portfolio.Invested){
                            SetHoldings(_ibm, 1.0m);
                            myprice = data[_ibm].Value;
                        } else {
                            if(data[_ibm].Value >  (myprice*1.05m)){
                                SetHoldings(_ibm, 0.5m);
                            }
                        }
                    }
                }
            }
            
            
          
     - Modify the *Launcher/config.json* file -
        
         - Change *algorithm-type-name* to `MyAlgo`.
     
         - Make sure that *algorithm-language* is set to `CSharp`
         
         - If your data is kept in a different directory modify *data-folder* to the appropriate path. For more info see  [Importing Custom Data](https://www.quantconnect.com/docs/algorithm-reference/importing-custom-data#Importing-Custom-Data).
       
     - Build the program by pressing `ctrl+shift+b`. The build command output is available via the TERMINAL for the LEAN Build task.
     
     - Run the program by selecting Run > LEAN > Play icon.
     
     - In the *DEBUG CONSOLE* you can see the output of the program including various statistics like Total Trades, Average Win, Net Profit etc.

		<video width="320" title="Adding new backtesting algorithm" controls>
                        <source src="img/lean-my-algo.mp4" type="video/mp4">
                        Your browser does not support the video tag.
                </video>


## Helpful commands

- Restore nuget packages by executing the following inside your container
```
cd /Lean
nuget restore QuantConnect.Lean.sln
```
- Compile your code using the container
```
cd /Lean
msbuild QuantConnect.Lean.sln
```
- Run your code using the container
```
cd /Lean/Launcher/bin/Debug
mono QuantConnect.Lean.Launcher.exe
```

- Run helper script to do nuget restore, compile and run operations together using `lean-build-run`.

- If you want to restart a exited *lean-env* container, use `docker start -i $(docker ps -a -q --filter "status=exited" --filter "name=lean-env")`

# Server Wiki
#### Guidelines and tips for server usage

Authored by Aidan Winters 2/15/2022. Updated 2/28/2022


## Gilbert Lab Servers:
- Marlowe (Node9): This is a single node on a compute cluster containing 9 nodes. Our node, Node 9, is available to the gilbert lab for interactive use (ie Jupyter, Rstudio). Keep in mind, jobs can also be submitted to the entire Marlowe cluster
  - Specs: 64 CPU, 1.5 TB RAM, 210 TB hard drive
- Norgay/Tenzing: These are twin servers that are used entirely interactively (no job queue) so make sure to allocate resources appropriately and communicate with other users when you are running resource-heavy jobs. These servers are shared with the Goodarzi lab.
  - Specs: 56 CPUs, 756 GB RAM, 65 TB Hard drive 

---
---
---

## Getting Started

### Setting up an Account
1. Marlowe: please email David.Quigley@ucsf.edu for a new account
2. Norgay/Tenzing: please email Aidan.winters@ucsf.edu with your desired username and follow instructions to reset password


### Things to know to use Command Line Interface (CLI):
There are several commands that you need to know to comfortably use Marlowe. 
A few of those include: `ls`, `cd`, `ssh`, `vim/nano`

This tutorial is pretty good to learn more: https://www.educative.io/blog/bash-shell-command-cheat-sheet


---

### Logging in to Marlowe
Marlowe is a computing cluster (also called a server) with 9 compute nodes and one 'head' or 'login' node that must first be accessed before accessing any of the compute nodes. We access this login node through our local computer (aka your laptop/desktop computer).
Below are the basic commands for ssh but <strong>I highly recommend you use the ssh config method (<a href="#ssh_config">described below</a>)</strong>

To log into the head node, type the following on your local computer: 
```
ssh <my_username>@marlowe.cc.ucsf.edu
```
Here, you can submit jobs that are then distributed amongst the compute nodes or log in to Node9 which has been set aside for Gilbert lab interactive use. 
Logging in to Node9 from the head node is done like so: 
```
ssh node9
```

Alternatively, you can log in to Node9 directly from your local computer using the following command: 
```
ssh <my_username>@node9.marlowe.cc.ucsf.edu -J <my_username>@marlowe.cc.ucsf.edu
```

---

### Removing Password requirement for logging in to marlowe
You may have noticed that logging in to Marlowe (Head node) from your local computer or logging in to Node9 from the head node require you to enter your password. To enable auto login, you must copy over an ssh key. You will need to do this twice to be able to log in to both the Head node and Node9. This requires 3 steps: creating an ssh key, copying the key to desired destination, verifying your password one last time. 
- To remove the password for logging in from local computer to Marlowe Head node: 
   - Navigate to your local computer's terminal and then type `ssh-keygen`  and hit enter through all the prompts
   - Then, type `ssh -i ~/.ssh/id_rsa <my_username>@marlowe.cc.ucsf.edu`
   - Enter your password and then try logging in with `ssh <my_username>@marlowe.cc.ucsf.edu`
- To remove the password for logging in from Marlowe Head node to Node9: 
   - Login to the head node and then type `ssh-keygen`  and hit enter through all the prompts
   - Then, type `ssh -i ~/.ssh/id_rsa <my_username>@node9`
   - Enter your password and then try logging in with `ssh <my_username>@node9`


### <a id="ssh_config">Set up SSH Config file</a>
This method is really nice because it simplifies logging into our servers and get rid of the annoying double jump needed for getting on Node9. To enable this, you need to add ssh profiles to a file called `config` in your `.ssh/` folder in your home directory. The path to that file would be `~/.ssh/config. If the file is not there please create one. To do this add the following to you .ssh config file and make sure to replace the areas where indicated: 
```
Host marlowe
  User <YOUR_USERNAME_HERE>
  HostName marlowe.cc.ucsf.edu
  ProxyJump bch-svr

Host node9
  User <YOUR_USERNAME_HERE>
  HostName node9.marlowe.cc.ucsf.edu
  ProxyJump marlowe
```

After doing this you should be able to log in to Marlowe like so: `ssh marlowe` and into node9: `ssh node9`

Additionally, if you need port forwarding for something like Jupyter (<a href="#port_forward">see below</a>) you can update the "node9" profile to look like this: 
```
Host node9
  User <YOUR_USERNAME_HERE>
  HostName node9.marlowe.cc.ucsf.edu
  ProxyJump marlowe
  LocalForward <MY_PORT_NUMBER_HERE> localhost:<MY_PORT_NUMBER_HERE>
```

Then set up the forwarding in the background like so: `ssh -Nf node9`

---
---
---

## Installing Conda

Conda is a package manager originally built for python but has expanded to handle all sorts of languages, including R (Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN). This replaces most packages that would typically be downloaded with the `pip` installer. I highly recommend using conda as the packages are generally well maintained and they make it possible to have isolated groups of packages together called conda 'environments'.

Most folks should install conda through Anaconda as it comes with most of the python packages you need. Follow these steps to intall: 
1. navigate to the [conda distribution site](https://www.anaconda.com/products/distribution) and copy the download link for the latest Linux installer.
2. Log on to Marlowe head node and type `wget <copied_anaconda_link>` to download the installer to Marlowe. That should download a file that looks like `Anaconda_SomeOtherStuff.sh`.
3. To run this installer, type `bash Anaconda_SomeOtherStuff.sh` and hit enter through the prompts. 
4. Verify conda was installed by typing `conda` into your terminal. 

Alternatively to Anaconda, there is a smaller conda install that is much faster but doesn't come with all the typical python packages like numpy/pandas. To install conda through Miniconda, follow the same steps but download the miniconda installer from [the conda site](https://docs.conda.io/en/latest/miniconda.html)

---

#### Creating New Conda Environments: 

Conda allows for isolated 'environments' or collections of packages. This is super helpful for organizing projects but also for keeping potentially problematic pacakges (not well-maintained, requires weird dependencies, etc.) from messing with your other packages. When you install conda, you are automtically operating out the `base` environment.
Creating a conda environment is super easy. 
- Create an empty environment: 
  `conda create --name my_environment`
- Create an environment with a specific python version
  `conda create --name py35_environment python=3.5`
- Create an environment and install a few packages at the same time: 
  `conda create --name my_environment jupyter numpy pandas`
  
Once you have created the environment, you can switch between which environment you are using by 'activating' it like so: 
`conda activate my_environment`
You should see your environment name in parentheses at the front of your command line prompt. You can go back to the base environment with: 
`conda activate base`

---

#### Using Conda Environments with Jupyter
Once you have created a new conda environment, aside from using those packages on the command line, it is useful to be able to use them through jupyter. There are two ways to do this. I strongly recommend installing your conda environment as a 'jupyter kernel'. There is a section below on how to do this. Another option is to run your jupyter notebook within the environment you wish to access (assuming it has its own installation of jupyter). This would look something like this: 
1. `conda activate my_environment`
2.  `jupyter notebook`

---

However, jupyter uses some files that are globally accessed by all your conda environments that makes this a confusing approach and why I recommend you install conda environments as separate jupyter kernerls. With installed kernels you can addtionally, within the same jupyter notebook session, use notebooks with different kernels instead of starting a new notebook for each different conda environment you want to access. 

---

#### Install Mamba for fast install: 
Conda is fine itself but if you find yourself constantly installing new packages and waiting for them to download/install all their dependencies, check out `mamba` which is a parallelized conda installer and is much faster. Mamba is easily installed through conda like so: `conda install -c conda-forge mamba`

---
---
---

## Interactive Use
Node9 is set aside for interactive use. This means that you do not have to submit jobs but can work on the computer directly. A major advantage of this is the ability to run a jupyter notebook or Rstudio on Node9. 

---


### Running Jupyter on Node9 and Connecting to your Notebook: 
- <strong>Running Jupyter notebook.</strong> Running your notebook is fairly simple. Typically, when you run jupyter on your laptop/desktop, jupyter automatically opens your notebook in your browser. However, there is not browser available on servers and so you need to connect your local computer's browser to the notebook running on Marlowe. This requires we specify a location, or port, for the jupyter notebook to be available.
  - First, ensure you have jupyter notebook installed (see conda section above)
  - Navigate onto Node9 and then type the following and choose a 4-digit port number: `jupyter lab --no-browser --port=<my_port>`
    - You can also use `jupyter notebook` here instead of `jupyter lab` but I recommend using the latter.
  - Check the output of running this command and verify that the port you selected was actually used by the jupyter notebook. If a new port was selected, make a note of it and use that port going forward. That port should remain available unless another user enters the same numbers.

- <strong>Connecting to your notebook (aka Port Forwarding).</strong> Now, we need to actually make the jupyter notebook available to our local computer's browser. 
  - This command uses ssh to 'listen' to the port we specified when we ran jupyter on Node9. Type the following: `ssh <my_username>@node9.marlowe.cc.ucsf.edu -J <my_username>@marlowe.cc.ucsf.edu -NL <my_port>:localhost:<my_port>`
  - Then, open your browser (Chrome recommended) and type the following address to access your notebook: `localhost:<my_port>`

Please see the Wynton page for a more in-depth explanantion of port forwarding: https://wynton.ucsf.edu/hpc/howto/jupyter.html

- <strong>Optional but HIGHLY recommended: Running your Jupyter notebook in the background:</strong>
- All commands run in the terminal are run within a 'session' or 'shell'. That means that when you run your jupyter notebook on Node9, it will be killed once you close that terminal window or log out of node9. This is almost always a huge pain in the ass. 
- To get around this, we can use the command `nohup` which explicitly runs a command and ignore all 'hangup' signals. In other words, your jupyter notebook will only stop if you kill it explicitly or if Marlowe goes down.
  - The modified jupyter command should look like this:  `nohup jupyter lab --no-browser --port=<my_port> &> jup.out &`
  - A couple important notes when running in the background: 
    - If you need to check if this command is still running, you should type `ps -x | grep jupyter`. This will output any running process that contains the word 'jupyter'.
    - You still need to verify what port was chosen by jupyter just in case the port you chose was already taken. Since we ran jupyter in the background, the output from jupyter is saved in a file called `jup.out`. Check that file to see what port was chosen by typing `head -n20 jup.out` wherever you ran the jupyter command.  


### Adding Conda Environments as Kernels to Jupyter
---

Creating a conda environement is useful for isolating distinct projects. But if you would like to use those collection of packages in jupyter, the best method is to install a conda environment as a 'kernel' or programming environment to jupyter. Because conda is useable with several languages, you can install python and R kernels this way. 

##### Installing a conda environment as a Python kernel
---
1. Create your conda environment as described above. Make sure that you [install ipykernel](https://anaconda.org/conda-forge/ipykernel) in that environment.
2. Activate your conda environment: `conda activate my_environment`
3. Then run this command and name your environment:  
    `python -m ipykernel install --user --name my_environment --display-name "My First Python Env"`
4. To access this kernel, either select the kernel when you make a new notebook or, for an existing notebook, you should see a drop down menu on the top left header of the jupyter notebook where you can select a kernel to use.

#### Installing an R environment using Conda (or mamba)
(Highly recommended: I wrote everything in `mamba` commands below since it is much faster but can be switched with `conda` if you prefer)
1. Install jupyter in your base environment: `mamba install -c conda-forge jupyterlab`
2. Create new conda env and simultaneously install R base and Rkernel: `mamba create -n MY_R_ENV -c conda-forge r-base r-irkernel`
    - Rkernel lets you 'install' this conda environment as a kernel on jupyter so you can select it in a jupyter notebook
3. Once those are installed activate that environment: `mamba activate my_R_env`
4. Verify that R is installed within that environment with `which R`
    - This should look something like this: `/my_home/mambaforge/envs/MY_R_ENV/bin/R`
5. Next, open up an R session by typing `R`
6. Then finally to install that conda environment as a jupyter kernel, run this in R with your desired environment name: `IRkernel::installspec(name = 'MY_R_ENV', displayname = 'My R Kernel')`
7. Now when you go to create a new notebook you should see "My R Kernel" as an option for conda kernels (usually in the top right corner of jupyter notebook)


---

## Connect Node9 to the Internet
To connect Node9 to the internet, you need to set up a proxy through the Marlowe head node. Add the following to your `.bashrc` in your home directory: 

```
export HTTP_PROXY=http://169.230.134.132:3128
export HTTPS_PROXY=http://169.230.134.132:3128
```

This allows you to install through conda on Node9 and use python/R packages on Node9 that require internet access. Warning though, this disabled conda installs for me on Marlowe head node. It is an imperfect solution. 

## Useful Tips: (TO BE ADDED)

Bash Prompt:

Network connection on Node9: 

Helpful extra tools: 
- Vim
- Autojump

---

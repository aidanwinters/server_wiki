# Server Wiki
#### Guidelines and tips for server usage

Authored by Aidan Winters 2/15/2022. Updated 2/28/2022


## Gilbert Lab Servers:
- Marlowe (Node9): This is a single node on a compute cluster containing 9 nodes. Our node, Node 9, is available to the gilbert lab for interactive use (ie Jupyter, Rstudio). Keep in mind, jobs can also be submitted to the entire Marlowe cluster
  - Specs: 64 CPU, 1.5 TB RAM, 210 TB hard drive
- Norgay/Tenzing: These are twin servers that are used entirely interactively (no job queue) so make sure to allocate resources appropriately and communicate with other users when you are running resource-heavy jobs. These servers are shared with the Goodarzi lab.
  - Specs: 56 CPUs, 756 GB RAM, 65 TB Hard drive 

---
## Getting Started

### Setting up an Account
1. Marlowe: please email David.Quigley@ucsf.edu for a new account
2. Norgay/Tenzing: please email Aidan.winters@ucsf.edu with your desired username and follow instructions to reset password

### Logging in to Marlowe
Marlowe is a computing cluster (also called a server) with 9 compute nodes and one 'head' or 'login' node that must first be accessed before accessing any of the compute nodes. We access this login node through our local computer (aka your laptop/desktop computer).
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

## Installing Conda

Conda is a package manager originally built for python but has expanded to handle all sorts of languages, including R (Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN). This replaces most packages that would typically be downloaded with the `pip` installer. I highly recommend using conda as the packages are generally well maintained and they make it possible to have isolated groups of packages together called conda 'environments'.

Most folks should install conda through Anaconda as it comes with most of the python packages you need. Follow these steps to intall: 
1. navigate to the [conda distribution site](https://www.anaconda.com/products/distribution) and copy the download link for the latest Linux installer.
2. Log on to Marlowe head node and type `wget <copied_anaconda_link>` to download the installer to Marlowe. That should download a file that looks like `Anaconda_SomeOtherStuff.sh`.
3. To run this installer, type `bash Anaconda_SomeOtherStuff.sh` and hit enter through the prompts. 
4. Verify conda was installed by typing `conda` into your terminal. 

Alternatively to Anaconda, there is a smaller conda install that is much faster but doesn't come with all the typical python packages like numpy/pandas. To install conda through Miniconda, follow the same steps but download the miniconda installer from [the conda site](https://docs.conda.io/en/latest/miniconda.html)


#### Install Mamba for fast install: 
Conda is fine itself but if you find yourself constantly installing new packages and waiting for them to download/install all their dependencies, check out `mamba` which is a parallelized conda installer and is much faster. Mamba is easily installed through conda like so: `conda install -c conda-forge mamba`

---

## Interactive Use
Node9 is set aside for interactive use. This means that you do not have to submit jobs but can work on the computer directly. A major advantage of this is the ability to run a jupyter notebook or Rstudio on Node9. 

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

---

## Useful Tips: 

Bash Prompt:

Network connection on Node9: 

Helpful extra tools: 
- Vim
- Autojump

---

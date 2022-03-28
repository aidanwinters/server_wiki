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
---

## Interactive Use
Node9 is set aside for interactive use. This means that you do not have to submit jobs but can work on the computer directly. A major advantage of this is the ability to runa jupyter notebook or Rstudio on Node9. 

### Running Jupyter on Node9 and Connecting to your Notebook: 
- <strong>Running Jupyter notebook.</strong> Running your notebook is fairly simple. Typically, when you run jupyter on your laptop/desktop, jupyter automatically opens your notebook in your browser. However, there is not browser available on servers and so you need to connect your local computer's browser to the notebook running on Marlowe. This requires we specify a location, or port, for the jupyter notebook to be available.
  - First, ensure you have jupyter notebook installed (see conda section above)
  - Navigate onto Node9 and then type the following and choose a 4-digit port number: `jupyter lab --no-browser --port=<my_port>`
    - You can also use `jupyter notebook` here instead of `jupyter lab` but I recommend using the latter.
  - Check the output of running this command and verify that the port you selected was actually used by the jupyter notebook. If a new port was selected, make a note of it and use that port going forward. That port should remain available unless another user enters the same numbers.

- <strong>Connecting to your notebook (aka Port Forwarding).</strong> Now, we need to actually make the jupyter notebook available to our local computer's browser. 
  - This command uses ssh to 'listen' to the port we specified when we ran jupyter on Node9. Type the following: `ssh <my_username>@node9.marlowe.cc.ucsf.edu -J <my_username>@marlowe.cc.ucsf.edu -NL <my_port>:localhost:<my_port>`
  - Then, open your browser (Chrome recommended) and type the follwing address to access your notebook: `https://localhost:<my_port>`

- <strong>Optional but HIGHLY recommended: Running your Jupyter notebook in the background:</strong>
---   

Please see the Wynton page for a more in-depth explanantion of port forwarding: https://wynton.ucsf.edu/hpc/howto/jupyter.html

---

## Useful Tips: 
---

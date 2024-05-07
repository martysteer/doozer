## Using ansible to setup bits on a raspberry pi

Steps to install ansible on your macbook (client):

```bash
brew install pipx
pipx ensurepath
pipx install --include-deps ansible
```

Setup the ansible inventory.ini. It imports the host shortname from .ssh/config, so get your ssh config working first, then you can use those hostname aliases in your ansible inventory:

```ini
[pihosts]
doozer
```

Test it works:

```bash
$ ansible pihosts -m ping -i inventory-example.ini
doozer | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

To make this your default ansible inventory, putting the path to your inventory.ini file into an environment variable allows you to rerun ansible commands without the explicit inventory -i switch:

```bash
export ANSIBLE_INVENTORY="~/inventory.ini"
# or make it persist for your user environment
# echo 'export  ANSIBLE_INVENTORY="~/inventory.ini"' >> ~/.bashrc
# echo 'export  ANSIBLE_INVENTORY="~/inventory.ini"' >> ~/.zshrc
ansible pihosts -m ping
```

Now you can run playbooks from your client machine to configure your pi (assuming it is on your local network).

<br />

## Running playbooks to configure doozer

Now, you can use ansible to configure your networked pi and set it up for stuff!

I kept the playbooks focussed on a singular task to try to make it easier for myself to understand as I grasped how ansible worked and what the purpose of using it as an abstraction control layer was.

Too much generalisation and too much abstraction is confusing and inhibits learning.
Keep it simple.
Keep automation to one or two tasks at a time.
Understand how the tasks relate to each other.
Understand how the system components relate to each other.
Comprehend your tools and methods.

Abstraction of network operations, which consider attention towards the TCP/IP stack, are useful to always keep in mind when working with computers and software. It's surprising how plugged in your machines have to be to get close to machine learning when they are airgapped on their own.

<br />

### Operating System tooling

The YAML playbooks in this directory can be used to configure the system for one thing at a time:

* Increase the size of the PiOS swap file:
  ```ansible-playbook swap.yaml```
* Install and setup basic Samba network share:
  ```ansible-playbook samba.yaml```
* Install pyenv:
  ```ansible-playbook pyenv.yaml```

<br />

### Software tooling

There are a number of ways to configure your software tools. How many abstractions you use is up to you. Understanding the cost/benefits of software abstractions is important. Software is not easily separated from it's host operating system.

Now use ansible to install the following software tooling:

- jupyterlab - kind of working. jupyter.yaml playbook installs jupyter lab into a pyenv managed python 3.8.4 virtual environment, and sets up a systemd service. This works fine and jupyter lab starts up on boot. Adding other kernels into it from other pyenv virtual envs isn't fully working properly.
- Hi-SAM heirarchical segment anything model project - this sets up separate pyenv environment, installs the code, and requirements (pytorch CPU only at the moment) and downloads the pretrained checkpoint files (about 16 large .pth files totalling approx. 6GB)
- 
- TFLite + Coral TPU device drivers
- 
- 
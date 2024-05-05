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

We have kept the playbooks focussed on a singular task to try to make it easier for someone who doesn't know ansible to learn to use it gradually, and to therefore gain confidence with researching and developing their own little computer for their own research software engineering project.

Too much generalisation and too much abstraction is confusing and inhibits learning.
Keep it simple.
Keep automation to one or two tasks at a time.
Understand how the tasks relate to each other.
Understand how the system components relate to each other.
Understand your tools and methods.

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

Now use ansible install the following software tooling:

- jupyterlab
- TFLite + Coral TPU device drivers
- 
- 
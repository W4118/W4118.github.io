# SSHing into your VM Instance

The VM GUI can seem excessive if all we really need out of our virtual machine is a terminal window. Most students are more comfortable working in their host machine than in the VM. We can enable this by configuring remote `ssh` access.

Note that this guide will refer to your VM as a _guest machine_, and your personal computer (running the VM) as the _host machine_. All commands should be executed in the **terminal of your guest machine** by default, unless they're prefixed with `[host]`.

## Set up OpenSSH on Ubuntu

Install OpenSSH:

```
$ sudo apt install openssh-server
```

To check whether the OpenSSH server is running:

```
$ sudo systemctl status ssh
```

If the server isn't running, start it using:

```
$ sudo systemctl start ssh
```

## SSH into your VM

### SSHing with mDNS Resolution (Recommended)

By default, VMware does not assign static IP addresses to local VMs. This means that VMware can change your VM’s IP address. It is possible to stop VMware from doing that, and you may do so if you wish. But an easier solution is to delegate the work of finding your VM’s IP address to your host machine instead of doing it yourself. This is possible because of mDNS resolution (aka zeroconf networking). To use this method, run:

```
[host] $ ssh <username>@<hostname>.local
```

You should use the username and hostname you used during VM setup. `<hostname>.local` will be resolved to your VM’s current IP address for you. This makes setting up an SSH config file very simple, and requires the least amount of work in the long run.

If you're using this method, from now on, when we say `<host>`, you should replace that with `<hostname>.local`.

### SSHing without mDNS Resolution (Not recommended)

Older systems might not support mDNS resolution. In that case, it is possible to retrieve the current IP address of your local VM manually. In the terminal of your VM, run:

```
$ ip addr
```

Once you obtain the current IP address, you may log in to your guest machine using:

```
[host] $ ssh <username>@<ip-addr>
```

> NOTE: Since VMware doesn’t assign static IP addresses by default, the IP address may change across sessions, which will make the SSH command fail. You’ll have to requery the IP address if that happens and replace the old IP address in your SSH command.

If you're using this method, from now on, when we say `<host>`, you should replace that with `<ip-addr>`.

## Set Up SSH Keys

### Using SSH to authenticate your host machine

Are you tired of entering your password to log into your VM whenever you connect via SSH? If so, you’ll want to set up SSH keys so that your host machine can SSH into your VM without using a password.

First, if you haven’t already, generate an SSH key pair on your host machine and add it to your `ssh-agent` following [this excellent guide by GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

Now that you have your key pair, your host machine can use it to authenticate when SSHing into your VM. Assuming that you named your SSH key pair `id_ed25519` (the default), run the following command:

```
[host] $ ssh-copy-id <username>@<host>
```

This copies the public Ed25519 key on your host machine to the `~/.ssh/authorized_keys` file on your guest machine. The server running on your VM will use this to authenticate the SSH client on your host machine.  If you named your SSH key pair something else, you can manually edit the `~/.ssh/authorized_keys` file on your guest machine to include the **public** key from your host machine.

### Pushing from your VM to GitHub

You will also need to ensure your VM can push directly to GitHub via SSH (this is needed since GitHub no longer supports `https` authentication). To do this, you’ll want to forward your host’s SSH credentials into your VM.

First, make sure you follow [this guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) to add the public key on your host machine to your GitHub account. This ensures that the key pair on your host machine can be used for authenticating with GitHub.

Next, create your SSH config file if you haven’t already:

```
[host] $ mkdir -p ~/.ssh
[host] $ touch ~/.ssh/config
```

The final step will be to 'give' your host's key pair to your VM, so that your VM can offer it up when authenticating with GitHub. Here are two configuration options that you can put in `~/.ssh/config` using your favorite text editor:

```
# Specific Rule (Recommended)

Host osvm
    HostName <host>
    User <username>
    ForwardAgent yes
    AddKeysToAgent yes

# Blanket Rule (Not recommended)

Host *
    ForwardAgent yes
    AddKeysToAgent yes
```

The blanket rule says that for any host you SSH into successfully, you want to forward your credentials. Note that if you decide to do this, you should be careful to **only SSH into trusted machines**. Using the specific rule also gives you the added advantage of being able to SSH into your VM with the following shorter command:

```
[host] $ ssh osvm
```

Alternatively, if you don’t want to deal with SSH configurations, you may simply generate a new SSH key inside your VM. Be sure to add that key to your GitHub profile using the guide mentioned above.

You can test your GitHub connection with the following command:

```
$ ssh -T git@github.com
```

If you see a greeting with your username, you’re all set! Happy coding!

## Using VS Code (Optional)

If you’re a fan of Visual Studio Code, you can work on your VM directly from a VS Code window on your host machine. This allows you to take advantage of all of VS Code’s features (such as quickly opening any file without navigating directories using `Ctrl + P`, which becomes really useful when dealing with Linux kernel source code).

To enable SSH access via VS Code, all you have to do is install the [VS Code SSH Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh). Then, make sure you've set up your SSH config file as mentioned above.

From now on, you can SSH into your VM by opening your Command Palette (`Ctrl-Shift-P`) and looking for `Remote-SSH: Connect to Host`. Alternatively, you can click the double triangle icon at the bottom left corner of the window. This will give you a variety of SSH-related options. Selecting `Connect to Host` or `Connect Current Window to Host` will give you a list of remote machines from your SSH config file. Choose your OS VM - if you’ve properly configured your SSH keys, it should log in automatically and start setting up a remote VS Code server.

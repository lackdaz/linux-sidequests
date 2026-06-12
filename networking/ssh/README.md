# SSH

## Prerequisites

- Basics of networking (ip addresses/hostnames)

This is the install steps for getting an ssh client up

## Installing openssh-client

```
sudo apt update
sudo apt install openssh-client
```

Make a `.ssh` folder at the home directory and navigate there
```
mkdir ~/.ssh
cd ~/.ssh
```
> Note: being in the .ssh directory makes setup a lot easier


Create ed25519 keys and fill out the interactive prompt, and exclude a passphrase for first-timers (read note)
```
ssh-keygen
```
> Note: typically you would do without a passphrase for passwordless login/scripting - so no passphrase please

Then a pair of private and public keys will be created in `~/.ssh`, in this example we will use `test` and `test.pub`. These are your ed25519 keys and the one with the `.pub` extension is something you can freely share around. Keep the private key safe!

Now copy the public key to the server device:
```
ssh-copy-id -i ~/.ssh/test maker@<local/public-ip-address>
```
> Note: where the -i flag refers to the filename of the public key. It might look a bit confusing as it looks like its copying the private key over but it just takes the filename and appends .pub extension.

An interactive prompt should pop up asking for your permission for access. Continue with yes and the password for the **server device** will be requested. Upon authentication you should see something like:
```
Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i ./test 'maker@linuxers-pi.local'"
and check to make sure that only the key(s) you wanted were added.
```
which includes a helpful prompt to help you ssh in. Copy that into the next command (replace with your login credentials):

```
ssh -i ./test '<user>@<copy-your-ip-address-or-hostname.local>'
```

and voila you should have passwordless login!

## Issues
> Note: If you somehow still did not manage passwordless login, please contact seth. I tested these changes with two raspberry pis and there might have been overly permissive settings for passwordless login when customizing the raspbian image

## Setting up SSH aliases

Create the ssh config file in `~/.ssh/config`
```
cd ~/.ssh
touch config
```

and open the file with a text editor of your choice, I prefer VScode for this because of indentation but `vim` will work fine too

Let's create our first entry (might be best to copy the first ones to avoid typos):
```
Host youralias
    HostName 192.168.1.100
    User maker
    IdentityFile ~/.ssh/test
    StrictHostKeyChecking accept-new
```
> Note: Replace with your ip-address and identity file. The config file does not support inline commenting so be careful of that.

> Note: Remember you can have multiple entries in this file - creating a list of remote targets that VSCode Remote can easily parse for you

So let's look at some very useful entries:

This is the boilerplate for github `ssh`. Compulsory if you are doing code development!:
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/git_id_ed25519
    IdentitiesOnly yes
```
> Note: Please change the identity file to the one you have created specific for GitHub. In this case, I used a passphrase type for added security.

You can also change the ip address to a local hostname which mDNS will resolve
```
Host locallinux
    HostName linux101.local
    User maker
    IdentityFile ~/.ssh/test
```
> Note: Only works locally on the same subnet, and with mDNS/multicast enabled via avahi/Bonjour - this lives on each local device.

More commonly you can also define a different port access than port 22, typically for external port forwarding to a public IP:
```
Host publiclinux
    HostName 123.45.67.8
    User maker
    IdentityFile ~/.ssh/test
    Port 2222
```

This helps to suppress locale warnings from the client-side when `ssh`-ing into raspberry pis - very specific but annoying errors:
```
Host *
    SetEnv LC_ALL= LC_CTYPE= LC_MESSAGES=
```


+++
title = 'Ssh Tunnel'
date = 2024-07-28T14:22:56-07:00
draft = false
+++

# SSH Tunnel

Best explaination of the SSH commands
[![SSH Tunnel](../ssh-tunnel.png)](../ssh-tunnel.png)

# Setting up Reverse Tunnel

## Setup a simple web server with a websocket component

```
git clone https://github.com/radian-software/python-in-a-box
cd python-in-a-box
rm -rf package-lock.json
npm i
PORT=8080 node server.js
```

After this `curl http://localhost:8080` should give a response. Use a browser to navigate to [http://localhost:8080](http://localhost:8080) and it shows a python terminal running in the browser. The python process is running on the local machine with xterm.js in the browser connected to a websocket hosted on "/ws" of the web server.

![python-in-a-box](../python-local.png)

## Setting a proxy to the local web server

We will setup a virtual machine on the same machine as a proxy for demo purposes. Alternatively you can setup a VM on AWS or other cloud providers.

```
brew install utm
```

Start UTM and create a new VirtualMachine via the "Gallery" tab. Download "Arch linux" (should take only a few seconds) and start the VM. Login to the VM using "root/root" as the username/password.

For this new VM, we need to enable SSH to remote login as root and create a tunnel. Open "/etc/ssh/sshd_config" and change/add the following:

```
GatewayPorts yes
PermitRootLogin yes
```

You can use a less-privilege user as this instruction is for demo purpose.

Restart the ssh service in the VM

```
systemctl try-restart sshd
```

Run the following in the VM and note the IPv4 address. For me it's 192.168.64.2

```
ip addr show
```

## Setup a reverse tunnel from your dev machine or laptop

```
ssh -R 0.0.0.0:8080:0.0.0.0:8080 root@192.168.64.2
```

This will ask you for the root password of the VM, it's "root". This will login and land you into a shell prompt. confusing, but it's ok. :)

Now your can navigate to `http://192.168.64.2:8080` and see the python-in-a-box app running in the browser.

![python-in-a-box](../python-proxied.png)

You should be able to open multiple instances and they should be independent.

Say you're in a situation where you have 2 PCs: A and B.
Both PCs are on the same network.

A: A PC running Windows which does not have admin privileges. 
It also has a firewall which blocks all incoming connections, but it can make outgoing connections to a local server.

B: A PC running Linux or Windows which has admin privileges

This tutorial will show you how to host a VNC server on A and connect to it from B.



For A, you can host a VNC server locally, but since you don't have admin privileges, you can't allow incoming connections to the server.
Therefore, you can only host the server on localhost.

We can use SSH port forwarding to forward A's local VNC server port to B, so that we can connect to it from B.

## Step 1: Install VNC server on A
Since this machine is running Windows, I found your options are:
* TightVNC
* UltraVNC
* TigerVNC

I've tried all 3, and TigerVNC was the only one where I could establish a VNC connection, for some reason.
Get TigerVNC here: https://github.com/TigerVNC/tigervnc/releases and follow the "binaries" link for the latest release.

It's either "tigervnc64-winvnc" or "tigervnc64" (todo: figure out which one it is).

It's an installer, so you will need to install it on a PC where you have admin privileges, and then copy the installed files to A.
Since A is locked down, one inconspicuous way to get the files to A is to send them via Telegram.

## Step 2: Configure VNC server on A
When you run the VNC server on A, you may get a popup asking you to allow the server to communicate through the firewall.
Since you don't have admin privileges, you can't allow it. This is fine, since we just intend to 
host the server on localhost.

Change the VNC server settings as needed.
I had to disable sending and receiving clipboard data, since it was causing the server to crash.
Instead, use this to handle clipboard transmission: https://github.com/ikubaysan/ws-clipboard

Configure the VNC server to be hosted on port 5900, which should be the default.

## Step 3: Install SSH server on B

### Linux 
If B is running Linux, you likely already have an SSH server installed.
Open a terminal and run `sudo apt install openssh-server` to install it if you don't have it.

Now open the SSH server config file: `sudo nano /etc/ssh/sshd_config`
Find the "AllowTcpForwarding" line and change it to "AllowTcpForwarding yes".
It may just have a "#" in front of it, which you need to remove.

Restart the SSH server: `sudo service ssh restart`

Get your private IP address: `ifconfig` and look for the IP that starts with "10.0.0" or "192.168.0".

### Windows
Install Ubuntu with WSL2. Now follow the same steps as above.
However, there's one more step. You need to download this Powershell script:

https://gist.github.com/daehahn/497fa04c0156b1a762c70ff3f9f7edae

Open it in a text editor and look for `$Ports = (22,80,443,8080)`
You can replace it with just `$Ports = (22)` if you want, or leave it as is. We only need port 22 for SSH.

Now run the script in Powershell.

Then in WSL, run `sudo service ssh restart`

## Test A's ability to connect to B's SSH server
On A, open Powershell and run `ssh <username>@<B's IP address>`
The username is your username on B. If B is Windows and a password is not set, you need to set one as that
will be your SSH password.

## Step 4: Forward A's VNC server port to B
In Powershell on A, run `ssh -R 5901:localhost:5900 <username>@<B's IP address>`

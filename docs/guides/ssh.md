
# Getting Started with SSH
#### Written by Sean Mapes
This guide will go over how to connect to a server using secure-shell host (SSH). To learn about SSH, you can refer to a powerpoint [here](/powerpoints/HPCC-09-25-23.pdf).

## Prequisites
- Any computer with access to the `ssh` command
- Ethernet access on LU's network
> You will need an open ethernet port on your device. We have USB to Ethernet adapters if you need them.

## Setup
The below values are used throughout the guide. Whenever you see one of these, replace it with the value relevant in your use.

Value | Description
---|---
**`IP_ADDRESS`** | The IP address of the system you are connecting to.
**`USER_NAME`** | The user name that you are logging into on the server.

The **IP address** will be provided for whatever system you are attempting to connect to. This guide will use our club system, [Laputa](/systems/laputa), at the IP address [10.92.51.56]().

The **user name** will be the the same as the first part of your LU email, including anything before the @. This guide will use the student with the email ([skmapes@liberty.edu](mailto:skmapes@liberty.edu)), meaning that the user name is [skmapes]().

## Connecting to a Server
To connect to a server with IP address of **`IP_ADDRESS`** as user **`USER_NAME`**, we use the command
<pre>
<code>ssh <b>USER_NAME</b>@<b>IP_ADDRESS</b></code>
</pre>
> As a reminder, you need to replace **`USER_NAME`** with your user name and **`IP_ADDRESS`** with the IP address of the server!

It will ask you for a password. This password is set to a default by the system administrators, so ask them what your password is or ask them to reset it if you don't know what it is.
> Don't freak out if you don't see anything when you are typing! By default, Linux doesn't show anything when you are typing passwords for security reasons.

Once you are finished, use the `exit` command to disconnect from SSH.
```
exit
```

### Example
For user [skmapes]() connecting to Laputa at [10.92.51.56](), the command would be:
```
ssh skmapes@10.92.51.56
```

Entering the password for [skmapes]() grants access to Laputa.
> As a reminder, you should **use your own user name**, this is just an example!

## SSH Keys
Entering a password everytime is annoying and insecure, so for all of our systems we recommend and eventually require an SSH key.

To generate a key, use the `ssh-keygen` utility:
```
ssh-keygen
```
> We want to do this on our local machine, not the server! Make sure that you have disconnected from SSH using `exit` before making a key.

Just press enter a few times to use the default values.

From here we need to copy the key over to the server to use it. For Linux it is a single command, but for Windows  we have some manual work to do.

### Linux
To copy the SSH key over to a server at **`IP_ADDRESS`**, use the `ssh-copy-id` command:
<pre>
<code>ssh-copy-id -i .ssh/id_rsa.pub <b>USER_NAME</b>@<b>IP_ADDRESS</b></code>
</pre>

Now you should be able to log into the server without needing to type in a password.

### Windows
We need to make an SSH folder with the proper permissions on the server.
<pre>
<code>ssh <b>USER_NAME</b>@<b>IP_ADDRESS</b>
mkdir .ssh
chmod -R 700 .ssh
exit</code>
</pre>

To copy the SSH key over to a server at **`IP_ADDRESS`**, use the `scp` command:
<pre>
<code>scp .ssh/id_rsa.pub <b>USER_NAME</b>@<b>IP_ADDRESS</b>:~/.ssh/authorized_keys</code>
</pre>

Now we need to set the permissions of <code><b>USER_NAME</b>@<b>IP_ADDRESS</b>:~/.ssh/authorized_keys</code>:
<pre>
<code>ssh <b>USER_NAME</b>@<b>IP_ADDRESS</b>
chmod -R 644 .ssh/authorized_keys
exit</code>
</pre>

Now you should be able to log into the server without needing to type in a password.

## Troubleshooting

1. Hangs after entering the SSH command, or host unreachable
    1. You need to make sure you are using the correct IP address for the server. Double check that you can access that IP by pinging the IP (<code>ping -c 3 <b>IP_ADDRESS</b></code>) and verifying that it works.
    1. If you are attempting to access any of our systems, you **must** be wired into LU's network.
    > You can use a network cable from a lab or any of the desktops in any of the labs if you cannot connect your own device.
1. Password isn't working
    1. Make sure that the user that you are attempting to log in as exists on the server.
    1. Check and make sure you are entering the password for the user on the server, not your local password.
    > If you are sure the user exists and your password is right, talk to one of the administrators of the server to reset your password.
1. SSH prompts for a password after copying over a key
    1. Make sure you copied over your public key (`id_rsa.pub`) and not the private key (`id_rsa`).
    1. Make sure that you have the correct permissions on your `.ssh` folder and your `.ssh/authorized_keys` within the server.

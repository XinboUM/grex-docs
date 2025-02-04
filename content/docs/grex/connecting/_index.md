---
bookCollapseSection: true
weight: 20
title: Connecting / Transferring data
---

# Connecting to Grex

In order to use almost any HPC system, you would need to be able to somehow connect and log in to it. ALso, it would be necessary to be able to transfer data to and from the system. The standard means for these tasks are provided by the [SSH protocol](https://en.wikipedia.org/wiki/Secure_Shell).

To log in to Grex in the text mode, connect to **grex.westgrid.ca** using an **SSH** (secure shell) client. The DNS name grex.westgrid.ca serves as an alias for two login nodes: **bison.westgrid.ca** and **tatanka.westgrid.ca** .

Uploading and downloading your data can be done using an **SCP/SFTP** capable file transfer client. The recommended clients are OpenSSH (providing **ssh** and **scp**, **sftp** command line tools on Linux and MacOS X) and PuTTY/WinSCP/X-Ming or MobaXterm under Windows. Note that since Jun 1, 2014 the original "SSH Secure Shell" Windows SSH/SFTP client is not supported anymore.

Since Dec. 2015, support is provided for the graphical mode connection to Grex using **X2go**.
 
[X2go](https://wiki.x2go.org/doku.php/download:start) remote desktop clients are available for Windows, MacOS X and Windows. When creting a new session, please chose either of the supported desktop environments: **"OPENBOX"** or **"ICEWM"** in the "Session type" menu. The same Westgrid login/password should be used as for SSH text based connections. 

For now, there is no load balancing support for connections, so while connecting to the host address grex.westgrid.ca will work, session suspend/resume functionality might require specifying connection to a physical Grex login node explicitly, using **tatanka.westgrid.ca** or **bison.westgrid.ca**.

Finally, for graphical mode it is possible to use VNC remote desktop connections to Grex by tunneling them through SSH. After connecting by SSH as usual, do **module load vncworkspace** command on Grex and follow the instructions on how to use the conveninet vncworkspace wrapper over TigerVNC startup commands.

See the documentation for more details on how to connect from various client operaton systems:

- [Connecting with SSH](/doc/docs/grex/connecting/ssh/)
- [Connecting with X2Go](/doc/docs/grex/connecting/x2go/)


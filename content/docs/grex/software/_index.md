---
bookCollapseSection: true
weight: 60
---

# Software

In HPC world, software is more often meant as codes that do some scientific or engineering computation, data processing and visualiation (as opposed to web services, relational databases, client-server businness systems, email and office, ... etc.).

Tools and libraries used to develop HPC software are also software, and have several best practices associated with them. Some of  that will be covered below. Without means to provide software to do computations, HPC systems would be rather useless.

Fortunately most HPC systems, and Grex is no exception, come with a pre-installed, curated software stack. This section covers how to find the installed software, how to access it, and what options you have if some of the software you need is missing.

## How software is installed and distributed

There are several mechanisms of software installation under Linux. One of them is using Linux software package manager (_apt_ on Ubuntu, _yum_ on Centos, etc) and a binary package repository provided by some third party. These package managers would install a version of the code system wide, into standard OS directories like _/usr/bin_ where it would be immediately available in the systems PATH (PATH is a variable that specifies where the operating systems would look for executable code).
 
This method of installation is often practiced on person's own workstations, because it requires no knowledge other than syntax of the OS package manager. There are however significant drawbacks to it for using on HPC clusters that consists of many compute nodes and are shared by many users.

- Need of root access to install in the base OS is a systems stability and security threat, and has a potential of users interfering with each other.

- Package managers as a rule do not keep several versions of the same package; they are geared towards having only the newest one (as in "software update"), which poses a problem for reproducible research.

- A package should be installed across all the nodes of the cluster; thus the installations should be somehow managed.

- Binary packages in public repos tend to be compiled for generic CPU architectures rather than optimized for a particular system.

Thus in HPC world, as a rule, only a minimal set of core Linux OS packages is installed by system administrators, and no access to package managers is given to the end users. There are ways to let users have their own Linux OS images through virtualization and containerization technolgies (see the Containers section) when it is really necessary.

On most of the HPC machines, the application software is recompiled from sources and installed into a shared filesystem so that each compute node has access to the same code. Multiple versions of a software package can be installed into each own PATH; dependencies between software (such as libraries from one package needed to be accessed by another package) are tracked via a special software called Environmental Modules.

The Modules package would manipulate the PATH (and other systems environment variables like **LD_LIBRARY_PATH**, **CPATH** and application-specific ones like **MKLROOT**, **HDF5\_HOME**) on user request, "loading" and "unloading" specified software items.

The two most popular Modules are the original [Tcl Modules](http://modules.sourceforge.net/) and its [Lmod](https://lmod.readthedocs.io/en/latest/) rewrite in Lua at [TACC](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod). On the new Grex, **Lmod** is used.

## Lmod

The main feature of Lmod is hierarchical module system to provide a better control of software dependencies. Modules for software items that depend on a particular core modules (toolchains: a compiler suite, a MPI library) are only accessible after the core modules are loaded. This prevents situations where conflicts appear when software items built with different toolchains are loaded simultaneously. Lmod will also automatically unload conflicting modules and reload their dependencies should toolchain change. Finally, by manipulating module root paths, it is possible to provide more than one software stack per HPC system. For more information, please refer the software stacks available on Grex.

## How to find the software with Lmod Modules

A "software stack" module should be loaded first. On Grex, there are two software stack, called GrexEnv and CCEnv, and standing for the sofware
built on Grex locally and the software envornment from ComputeCanada, correspondingly. GrexEnv is the only module loaded by default. 

When a software stack module is loaded, the **module spider** command will find a specific software item (for example, GAMESS; note that
all the module names are lower-case on Grex and on ComputeCanada stacks) if it exist under that stack:

  ```module spider gamess```

It might return several versions; then usually a subsequent command with the version is
used to determine dependencies required  for the software. In case of GAMESS on Grex:

```module spider gamess/Sept2019```

It will advise to load the following modules:  _"intel/15.0.5.223  impi/5.1.1.109"_.
Then, **module load ** command can be used actually load the GAMESS environment (note that the dependencies must be loaded first:

```module load  intel/15.0.5.223  impi/5.1.1.109```

```module load gamess/Sept2019```

For more information about using Lmod modules, please refer to Compute Canada documentation about [Using Modules](https://docs.computecanada.ca/wiki/Utiliser_des_modules/en) and [Lmod User Guide](https://lmod.readthedocs.io/en/latest/010_user.html).

## How and when to install software in your HOME directory

Linux (Unlike some Desktop operatinhg systems) has a concept of user permissions separation. Regular users cannot, unless explicitly permitted, access systems files and files of other users.

You can almost always install software without **super-user** access into your /home/$USER directory. Moreover, you can manage the software with Lmod: Lmod automatically searches for module files under _$HOME/modulefiles_ and adds the modules it discovers there into the modules tree so they can be found by _module spider_, loaded by _module load_, etc.

Most Linux software can be installed from sources using either [Autoconf](https://www.gnu.org/software/autoconf/) or [CMake](https://cmake.org/) configuration tools. These will accept _--prefix=/home/$USER/my-sofware-version_ or _-DCMAKE_INSTALL_PREFIX=/home/$USER/my-software_ version.

Software that come as binary archive to be unpacked can be simply unpacked into your home directory location. Then, the paths should be set for the sofware to be found: either by including the environment variable in _$HOME/.bashrc_ or by creating a specific module in _$HOME/modulefiles/my-software/version_ following Lmod instructions for [writing Modules](https://lmod.readthedocs.io/en/latest/015_writing_modules.html). 

There exist binary software environments like Conda that manage their own tree of binary-everything. These can be used as well, with some caution, because automatically pulling everything might conflict with the same software existing in the HPC environment (Python package paths, MPI libraries, etc.).

However, if a software is really a part of the base OS (something like a graphics Desktop software, etc.), it can be hard to rebuild from sources due to many dependencies. If needed, it may be better if installed centrally or used in a container (see Containers documentation).


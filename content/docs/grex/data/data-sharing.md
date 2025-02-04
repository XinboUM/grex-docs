# Data sharing

Sharing of accounts login information (like passwords or SSH keys) is stricty forbidden on Grex, as well as on most of the HPC systems. There is a mechanism of data/file sharing that does not require sharing of the accounts. To access each others' data on Grex, the UNIX groups and permissions mechanism can be used as explined below. We use a Westgrid exlpanation because Grex is still using Westgrid accounts and groups database, so the text below is still relevant.

## UNIX groups

Each UNIX (or Linux) file or directory is owned by an individual user and also by a group (which may be comprised of several users). The permission to access files and directories can be restricted to just the individual owning the file, to the group that owns the file, or access can be unrestricted.

By default, each WestGrid account (username) is set up with an associated UNIX group containing just that single username. So, even if you have set permission for your UNIX group to access files, they are still not being shared with anyone else. You can override the default by using the chmod command to set unrestricted read access to your files. However, if you need more specific control over access, you can ask us to create a special UNIX group containing the usernames of other researchers with whom you want to share data by sending an email to WestGrid [support](mailto:support@computecanada.ca) to ask that a new UNIX group be created for you. Include a list of the users who should be added to that group. One user should be designated as the authority for the group. If a request to join the group is made from someone else, WestGrid will ask the designated authority for the permission to add the new researcher to the group. The group name must be of the format wg-xxxxx where xxxxx represents up to five characters. Please indicate your preference for a group name in your email.

The group will be set up on all the WestGrid systems. This may take a day or two to set up. You will get an email whenever you are added or removed from a WestGrid UNIX group.

Now that you have a wg-xxxxx UNIX group created, you can set up the data sharing with it, by setting the permissions as described below.

The directory you wish to share should be owned by the group and permitted to the group. For example:

  ```chgrp -R wg-group dir```

  ```chmod g+s dir```

You must ensure that there is access to parent directories as well.

A directory and all the files in it can be permitted to the group as follows:

  ```chmod -R g+rX /global/scratch/dirname```

Tto set access for the /global/scratch/dirname directory and all its subdirectories. Note the uppercase X in the command. This will set x permissions on the subdirectories (needed for others to list the directories) as well as regular execute permission on executable files.

If you want the to allow other members to not only read files in the shared directory "dir", but also permit write access to allow them to create and change files in that directory, then all members in the group must add a line:

 ```umask 007```

to the _.bashrc_ or _.cshrc_ file in their respective home directories. Furthermore, you must add write permission to the shared directory itself:

  ```chmod -R g+rwX dir```

which would allow read and write access to the directory dir and all its files and subdirectories.

## Linux ACLs

On Lustre filesystem (_/global/scratch/$USER_) it is possible to use Linux access control lists (ACLs) which offer more fine-grained control over access than UNIX groups.
ComputeCanada's [Sharing Data](https://docs.computecanada.ca/wiki/Sharing_data) documentation might be relevant, with one caution: on Grex, there is no project layout as exists on ComputeCanada.

An example setting of ACL command, to allow for "search" access of the top directory to a group wg-abcdf, presumably with some of the directories under it being shared by a UNUX group:

```setfacl  -m g:wg-abcdf:X /global/scratch/$USER```


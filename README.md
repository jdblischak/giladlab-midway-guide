# Guide to using RCC Midway

John Blischak

2016-07-27

**Last updated:** 2017-07-13

**Warning:** RCC Midway has recently transitioned to be composed of both Midway1
  (previously just Midway) and the new Midway2. Thus the intstructions below may
  not always work depending on which version of Midway you are on. In general,
  you should use Midway2 because it has newer infrastructure. However, it still
  only has available a subset of the software available on Midway1, so if you've
  already started a project, you should continue using Midway1.

## Introduction

We are transitioning from our private cluster (PPS) to [Midway][], a
University resource managed by the Research Computing Center
([RCC][]). For new or recently started projects, you should strongly
consider using Midway. In general, you should be able to find most
everything you need in the [User Guide][guide]. The purpose of this
document is to make the transition as easy as possible, and also
document procedures specific to how our lab uses Midway.

[midway]: https://rcc.uchicago.edu/resources/high-performance-computing
[rcc]: https://rcc.uchicago.edu/
[guide]: https://rcc.uchicago.edu/docs/

## Obtaining an account

To access RCC Midway, you need to fill out the [General User Account
Request][request] form. Our Principal Investigator Account Name is
pi-gilad. This will send an email to Yoav for approval. Once you
receive the confirmation email, you can log into the cluster using
your CNet ID and password.

```
ssh <insert CNet ID>@midway.rcc.uchicago.edu
```

To log directly into Midway2, add the number 2 to the cluster name:

```
ssh <insert CNet ID>@midway2.rcc.uchicago.edu
```

[request]: https://rcc.uchicago.edu/getting-started/general-user-account-request

## Running jobs

Midway uses a different job scheduler than PPS. Specifically, PPS uses
[Sun Grid Engine][sge] and Midway uses [Slurm][]. To start an interactive
session like `ql`, type the following:

```
sinteractive
```

[sge]: http://star.mit.edu/cluster/docs/0.93.3/guides/sge.html
[Slurm]: http://slurm.schedmd.com/

Conveniently, you will be in the same working directory in which you
ran the command. By default, this requests a session with 1 GB of
RAM. You can specify the exact amount in MB uses the flag `--mem`:

```
# Request 4 GB
sinteractive --mem=4g
# Can specify as 4g, 4G, or 4000
```

To submit a batch job from a shell script similar to `qsub`:

```
# Equivalent of:
# qsub -l h_vmem=8g -cwd script.sh
sbatch --mem=8g script.sh
```

To view your jobs similar to `qstat`:

```
squeue -u <insert CNet ID>
```

To run a quick command without saving it to a file, use the `--wrap`
flag:

```
# Equivalent of:
# echo "pwd" | qsub -l h_vmem=8g -cwd
sbatch --mem=8g --wrap="pwd"
```

To cancel a job, use `scancel`:

```
# cancel a specific job by ID
scancel 12345
# cancel all your current jobs
scancel --user=<insert CNet ID>
```

By default, a job will be sent to Midway1 (even from a Midway2 login node). To
specifically send a job to Midway2, specify the partition:

```
sbatch --mem=8g --partition=broadwl script.sh
```

For more details, see the documentation page [Using Midway][jobs].

[jobs]: https://rcc.uchicago.edu/docs/using-midway/index.html#using-midway

## Managing your data

There are 4 main places to store your data: home, scratch, project and
project2.

1. Your home directory has a size limit of 25 GB, so it should be used
mainly for storing code, configuration files, and installed software.

2. If you are on Midway1, you have 100 GB of scratch space at
`/scratch/midway/<insert CNet # ID>/`. There is also a symlink in your home
directory named # `scratch-midway`. You should use this space for intermediate
data # files that can be easily reproduced. Do not store any raw data here. If
you are on Midway2, you have 100 GB of scratch space at
`/scratch/midway2/<insert CNet ID>/`, and there is no shortcut in the home
directory. You can only access the scratch space of the filesystem you are
currently on.

3. Everyone in our lab shares 11 TB of space in our project directory
`/project/gilad`. Currently it is not regularly backed up, so make sure this is
not the only place your have raw data stored. Once you start using Midway, you
should create your own directory. As a convention, please use your CNet ID as
the name of the directory.

```
mkdir /project/gilad/<insert CNet ID>
```

4. We have an additional 20 TB of space in `/project2/gilad`. This is the
   preferred location to store new data because it is regularly backed up. The
   old storage location `/project` is full, so it is no longer possible to
   purchase additional space there. The data in `/project2/gilad` is
   unaccessible from Midway1 compute nodes, so do not store data here if you are
   using Midway1. Please create a subdirectory using your CNet ID.

To view how much space you have available, run the command `quota`.

## Managing file permissions

To make it easier for all lab members to collaborate, you will need to
manually adjust the permissions for your files on Midway. If you are
unfamiliar with Unix file permissions, see this [quick
explanation][quick] (for more details see the [Wikipedia
page][wiki]). The directions below will set the permissions such that
anyone in our lab can read your files and execute the software you
install. However, they will not have write permission, i.e. no one can
edit, move, delete, or in anyway manipulate your files.

[quick]: http://www.thinkplexx.com/learn/article/unix/command/chmod-permissions-flags-explained-600-0600-700-777-100-etc
[wiki]: https://en.wikipedia.org/wiki/File_system_permissions

By default, your home directory is only viewable by you. To make your
home directory readable (but not writable) by all members of our lab,
follow these steps:

```
chgrp -R pi-gilad ~
chmod -R 750 ~
```

Your scratch space is also private by default. To make it readable
(but not writable) by all members of our lab, follow similar steps as
above:

```
chgrp -R pi-gilad /scratch/midway/<insert CNet ID>
chmod -R 750 /scratch/midway/<insert CNet ID>
```

The files in the personal directory you create in our shared lab space
are automatically readable and writable by anyone in the lab. This is
not ideal, because someone else could accidentally modify or delete
your files. Similar to above, you can fix this will the following:

```
chmod -R 750 /project/gilad/<insert CNet ID>
```

Unfortunately, as you create new files in the shared space, they will
also automatically be set so that any lab member can edit. Thus you
will want to periodically run the above command, especially after
creating lots of new files.

## Installing software

Midway uses [Environment Modules][modules] to manage software
installation. To see all the software they have available, run `module
avail`. To load some common software used by our lab, run the
following:

```
module load bedtools python R samtools subread
```

You could also add a line like this to your `.bashrc` file so that
this software is always loaded when you login to Midway. See the
documentation on [Software][] for more information.

If software that you need is not already available via `module`, then
you can simply install it locally for yourself and add it to your
`PATH` variable in your `.bashrc`. If you are having trouble
installing the software locally, or you think the software would be
useful for many Midway users, you can request that RCC staff install
it by emailing them at `help@rcc.uchicago.edu`.

[modules]: http://modules.sourceforge.net/
[software]: https://rcc.uchicago.edu/docs/software/index.html

## Accessing your files

The [User Guide][guide] has thorough documentation on how to transfer
files to and from Midway (see [Data Transfer][transfer]). Briefly,
because there is no tunneling like the PPS cluster, it is much easier
to transfer files via `scp`, WinSCP, or similar tools. You can also
mount your Midway directories using Samba. This works well, but the
main inconvenience is you have to sign into the [UChicago VPN][vpn]
when working off-campus.

[transfer]: https://rcc.uchicago.edu/docs/data-transfer/index.html
[vpn]: https://cvpn.uchicago.edu/

## RStudio Server

**Warning:** This service is currently unavailable becaues it was too
  unstable. RCC Staff is investigating how to best host this service. However in
  the future it will likely only be offered on Midway2.

The RCC runs an RStudio Server at
https://rstudio.rcc.uchicago.edu. This service is still in beta
testing, so it is not documented in the [User Guide][guide]. It is
running R 3.2.1. By default, it looks for packages installed in
`~/R_libs` and `/software/R-3.2-el6-x86_64/lib64/R/library`. To have a
consistent environment on Midway, you need to complete the following
two steps:

1. Add the line `module load R/3.2` to your `.bashrc` file so that you
are using the same version of R.

2. Create a file named `.Renviron` in your home directory. It should
contain the following text: `R_LIBS_USER=~/R_libs`.

To confirm you have everything set up correctly, you can run the
following commands. You should see the same results (with the
exception of your username instead of mine).

```
Rscript --version
# R scripting front-end version 3.2.1 Patched (2015-07-12 r68650)
Rscript -e ".libPaths()"
# [1] "/home/jdblischak/R_libs"
# [2] "/software/R-3.2-el6-x86_64/lib64/R/library"
```

## Jupyter Hub

The RCC recently started running a Jupyter Hub server. You can login with your
username and password at https://jupyter.rcc.uchicago.edu/hub. Note that it is
running Python 3.5.

```
>>> import sys
>>> print(sys.version)
3.5.2 |Anaconda 4.1.1 (64-bit)| (default, Jul 2 2016, 17:53:06)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)]
```

## SSH keys

Each time you login via `ssh`, you will be prompted to enter your
password. To avoid this requirement, you can generate [SSH
keys][ssh]. SSH keys are a form of encryption. You create a private
key which is stored on your local computer. You then provide your
public key to remote locations that you want to access without entering a
password (e.g. Midway, GitHub, etc.). When you login via `ssh`, it
will check to see if your local computer contains the private key that
matches the public key.

To create SSH keys, follow these [directions from GitHub][gh-ssh]. Once you
have a public and private key on your local computer, create a new file in your
home directory on Midway.

```
touch ~/.ssh/authorized_keys
```

Copy-paste the contents of `id_rsa.pub` on your local computer into this
new file on Midway. Now you can login to Midway from this local
computer without entering your CNet password. To set this up for an
additional local computer, you need to create a new set of keys on
that computer and then append the public key to the `authorized_keys`
file.

In general, it is advisable to keep your `.ssh` directory private,
which can be accomplished with the following:

```
chmod -R 700 ~/.ssh
```

This setup works for logging into both Midway1 and Midway2, so you only need to
perform this once per computer.

[ssh]: https://en.wikipedia.org/wiki/Secure_Shell#Key_management
[gh-ssh]: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

## Configuring Git

Git is one of the software modules that are automatically loaded when
you login to Midway.

```
$ module list
Currently Loaded Modulefiles:
 1) vim/7.4         3) emacs/24        5) env/rcc
 2) subversion/1.8  4) git/2.7         6) slurm/current
```

When you first start using Git on a new computer, it is recommended to
set a global username and email. These are used to attribute your
commits to you. For more instructions on getting started with Git, see
the [Software Carpentry Git lesson][swc-git].

```
$ git config --global user.name "First Last"
$ git config --global user.email "your.email@server.com"
```

[swc-git]: http://swcarpentry.github.io/git-novice/

## Getting help

If you can't find a solution in the [User Guide][guide], send an email
to `help@rcc.uchicago.edu` with a detailed description of your
problem. If your problem is specific to how we organize our shared lab
storage space, you can open an [Issue][] in this repository.

[issue]: https://github.com/jdblischak/giladlab-midway-guide/issues

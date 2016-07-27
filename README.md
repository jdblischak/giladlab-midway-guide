# Guide to using RCC Midway

John Blischak

2016-07-27

**Last updated:** 2016-07-27

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

[request]: https://rcc.uchicago.edu/getting-started/general-user-account-request

## Quick start guide

To get started immediately, first load some common software used by
our lab. Midway uses Environment Modules, which you can learn more
about below.

```
module load bedtools python R samtools subread
```

Midway uses a different job scheduler than PPS.  To start an
interactive session like `ql`, type the following:

```
sinteractive
```

Conveniently, you will be in the same working directory in which you
ran the command. By default, this requests a session with 1 GB of
RAM. You can specify the exact amount in MB uses the flag `--mem`:

```
# Request 4 GB
sinteractive --mem=4000
```

To submit a batch job from a shell script similar to `qsub`:

```
# Equivalent of:
# qsub -l h_vmem=8g -cwd script.sh
sbatch --mem=8000 script.sh
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
sbatch --mem=8000 --wrap="pwd"
```

## Managing file permissions

## SSH keys

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

## Getting help

If you can't find a solution in the [User Guide][guide], send an email
to `help@rcc.uchicago.edu` with a detailed description of your
problem. If your problem is specific to how we organize our shared lab
storage space, you can open an [Issue][] in this repository.

[issue]: https://github.com/jdblischak/giladlab-midway-guide/issues

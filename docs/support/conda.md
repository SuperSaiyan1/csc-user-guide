# Using Conda, best practices

*Juha Lento, 2019-05-21, CSC*

Please read the text, and do not immediately skip to the cooking book
instructions at the Examples chapter :)

## What is conda

[Conda] is a software install tool that can manage software dependencies. It is
a bit similar to yum or apt, plus python virtual environments, if you are
familiar with those. Conda

- can be used to install packages in any language
- works the same in Linux, Mac OS and Windows
- works the same in machines from laptops to large clusters
- packages contain pre-compiled binaries, and the recipe they we built with
- does not require administrator privileges to run, unlike yum and apt
- can create multiple separate software stack roots, called Conda environments
- is designed for single user usage

NOTE: The term "environment" relates to two separate concepts, here. The term
*(1) Conda environment*, similar to virtual environment in Python, refers to one
of the user's software stack root directories, basically. The activation of a
Conda environment simply means that the path to the `bin` directory
of a Conda software stack root is appended to the user's *(2) shell
environment*'s `PATH` environment variable.

## When to use Conda and what kind of software you should install with it?

Conda is well suited for installing desktop type software, and complex,
possibly conflicting, package dependencies in Python, LaTex, or R, for example.
It is designed to be used on personal software installs. Naturally, the software
installed with it can be made available to others, too. The configuration files, [environment.yaml], for the conda software environments can be easily shared with others.

### When to yum, apt, or brew

If you are running on a personal Linux machine, such as laptop, have
administrator privileges, and intend to run the program only on your personal
machine, using yum, apt or homebrew may be more convenient.

### When to build from the sources

If you are installing a MPI parallel and/or performance optimized application on
a HPC cluster, follow the instructions of the computing center about building
software from the sources. The software dependencies in HPC environments are
usually handled using environment module system. For development work, using a
laptop and possibly conda or yum/apt/homebrew for installing dependencies and
development tools, is still likely more convenient.

There are package and environment management tools for building and installing
HPC software from sources, such as [Spack]. Those are not as widely used as
Conda, but they do compile and optimize the software for the particular
architecture, and do installs system wide by default.

## Conda channels (package repositories)

Conda channels are similar to Linux distributions' repositories, such as Ubuntu,
CentOS or Debian. The most popular Conda channels are commercially maintained
Anaconda, and community maintained Conda-forge.

## How to install conda

Install Conda by downloading the suitable installer script from [Miniconda], see
detailed instructions in the Example chapter below. Miniconda contains only a
minimal set of packages, that allow you to run conda commands and install
additional packages from different channels.

All files created by the installer script go under the install root
`<condaroot>` that is given at install time. By default, which is also a
recommended practice, all files installed subsequently with conda go under
the same install root. The only exception is conda's configuration file,
`.condarc`, which goes under user's home directory if user saves any settings
with `conda config` command.

### Python versions in the install scripts

You can use python 3 version to install environments with python 2 interpreter,
and vice versa. I recommend downloading python 3 version, which by default
installs python 3 in Conda's `base` environment.

### Miniconda or Anaconda

On a personal workstation you can also install [Anaconda] distribution, which in
addition to minimal set of packages, installs also a large number of packages
from Anaconda channel. You can also install the same packages later from
Anaconda channel if you start with Miniconda. The only practical difference is
just in what is installed by default.

## How to find if there is a conda package for a software?

First, google `conda <name>`, where `<name>` is the name of the package. Similar
to using yum, apt or homebrew, guessing the name of the package that contains
particular application may be the most difficult task. If there is a Conda
package for the software, Google hits usually contain the exact names of the
packages and in which channel (repository) they can be installed from.

## What could possibly go wrong?

### Running different conda than you think

If you follow the example instructions below, conda command is actually set up
as a shell function, that refers to an environment variable `CONDA_EXE`. You can
see how it is defined with command `type conda`. Now, depending on the value of
the environment variable `PATH`, there might be other conda commands in the
path. For example, if you load bioconda module in taito.csc.fi, you may see the
following confusing output:

```bash
$ echo $CONDA_EXE
/wrk/jle/DONOTREMOVE/conda/miniconda3/bin/conda
$ which conda
/homeappl/appl_taito/bio/bioconda/miniconda2/bin/conda
```

The lesson here is that make sure you are using only a single conda setup at a
time.

### Shell initialization and other configuration files modifying user's shell environment

Many software install documentations and scripts, including Miniconda, give an
option of adding setup lines into user's shell initialization scripts,
`.bashrc`, `.profile`, etc, which modify user's shell environment so that a
particular software is automatically set up for each new shell or login. This is
convenient, but may lead to conflicts that are hard to find later. A safer
practice is to put all these setup commands in separate scripts, let's say into
user's `~/setup_scripts/` folder, and then explicitly source them only when
necessary.

### Messed up environment variables

There are many environment variables that are used to override or extend the
defaults in how commands and libraries are searched, or which directories
particular applications use. Examples of such environment variables are
`PATH`, `LD_LIBRARY_PATH`, `CONDA_*`, and `PYTHONPATH`.

As a general safe practice, try to rely on these environment variables as little
as possible. Some environment module systems, such as the one in taito.csc.fi,
do extensively depend on modifying the values of environment variables. In
taito.csc.fi it might be a good idea to run

```bash
module purge
```

before starting to use Conda.

### Mixing packages from different channels, or simply outdated packages

Please note that installing packages from different channels to a single Conda
environment does not always work. That is a bit similar to trying to mix
packages from Ubuntu and Debian. The solution is to simply set up separate Conda
environments for different tasks if in doubt about the compatibility.

Some of the smaller channels are not always up-to-date or properly maintained,
and packages from those may break your Conda environment. Fortunately you can do
rollbacks on Conda environments, or simply try new packages in testing/staging
environments before including them into your favorite environments.

### Sorting out configuration related problems

 The best friends to sort out conda configuration or shell environment related
 problems are the following commands:

```bash
# Are you using the version you think you are?
conda --version

# The single most useful command to check configuration settings?
conda info

# Is there something extra in the command or library search paths?
echo $PATH $LD_LIBRARY_PATH

# You may need to unload some modules?
module list

# Are some environment variables overriding the default conda configuration?
env | grep ^CONDA_

# Is something set up by default at every login or new shell?
cat ~/.bashrc ~/.bash_profile ~/.profile
```

### Sorting out unmaintained or otherwise broken packages

If you encounter a broken package, a package that does not have the feature you
need, or an outdated package, it is possible to re-build the package yourself.
The details of this are slightly out of the scope of this document, but building
Conda packages perfectly doable. Basically, you need to install `conda-build`
Conda package, modify the files `meta.yaml` and `build.sh` in the
`<condaroot>/pkgs/<package>/info/recipe` sub-folder in the Conda package.

## Examples

The examples below should work without modifications in taito.csc.fi. The basic
usage is the same in other machines, other clusters, laptops, etc.

### Installing conda

If you are planning to install Conda packages yourself, instead of using
applications that are installed with Conda by someone else, like bioconda or
geoconda in taito.csc.fi, I recommend installing a personal copy of Miniconda3
as

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -p $WRKDIR/DONOTREMOVE/miniconda3
```

All conda files will be installed under the chosen Conda root install directory,
here `$WRKDIR/DONOTREMOVE/miniconda3`, with the exception of `.condarc`, which
will be in the user's home directory. The option `-b` simply skips some
questions and adding fo the automatic initialization lines into user's
`.bash\_profile`.

The Conda install root directory contains basically the following
subdirectories:

- `/bin`, '/lib', ... the usual Linux directories for the Conda "base"
  environment
- `/envs` where all Conda environments will reside
- `/pkgs` Conda package cache

### Activating conda

If you installed Conda into directory `$WRKDIR/DONOTREMOVE/miniconda3`, you can
activate conda by sourcing the initialization script

```bash
source $WRKDIR/DONOTREMOVE/miniconda3/etc/profile.d/conda.sh
```

This simply sets couple of shell environment variables, a conda command as a shell function, and modifies prompt so that it shows the name of the currently activated Conda environment.

When activating a new conda install, it's a good idea to run

```bash
conda info
```

to verify that the Conda configuration is ok.

### Installing packages, named environments, environment.yaml

I recommend

1. installing all conda packages into named environments, which go under `/envs`
   subdirectory, instead of installing them into the base environment (conda
   install root), and
2. using environment.yaml configuration files instead of adding packages to
   environments directly from the command line.

In practice, you only need to create a single environment.yaml file for each
your environments, and then use a single conda command

```
conda env create -f <envname>.yaml
```

to create the whole environment.

Updating the packages, or adding new packages to the an existing environment is
done by modifying the environment.yaml file, and then running

```
conda env update -f <envname>.yaml
```

### Examples of environment.yaml files

As the first example, let's use the environment.yaml file [conda-docs-env.yaml],
that can be used to install [mkdocs] with couple of plugins:

```
name: docs
channels:
  - conda-forge
dependencies:
  - python
  - pip
  - pip:
      - mkdocs
      - pymdown-extensions
      - mkdocs-windmill
```

The first field, `name`, simply defines the name of the Conda environment. The
second field, `channels`, list from which channels the packages are pulled from
to this environment. Field `dependencies` lists the actual Conda packages that
are installed into the environment. Note, that Conda integrates nicely with
Python pip, and you can also include pip packages, that are installed using pip,
into the Conda environment (Sometimes you need to clean pip caches that are not
under Conda's control).

Conda uses a "channel priority" for determining where to install packages from,
i.e. it tries first to install packages from the first listed channel. If no
package versions are specified, Conda always installs the latest versions.

As a second, more complex example, let's look at an environment for C program development, defined in file [c-ide.yaml]

```yaml
name: c-ide
channels:
  - /wrk/jle/DONOTREMOVE/conda/channels/csc-forge-based
  - conda-forge
  - defaults
  - anaconda
dependencies:
  - git
  - font-ttf-source-code-pro
  - emacs
  - global
  - ctags
  - clangdev
  - cmake
  - make
```

The first listed channel is in a local directory. In this case it is used as a
repository for a self created package, here [GNU Global], which does not(?) have
an existing Conda package in Anaconda or Conda-forge repositories. Naturally
this environment can only be created in machine taito.csc.fi, if package
`global` is included.

Adding the environment.yaml file to the source repository of your project is
probably an excellent idea. This allows an easy way to replicate the same
environment in multiple machines. For example, you can do development
conveniently in a local machine and then copy the environment to production
platform. Also, you can easily share the environment with other developers.

### Removing unused packages

 Command

```bash
conda env remove -n <envname>
```

removes the named environment. Command

```bash
conda clean
```

removes unused packages from the local package cache `/pkgs`.

### Creating environments so that other user's can access them

Giving other users an access to your Conda environment is as easy as giving them
read access to the directory containing the environment, in principle. In
practice it is very easy to update packages, and then forget to give read access
to the updated files. Also, some additional considerations need to be made, if
multiple persons are maintaining the environment, and accidental overwrites
and other mistakes are to be avoided.

Probability of these mistakes can be minimized by creating a separate
project/Unix group and user accounts for environment maintainers, and then
performing the environment maintenance task within a special shell environment.
Some ideas for the shell environment setup can be found in file
[bash_profile_extras.sh].

[Conda]: https://docs.conda.io/en/latest
[environment.yaml]: https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file
[Spack]: https://spack.io
[Miniconda]: https://docs.conda.io/en/latest/miniconda.html
[Anaconda]: https://www.anaconda.com/distribution
[conda-docs-env.yaml]: conda-docs-env.yaml
[mkdocs]: https://www.mkdocs.org
[bash_profile_extras.sh]: bash_profile_extras.sh
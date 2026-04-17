---
title: Accessing software via Modules
teaching: 30
exercises: 15
---



::::::::::::::::::::::::::::::::::::::: objectives

- Load and use a software package.
- Explain how the shell environment changes when the module mechanism loads or unloads packages.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do we load and unload software packages?

::::::::::::::::::::::::::::::::::::::::::::::::::

On a high-performance computing system, it is seldom the case that the software
we want to use is available when we log in. It is installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two well known examples are Python and C compiler versions.
Python 3 famously provides a `python` command that conflicts with that provided
by Python 2. Software compiled against a newer version of the C libraries and
then run on a machine that has older C libraries installed will result in an
opaque `'GLIBCXX_3.4.20' not found` error.

Software versioning is another common issue. A team might depend on a certain
package version for their research project -- if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allows a set of researchers to
prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may require a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work.

## Environment Modules

Environment modules are the solution to these problems. A *module* is a
self-contained description of a software package -- it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the *man*
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:


```bash
[user@fugg1 ~]$ module avail | less
```

```output

------------------------------------ /beegfs/Tools/easybuild/meta-module ------------------------------------
   2019b-rpath      2021a-norpath    2021a        2022a        2023a
   2020b-norpath    2021a-rpath      2021a_AL9    2022a_AL9    2025

-------------------------------------- /opt/lmod/lmod/modulefiles/Core --------------------------------------
   lmod    settarg

----------------- This is a list of module extensions "module --nx avail ..." to not show.
 -----------------
    ADGofTest                         (E)     flit                          (E)
    AICcmodavg                        (E)     flit-scm                      (E)
    AMAPVox                           (E)     flit_core                     (E)
    AUC                               (E)     flit_scm                      (E)
    AlgDesign                         (E)     fma                           (E)
    Algorithm::Dependency             (E)     fmri                          (E)
    Algorithm::Diff                   (E)     fontBitstreamVera             (E)
    AnyEvent                          (E)     fontLiberation                (E)
    App::Cmd                          (E)     fontawesome                   (E)
    App::cpanminus                    (E)     fontquiver                    (E)
    AppConfig                         (E)     fonttools                     (E)
    Archive::Extract                  (E)     forcats                       (E)
    Array::Transpose                  (E)     foreach                       (E)
    Array::Utils                      (E)     forecast                      (E)
    Authen::NTLM                      (E)     foreign                       (E)
    Authen::SASL                      (E)     formatR                       (E)
    AutoLoader                        (E)     formula.tools                 (E)
    B::COW                            (E)     fossil                        (E)
    B::Hooks::EndOfScope              (E)     fpc                           (E)
    B::Lint                           (E)     fpp                           (E)
    BB                                (E)     fracdiff                      (E)
lines 1-31

    fitdistrplus                      (E)     zip                           (E)
    flashClust                        (E)     zipfile36                     (E)
    flexclust                         (E)     zipp                          (E)
    flexmix                           (E)     zoo                           (E)
    flextable                         (E)

[removed most of the output here for clarity]

These extensions cannot be loaded directly, use "module spider extension_name" for more information.

  Where:
   E:  Extension that is provided by another module

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

Note that piping the output through `less` allows us to search within the output using the <kbd>/</kbd> key.

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so.

```bash
[user@fugg1 ~]$ module list
```

```output
No Modulefiles Currently Loaded.
```

## Loading and Unloading Software

To load a software module, use `module load`.

In this example we will use Python 3. Initially, it is not loaded.
We can test this by using the `which` command. `which` looks for
programs the same way that Bash does, so we can use it to tell us
where a particular piece of software is stored.

```bash
[user@fugg1 ~]$ which python3
```


If the `python3` command was unavailable, we would see output like

```output
/usr/bin/which: no python3 in (/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin:/opt/software/slurm/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/user/.local/bin:/home/user/bin)
```

Note that this wall of text is really a list, with values separated
by the `:` character. The output is telling us that the `which` command
searched the following directories for `python3`, without success:

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin
/opt/software/slurm/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
/opt/puppetlabs/bin
/home/user/.local/bin
/home/user/bin
```

However, in our case we do have an existing `python3` available so we see

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin/python3
```

We need a different Python than the system provided one though, so let us load
a module to access it.

We can load the `python3` command with `module load`:


```bash
[user@fugg1 ~]$ module load Python
[user@fugg1 ~]$ which python3
```

```output
```

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH`
environment variable. `$PATH` is a special environment variable that controls
where a UNIX system looks for software. Specifically `$PATH` is a list of
directories (separated by `:`) that the OS searches through for a command
before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```bash
[user@fugg1 ~]$ echo $PATH
```

```output
/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/SQLite/3.31.1-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Tcl/8.6.10-GCCcore-x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/GCCcore/x.y.z/bin:/cvmfs/pilot.eessi-hpc.org/2020.12/compat/linux/x86_64/usr/bin:/opt/software/slurm/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/user01/.local/bin:/home/user01/bin
```

You'll notice a similarity to the output of the `which` command. In this case,
there's only one difference: the different directory at the beginning. When we
ran the `module load` command, it added a directory to the beginning of our
`$PATH` -- or "prepended to PATH". Let's examine what's there:


```bash
[user@fugg1 ~]$ ls /cvmfs/pilot.eessi-hpc.org/2020.12/software/x86_64/amd/zen2/software/Python/3.x.y-GCCcore-x.y.z/bin
```

```output
2to3              nosetests-3.8  python                 rst2s5.py
2to3-3.8          pasteurize     python3                rst2xetex.py
chardetect        pbr            python3.8              rst2xml.py
cygdb             pip            python3.8-config       rstpep2html.py
cython            pip3           python3-config         runxlrd.py
cythonize         pip3.8         rst2html4.py           sphinx-apidoc
easy_install      pybabel        rst2html5.py           sphinx-autogen
easy_install-3.8  __pycache__    rst2html.py            sphinx-build
futurize          pydoc3         rst2latex.py           sphinx-quickstart
idle3             pydoc3.8       rst2man.py             tabulate
idle3.8           pygmentize     rst2odt_prepstyles.py  virtualenv
netaddr           pytest         rst2odt.py             wheel
nosetests         py.test        rst2pseudoxml.py
```

Taking this to its conclusion, `module load` will add software to your `$PATH`.
It "loads" software. A special note on this - depending on which version of the
`module` program that is installed at your site, `module load` will also load
required software dependencies.


To demonstrate, let's use `module list`. `module list` shows all loaded
software modules.

```bash
[user@fugg1 ~]$ module list
```

```output
No modules loaded
```

```bash
[user@fugg1 ~]$ module load 2025 GCC/14.3.0
[user@fugg1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) 2025   2) GCCcore/14.3.0   3) zlib/1.3.1   4) binutils/2.44   5) GCC/14.3.0

```

So in this case, loading the `GCC/14.3.0` module (loads other dependent module such as GCCcore, zlib, bunutils and GCC).
Let's try unloading the `GCC/14.3.0` package.

```bash
[user@fugg1 ~]$ module unload GCC/14.3.0
[user@fugg1 ~]$ module list
```

```output
Currently Loaded Modules:
  1) 2025
```

So using `module unload` "un-loads" a module, and depending on how a site is
configured it may also unload all only specific module (in our case it unloads
all the dependent modules).
If we wanted to unload everything at once, we could run `module purge`
(unloads everything).

```bash
[user@fugg1 ~]$ module purge
[user@fugg1 ~]$ module list
```

```output
No modules loaded
```

Note that `module purge` is informative. It will also let us know if a default
set of "sticky" packages cannot be unloaded (and how to actually unload these
if we truly so desired).

Note that this module loading process happens principally through
the manipulation of environment variables like `$PATH`. There
is usually little or no data transfer involved.

The module loading process manipulates other special environment
variables as well, including variables that influence where the
system looks for software libraries, and sometimes variables which
tell commercial software packages where to find license servers.

The module command also restores these shell environment variables
to their previous state when a module is unloaded.

## Software Versioning

So far, we've learned how to load and unload software packages. This is very
useful. However, we have not yet addressed the issue of software versioning. At
some point or other, you will run into issues where only one particular version
of some software will be suitable. Perhaps a key bugfix only happened in a
certain version, or version X broke compatibility with a file format you use.
In either of these example cases, it helps to be very specific about what
software is loaded.

Let's examine the output of `module avail` more closely, using the pager since
there may be reams of output:


```bash
[user@fugg1 ~]$ module avail | less
```

```output

------------------------------------ /beegfs/Tools/easybuild/meta-module ------------------------------------
   2019b-rpath      2021a-norpath    2021a        2022a        2023a
   2020b-norpath    2021a-rpath      2021a_AL9    2022a_AL9    2025

-------------------------------------- /opt/lmod/lmod/modulefiles/Core --------------------------------------
   lmod    settarg

----------------- This is a list of module extensions "module --nx avail ..." to not show.
 -----------------
    ADGofTest                         (E)     flit                          (E)
    AICcmodavg                        (E)     flit-scm                      (E)
    AMAPVox                           (E)     flit_core                     (E)
    AUC                               (E)     flit_scm                      (E)
    AlgDesign                         (E)     fma                           (E)
    Algorithm::Dependency             (E)     fmri                          (E)
    Algorithm::Diff                   (E)     fontBitstreamVera             (E)
    AnyEvent                          (E)     fontLiberation                (E)
    App::Cmd                          (E)     fontawesome                   (E)
    App::cpanminus                    (E)     fontquiver                    (E)
    AppConfig                         (E)     fonttools                     (E)
    Archive::Extract                  (E)     forcats                       (E)
    Array::Transpose                  (E)     foreach                       (E)
    Array::Utils                      (E)     forecast                      (E)
    Authen::NTLM                      (E)     foreign                       (E)
    Authen::SASL                      (E)     formatR                       (E)
    AutoLoader                        (E)     formula.tools                 (E)
    B::COW                            (E)     fossil                        (E)
    B::Hooks::EndOfScope              (E)     fpc                           (E)
    B::Lint                           (E)     fpp                           (E)
    BB                                (E)     fracdiff                      (E)
lines 1-31

    fitdistrplus                      (E)     zip                           (E)
    flashClust                        (E)     zipfile36                     (E)
    flexclust                         (E)     zipp                          (E)
    flexmix                           (E)     zoo                           (E)
    flextable                         (E)

[removed most of the output here for clarity]

These extensions cannot be loaded directly, use "module spider extension_name" for more information.

  Where:
   E:  Extension that is provided by another module

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

If the software your Slurm script runs requires on a specific version
of a dependency, make sure you use the full name of the module, rather
than the _default_ loaded when you give only its name (up to the first
slash).

:::::::::::::::::::::::::::::::::::::::  challenge

## Using Software Modules in Scripts

Create a job that is able to run `python3 --version`. Remember, no software
is loaded by default! Running a job is just like logging on to the system
(you should not assume a module loaded on the login node is loaded on a
compute node).

:::::::::::::::  solution

## Solution

```bash
[user@fugg1 ~]$ nano python-module.sh
[user@fugg1 ~]$ cat python-module.sh
```

```output
#!/bin/bash
#SBATCH 
r config$sched$comment` -t 00:00:30

module load Python

python3 --version
```

```bash
[user@fugg1 ~]$ sbatch  python-module.sh
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Load software with `module load softwareName`.
- Unload software with `module unload`
- The module system handles software versioning and package conflicts for you automatically.

::::::::::::::::::::::::::::::::::::::::::::::::::

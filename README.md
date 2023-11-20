# GridapWorkshop2023Material

The practical part of the workshop will consist of instructors' guided hands-on tutorials and exercises.
This will be carried out  either
on the attendees laptops (mostly first day) or the [Gadi supercomputer](https://opus.nci.org.au/display/Help/Gadi+User+Guide) at 
NCI (mostly second day). You will find below the instructions to set up the software environment required in both cases.
For the Gadi supercomputer, you will get an invitation from the instructors to create an account prior to the 
workshop.

## Required software

Before being able to work on the workshop material, you will need to install the following software on your laptop:

- Install [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), the version control system we will use. **For Windows users, we strongly recommend installing [git for Windows](https://gitforwindows.org/). This will also install a bash shell, which will allow you to follow the rest of the installation instructions verbatim.**
- Download and install Julia based on the platform you are using from the [Julia](https://julialang.org/downloads/platform/) home page.
- Download and install VSCode based on the platform you are using from the [VSCode](https://code.visualstudio.com/download) home page. 
- [Install](https://www.julia-vscode.org/docs/dev/gettingstarted/#Installing-the-Julia-extension) and [configure](https://www.julia-vscode.org/docs/dev/gettingstarted/#Configuring-the-Julia-extension) the [Julia extension for VSCode](https://code.visualstudio.com/docs/languages/julia). Some interesting features of the Julia extension for VSCode are covered in the following [YouTube](https://www.youtube.com/watch?v=IdhnP00Y1Ks&t=125s) video.
- Install [Paraview](https://www.paraview.org/download/) post-processing software. We will use the basic features of ParaView. In any case, if you are 
  keen on learning more,  there is a whole [YouTube channel](https://www.youtube.com/playlist?list=PLvkU6i2iQ2fpcVsqaKXJT5Wjb9_ttRLK-) on Paraview that will give you many more ideas. 
- Finally, you will need an **ssh** client to connect to Gadi. Generally, every modern OS should have one installed by default. To check if you have one, open a terminal and type `ssh`. If a message like `usage: ssh ...` appears, you are good to go.

## Getting the workshop material

The workshop material is available as a git repository [here](https://github.com/gridap/GridapWorkshop2023Material). You can either download it as a zip file or clone the repository using git. We strongly recommend the latter as you will be able to automatically pull the most up-to-date changes as per required.

If your system has an ssh client, you can clone the repository using the following command

```bash
git clone git@github.com:gridap/GridapWorkshop2023Material.git
```

from the terminal. Alternative methods to clone the repository can be found [here](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository).

## Setting up the environment on your local computer from the terminal

Move into the newly cloned repository and start Julia from the terminal by typing

```bash
julia --project=.
```

Then, press `]` to enter the package manager and run

```julia
(GridapWorkshop) pkg> instantiate
```

to install and precompile all the packages needed for the workshop. This may take a while.

More information on Julia Environments can be found [here](https://pkgdocs.julialang.org/v1/environments/).

## Setting up the environment on your local computer with VSCode

1. Open VSCode. Then, on the top menu, select `File->Open Folder`, and select the workshop's material folder you just cloned.
2. Ensure that the Julia environment in the bottom status bar of VSCode is `GridapWorkshop`. Click [here](https://www.julia-vscode.org/docs/dev/userguide/env/#Julia-Environments) for instructions on how to do that.
3. Open the Julia REPL in VSCode. To this end, open the command palette with the keyboard key combination `Crtl+Shift+P`.
4. On the command palette, type `"julia"`. You should get a drop-down list with different options. Select `Julia: Start REPL` option. This should open the Julia REPL on the VSCode's terminal window at the bottom.
5. Run the `instantiate` package manager command as described in the previous section. 

## Setting up the environment on Gadi

First, we will need to log into Gadi. You will receive an email at some point with an invitation to create an account. At the end of the process, you will get a username and password (which you should change). If your username is `aaa777`, you can connect to Gadi by typing

```bash
ssh aaa777@gadi.nci.org.au
```

When prompted, enter your password. You should now be logged into Gadi and located in your home directory. We would also recommend setting up passwordless ssh access to Gadi (but it is not required).

In addition to your home directory, you have access to a scratch space, a project-wide shared filesystem that is optimized for parallel access. This is where we will be working during the workshop. Start by creating a personal folder within the project's scratch space and linking it back to your home directory:

```bash
mkdir /scratch/vp91/$USER
ln -s /scratch/vp91/$USER $HOME/scratch
```

Move into your newly created scratch directory and clone the workshop repository as described above. Once the workshop repository is cloned, move into the `/gadi` subdirectory. This directory contains the distributed codes we will be using. Load the Gadi environment by running

```bash
source modules.sh
```

The script `modules.sh` has several purposes, and needs to be sourced every time you log in. It loads the Julia module, the Intel-MPI modules, and sets up some environment variables we will need.

Next, repeat the steps described in the previous section to setup the Julia environment in serial.
In addition, you will need to setup the environment for parallel. To do so, run the following commands on the terminal:

```bash
julia --project=. -e 'using MPIPreferences; MPIPreferences.use_system_binary()'
julia --project=. -e 'using Pkg; Pkg.build()'
```

The first line sets up `MPI.jl` to work with the Intel-MPI binaries installed on Gadi instead of the Julia-specific artifacts. The second line does the same for `GridapPETSc.jl` and `GridapP4est.jl`, which are the Gridap wrappers for PETSc and P4est, respectively.

## Creating a system image

Unfortunately, there is distributed version of the Julia REPL. This means running MPI codes interactively is not possible. Moreover, Julia notoriously suffers from long TTFX (Time To First eXecution) times due to Just-In-Time compilation. Although this problem is being the focus of the latest releases, it can still be tedious to work within an edit-run-debug cycle.

To alleviate this problem, we will create a system image that contains all the packages we will need during the workshop. This will allow us to start Julia with the system image preloaded, and thus avoid the long TTFX times.

The `/gadi/compilation` directory contains files that allow us to do just that using [`PackageCompiler.jl`](https://julialang.github.io/PackageCompiler.jl/stable/). Although we will be creating a sysimage (to be used with Julia), this package also has options to create dynamic libraries (to be used with C/C++/Fortran) and executables.

First, we will need to install `PackageCompiler` package. It does not need to be installed within the project, so just run

```bash
julia -e 'using Pkg; Pkg.add("PackageCompiler")'
```

Next, run the following commands from the `/gadi` directory to create the system image

```bash
julia --project=. compilation/compile.jl
```

This will take a while (around 5/6 mins), and will create a system image `GadiTutorial.so` in the `/gadi` directory. You can then test this sysimage by running

```bash
julia --project=. -JGadiTutorial.so compilation/warmup.jl
```

Alternatively, we have also provided a BATCH script that creates the system image on a compute node. This is useful if the architecture of the compute nodes is different from the login nodes (for instance, when running on GPU nodes). It is also good practice to not run heavy computations on the login nodes. To run compile remotely, run

```bash
qsub compilation/compile.sh
```

You can see the status of the job by running `qstat`.

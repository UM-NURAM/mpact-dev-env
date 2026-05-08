# Quickstart
If you think you have an environment with all prerequisites satisfied, and hate reading, then you can try to simply install the TPLs with
```
$ git clone https://github.com/UM-NURAM/mpact-dev.git
$ cd mpact-dev/mpact_tpls
$ cmake -S . -B build -D CMAKE_INSTALL_PREFIX=<path to tpl install dir> -D MPACT_TPLs_BUILD_PARALLEL=4
$ cmake --build build
```

Note that you should specify the directory in which to install the TPLs **`<path to tpl install dir> `**.
Also note **it will take more than 2 hours** with 4 processors to build and install the TPLs.

Once the build step is complete, the TPLs are installed and it is safe to delete the build directory.
```
rm -rf build
```
Finally load the TPLs into your environment with
```
source <path to tpl install dir>/load_dev_env.sh
```

# Introduction
This is an open source project for setting up a development and runtime environment suitable for MPACT.
Presently, it provides instructions and information about compatible operating systems and prerequisite software for the recommended Third Party Libraries (TPLs) to build MPACT.
For the most part, the prerequisites are recommended to be handled by the OS package manager and the TPLs are to be installed from source using the instructions here.

There are also dependencies on Python 3 and several Python packages.
Here it is also recommended to install Python 3 and the requisite packages locally, rather than at the OS level.

**It is highly recommended to use the versions of the compilers and TPLs specified here.**
While it is possible to use other compilers or libraries and successfully build MPACT, specific vendors and versions not listed here are not extensively tested.
We currently find the overall software stack for MPACT to be somewhat fragile, therefore significantly more effort should be expected to install MPACT when deviating from these specifications.
The current TPL software stack installed by this project is
| Package | Version | SHA1 |
| :------ | ------: | :-----: |
| `mpich` | 4.2.3   | 1be671c6f1293ab7f4dbf7ac71c3890de3dabff0 |
| `hdf5`  | 1.10.1  | 73b77a23ca099ac47d8241f633bf67430007c430 |
| `lapack`| 3.7.1   | 84c4f7163b52b1bf1f6ca2193f6f48ed3dec0fab |
| `petsc` | 3.13.6  | 2b6475e092356f4cb67afccb755a853db6eb5285 |
| `slepc` | 3.13.4  | 2dbc73a17adc87a36d52c991bcbfa81db036454b |

This project can also optionally install `libpng-1.6.55`, `pugixml-1.15`, and `gmsh-4.12.2`.


Lastly, in the near future, we plan to incorporate Singularity container definitions to be able to build your own Singularity container.
As an alternative to building these containers, you will eventually be able to pull the container yourself.

## Suggested Operating Systems
MPACT is a command line console program designed to run under Linux.
The recommended Linux distributions and versions are provided in the table below.

| Linux Distribution | Version |
| :----------------- | ------: |
|  Ubuntu            | 24.04   |
| Rocky              | 8.10    |

On windows machines, using Windows Subsystem for Linux (WSL) with one of the above Linux distributions is recommended.
macOS and arm64 architectures are not regularly tested, therefore no instructions exist for this OS at this time.

If you are unsure about which version of Linux you are using, you can check your Linux operating system using the following command.
```
cat /etc/os-release
```

# Prerequisites
The software necessary to build MPACT and its recommended TPLs is given in the following table.

| Package | Minimum Version | Ubuntu 24.04 default | Rocky 8.10 Default |
| :------ | --------------: | -------------------: | -------------: |
| git      | 1.8.3          | 2.43.0               | 2.43.7         |
| cmake    | 3.18           | 3.28.3               | 3.26.5         |
| GNU Make | 3.82           | 4.3                  | 4.2.1          |
| gcc      | 8.3.0          | 13.3.0               | 8.5.0          |
| g++      | 8.3.0          | 13.3.0               | 8.5.0          |
| gfortran | 8.3.0          | 13.3.0               | 8.5.0          |
| Perl     | 5.16.3         | 5.38                 | 5.26.3         |
| Python3  | 3.6.4          | 3.12.3               | 3.6.8          |

As mentioned previously, in addition to a Python 3 interpreter, there are a few required packages
 * `numpy`
 * `matplotlib`
 * `pandas`
 * `h5py`
 * `xdrlib` (required by recommended version of PETSc)

An important note about `xdrlib` is that it was deprecated in Python 3.11 and removed in Python 3.13.
If you are using a version of Python equal to greater than 13.0 you will need to install the `standard-xdrlib` python package.

Lastly, some other packages that are highly recommended but not required are:
 * `which`
 * `vim`
 * `htop`
 * `texlive-full`
 * `doxygen`
 * `pugixml`
 * `libpng`
 * `python3-venv`
 * `python3-pip`

The `python3-pip` and `python3-venv` are to facilitate setting up a Python 3 virtual environment in which you can easily install Python packages.
These are required for Ubuntu-24.04.
As an alternative however, you may wish to use your own local installation of Python 3, like those from [`anaconda`](https://www.anaconda.com/download) or [`miniconda`](https://www.anaconda.com/docs/getting-started/miniconda/main).
In fact, this may be preferred in most cases.
This is totally fine and should not create any issues, but if you use a non-OS distribution of Python, pay careful attention to the [python package installation instructions](#other-python-3-distributions).

## Installing prerequisites (10 minutes)

### Ubuntu-24.04

```
sudo apt-get update
sudo apt-get install -y gcc g++ gfortran cmake make git perl python3 python3-venv python3-pip which vim libpng-dev
```

### RockyLinux-8.10

```
sudo dnf update
sudo dnf install -y gcc gcc-c++ gcc-gfortran cmake make git perl which vim libpng-devel
```

## Installing python prequisite packages (5 minutes)

### Ubuntu-24.04
Assuming you have installed `python3` along with `venv` and `pip`, then the recommended way to proceed is to create a Python 3 virtual environment.
This is because on Ubuntu-24.04 the Python environment is marked as "externally managed" and this prevents `pip` from installing packages at the system or user site-package levels.
This can be overriden, but may cause you more trouble than you want, so this why a virtual environment is recommended.
```
python3 -m venv ~/.venv/mpact-env
source ~/.venv/mpact-env/bin/activate
python3 -m pip install numpy matplotlib pandas h5py
```

If your Ubunut environment has `python3` but does not have `venv` or `pip` and you do not have admin rights, then its recommended you install your own Python 3 distribution like anaconda or miniconda.

### RockLinux-8.10
Assuming you have `python3` installed by the OS package manager, then the following command should work.
```
python3 -m pip install --user numpy matplotlib pandas h5py
```
### Other Python 3 Distributions
If you are using an anaconda/miniconda distribution of python then you can install the packages with conda using
```
conda install numpy matplotlib pandas h5py
```
Note that you can also do this in a custom conda environment by doing the following **first**.
```
conda env create mpact-env
conda activate mpact-env
```

# Installing the Third Party Libraries (2 to 4 hours)
Once you have all the prerequisites installed, then the next steps should be fairly automatic.
At this stage it is just the instructions in the [quickstart](#quickstart).

## CMake Options
All of the CMake options supported by this project are echoed to the screen during the configure.
The default configuration is
```
    MPACT_TPLs_INSTALL_MPI     = ON
    MPACT_TPLs_INSTALL_HDF5    = ON
    MPACT_TPLs_INSTALL_LAPACK  = ON
    MPACT_TPLs_INSTALL_PETSC   = ON
    MPACT_TPLs_INSTALL_SLEPC   = ON
    MPACT_TPLs_INSTALL_PNG     = OFF
    MPACT_TPLs_INSTALL_PUGIXML = OFF
    MPACT_TPLs_INSTALL_GMSH    = OFF
    MPACT_TPLs_LOG_TO_SCREEN   = OFF
    MPACT_TPLs_BUILD_PARALLEL  = 4
    CMAKE_INSTALL_PREFIX       = /usr/local
```
Any option can be set by adding `-D <OPTION>=<VALUE>` when invoking the configure step:
```
cmake -S . -B build
```

## Testing changes to packages and versions

---
title: "Running Singularity containers with GPUs"
teaching: 30
exercises: 20
questions:
# - "Can I run MPI parallel codes from Singularity containers on a local/institutional/national HPC platform?"
- "How do I set up and run with GPUs from a Singularity container?"
objectives:
- "Learn how GPU applications within Singularity containers can be run on HPC platforms"
keypoints:
- "Singularity images require special access to GPUs due to dependency on libraries on the host operating system."
- "Understand the dependency between host operating system files and GPU runtime libraries in the container."
---

## Running Singularity containers with GPUs

### GPU overview

GPUs are dedicated pieces of hardware that can perform certain jobs very quickly.  On our supercomputer we have a
number of GPU nodes that are based on Nvidia Tesla P100 and V100 technology.  To access these cards there is a kernel
driver to interface with the hardware and libraries that communicate between application and kernel.

Singularity support of [GPUs](https://sylabs.io/guides/3.7/user-guide/gpu.html) includes Nvidia and AMD.

### GPU codes with Singularity containers

We've already seen that building Singularity containers can be impractical without root access. Since we're highly unlikely to have root access on a large institutional, regional or national cluster, building a container directly on the target platform is not normally an option.

Singularity attempts to make available in the container all the files and devices required for GPU support.  However,
when building the image we can use some knowledge and install, for example, Nvidia CUDA libraries if we know it will
support the system we are targetting.  Anaconda for example installs CUDA runtime package for code that uses it.

### Building and running a Singularity image with GPU support

Let us try running Pytorch with GPU support inside an Anaconda environment.  The key option to supply `singularity` is
the `--nv` argument to bring into the container all the required files to access the GPU.  For example

~~~
$ singularity run --nv my_image.sif
~~~
{: .language-bash}

#### Building and testing an image

Lets create a directory and within the directory create a `.def` file with the following contents.

~~~
Bootstrap:docker  
From:continuumio/miniconda:4.7.12

%labels
MAINTAINER Thomas Green

%environment

%runscript
. /etc/profile
conda activate pytorch
exec python3

%post
# Create some common mountpoints for systems without overlayfs
mkdir /scratch
mkdir /apps

. /etc/profile
conda create --name pytorch
conda activate pytorch
conda install pytorch torchvision torchaudio -c pytorch
~~~
{: .output}

A quick overview of what the above definition file is doing:

 - The image is being bootstrapped from the `miniconda` Docker image.
 - In the `%post` section:
   - Create some local bind locations. Backwards support for systems not supporting overlayFS.
   - Source the `/etc/profile` to bring in the environment.
   - Run the `conda` commands.
 - In the `%runscript` section: A runscript is set up to run Python.

> ## Build and test the Pytorch image
>
> Using the above definition file, build a Singularity image named `pytorch.sif`.
> 
> Once you have built the image, use it to run python and try using Pytorch and run [example code](https://pytorch.org/tutorials/beginner/pytorch_with_examples.html)
> 
> > ## Solution
> > 
> > You should be able to build an image from the definition file as follows:
> > 
> > ~~~
> > $ singularity build --remote pytorch.sif pytorch.def
> > ~~~
> > {: .language-bash}
> > 
> > If successfully built we can try running the container.
> > ~~~
> > $ singularity run --nv pytorch.sif
> > ~~~
> > {: .language-bash}
> > 
> {: .solution}
{: .challenge}

This has demonstrated that we can successfully run GPU code from within a Singularity container.

## Singularity wrap-up

This concludes the 5 episodes of the course covering Singularity. We hope you found this information useful and that it has inspired you to use Singularity to help enhance the way you build/work with research software.

As a new set of material, we appreciate that there are likely to be improvements that can be made to enhance the quality of this material. We welcome your thoughts, suggestions and feedback on improvements that could be made to help others making use of these lessons.

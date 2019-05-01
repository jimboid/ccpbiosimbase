============================
The CCPBioSim base container
============================

This container forms the basis of our cloud based training platform. It is
designed to be very minimal such that it only provides basic system utilities
and tools along with the JupyterHub server for serving multiuser Jupyter
notebooks.

This particular container does not contain any Jupyter based training material,
but simply sets up and configures the JupyterHub server and a number of basic
system utilities that will enable this container to function as a reliable base
container for specific workshop courses. A description of what is installed and
how to use this base is below.

Container Contents
------------------

**Base operating system** Ubuntu Bionic Beaver (18.04)

**Installed via apt**

    - bc
    - bzip2
    - ca-certificates
    - cmake
    - fonts-liberation
    - g++
    - gcc
    - gfortran
    - git
    - less
    - libgfortran3
    - locales
    - make
    - nano
    - openssh-client
    - python3
    - python3-pip
    - python3-setuptools
    - rsync
    - sudo
    - wget
    - zlib1g-dev

**Installed manually**

    - tini
    - miniconda

**Installed via miniconda**

    - notebook-5.2.x
    - jupyterhub-0.8.x
    - jupyterlab-0.31.x
    - tornado-4.5.3

**Installed via pip**

    - jupyterhub-tmpauthenticator


How to Use
----------

This container is not designed to be deployed by itself, as it has no notebooks
installed. It is designed to be used as a base container for other containers
which will contain training material and any specific software libraries that 
these notebooks use. This base image is already built and available on Docker
Hub and to include it in your project you only need add the line::

    FROM ccpbiosim/ccpbiosimbase:latest

to your Dockerfile. This will then pull this base container from Docker Hub and
build your container on top of it. So you should add the code to copy files,
install extra software packages or do environment configuration as part of your
own Dockerfile.

If you need to install software as root then you can change to the root user by
adding::

    USER root

to your dockerfile followed by the commands you need to run. You should always
try to aim to minimise the root operations by changing back to the default user,
and always end your dockerfile by changing back to the default user regardless
of whether it has been done elsewhere in the dockerfile, this helps reduce
mistakes. You do this by::

    USER $NB_USER

A great deal of software packages can be installed via pip or conda. When using
pip it is a good idea to let it install to the local user python site packages
so to avoid privilege issues, for example::

    RUN pip install --user pandas

Copying your notebook files is very simple. You should simply place your
notebooks in the same directory as the dockerfile and then include something
like::

    ADD --chown=jovyan:100 *.ipynb $HOME/
    ADD --chown=jovyan:100 data $HOME/data

The --chown part is important as this will change the ownership of your files
from the root user to the notebook user. Unfortunately for the time being the
--chown flag on ADD and COPY directives in docker do not expand environment
variables. This is a requested feature in the docker development programme at
the time of writing of this document. 

Editors
-------

By default the base container has nano installed. There is no reason that you
can't install other editors in your container. But to keep things to a minimum
we are only supplying nano in the base container. To install others, simply
change to root user and run a 'sudo apt install <editor>' and change back to 
$NB_USER.

Ports
-----

In our containers we are using the JupyterHub default port 8888, so you should
forwards this port when deploying locally::

    docker run -p 8888:8888 ccpbiosim:ccpbiosimbase

Contributing
------------

Whilst as a consortium we are open to contributions to our software tools. In
this case due to the fact that one of our services is built on top of this
container, we will be rigorous with defining what should and should not form
the base container. Suggestions and contributions are welcome, and they will be
scrutinised via code review prior to acceptance.

Known Problems
--------------

Packages that have tornado as a dependency and change the version installed
above will cause the Jupyter notebook server to have connection problems with
the Jupyter kernel. This happens irrespective of whether it is installed via
conda or pip. So when installing software in your container, be vigilant with
this. 


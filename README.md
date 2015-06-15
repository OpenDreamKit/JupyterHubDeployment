Notes, files, ... around the deployment of JupyterHub instances with various math software
==========================================================================================

So far those are some notes we took with Thierry Dumont while playing
around with JupyterHub and Sage.

# Basic installation on Ubuntu 15.04

- Follow the instructions on https://github.com/jupyter/jupyterhub,
  replacing `pip` by `sudo pip3` in each command

- `sudo pip3 install zmq` complains with:

      build/temp.linux-x86_64-3.4/scratch/vers.c:4:17: fatal error: zmq.h: Aucun fichier ou dossier de ce type
      #include "zmq.h"

  but seems happy after ...

# Installing and configuring a jupyterhub with single user docker spawner

- Ugrade to Docker 1.6 and to a recent Docker Python client:

      curl -sSL https://get.docker.com/ | sh
      pip3 install --upgrade docker.py

  Rationale:

    - The Python Docker client shiped with Ubuntu 15.04 (package
      docker-python) is not recent enough for
      jupyterhub/dockerspawner; so it needs to be upgraded with:

        pip3 install --upgrade docker.py

    - This recent docker client requires Docker 1.6 while
      Ubuntu 15.04 ships Docker 1.5.

  To test the compatibility between Docker and the Docker Python client:

      sudo python3
      from docker import Client
      c = Client(base_url='unix://var/run/docker.sock')
      c.containers()

- Follow the instructions on https://github.com/jupyter/dockerspawner:

      git clone git@github.com:jupyter/dockerspawner.git	
  
      cd dockerspawner
      sudo pip3 install -r requirements.txt
      sudo python3 setup.py install

      --> does not work for me (td) on Ubuntu 14.04 LTS:
      cd dockerspawner
      python3 setup.py install
      <---- seems ok.

- Get the docker image (virtual size: 2.9 Go!)

      docker pull jupyter/singleuser

- Create a JupyterHub configuration file:

      jupyterhub --generate-config

- Edit jupyterhub_config.py and add:

      c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'

- Current status:

      Once authenticated on jupyterhub, and redirected to the jupyter
      web server spawned within the singleuser docker container, we
      get a "505 internal server error".

# Adding the Sage kernel into a system-wide jupyter

This is a quick hack to achieve it if you have run `sage -notebook=ipython`:

    cp -r ~/.sage/ipython-3.1.0/kernels/* /usr/local/share/jupyter/kernels/

    ## This does not work in a multiuser environment: j-hub seems to see only
    ## ~/.sage/ipython-3.1.0/kernels/

    Actually, kernels/* are:

    doc: a symbolic link to  /usr/local/sage/src/doc/output/html/en
    
    logo-64x64.png:  a symbolic link to /usr/local/sage/local/share/sage/ext/notebook-ipython/logo-64x64.png

    logo.svg a symbolic link to /usr/local/sage/local/share/sage/ext/notebook-ipython/logo.svg
    
    Change "/usr/local/sage" to the correct path.

    kernel.json:  an example:

    {"display_name": "Sage 6.8.beta2", "argv": ["/usr/local/sage/sage",
    "-python", "-m", "sage.repl.ipython_kernel", "-f",
    "{connection_file}"]}moi@kepler:/usr/local

    Documentation for "kernels" in Ipython3 is at:
    https://ipython.org/ipython-doc/dev/development/kernels.html
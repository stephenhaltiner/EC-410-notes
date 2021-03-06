Docker
================

  - [Why do we care?](#why-do-we-care)
  - [How it works](#how-it-works)
  - [Base R container](#base-r-container)
  - [RStudio+ container](#rstudio-container)
  - [Sharing files with a container](#sharing-files-with-a-container)

#### Resources

  - [Rocker wiki](https://github.com/rocker-org/rocker/wiki)
  - [ROpenSci Docker
    Tutorial](http://ropenscilabs.github.io/r-docker-tutorial)

## Why do we care?

1.  **Reproducibility**: Code is guaranteed to work on any system
2.  **Deployment**: Saves time and installation headaches when running
    code in the cloud/cluster

Docker containers are like ISO shipping containers. They accommodate all
manner of goods (software) and can go on ship, road, or rail (operating
systems).

## How it works

1.  Start with a stripped-down OS, usually a Linux distro like Ubuntu.
2.  Install *all* the programs and dependencies needed to run your code.
3.  (Add any extra configurations you want.)
4.  Package everything up as a *tarball* (a format for storing many
    files as a single object).

## Base R container

Start a simple container with little more than a base R installation:

`$ docker run --rm -ti rocker/r-base`

`docker run` flags:

  - `--rm` Automatically remove the container once it exits (i.e. clean
    up)
  - `-ti` Launch with interactive (`i`) shell/terminal (`t`)
  - Type `man docker run` for a full list of flag options

See a list of running containers on your system:

`$ docker ps`

If you don’t want to launch directly into your container’s R console,
you can start it in the bash shell instead:

`$ docker run --rm -ti rocker/r-base /bin/bash`

This time to close and exit the container, you need to exit the shell,
e.g.:

`root@09dda673a187:/# exit`

## RStudio+ container

Let’s get the R + RStudio + tidyverse image:

`$ docker run -d -p 8787:8787 -e PASSWORD=mypassword rocker/tidyverse`

  - All RStudio+ images in the Rocker stack require a password. It can
    be anything except “rstudio” which is the default username.
  - You can change the username by adding `-e USER=myusername` to the
    above command.

This time we aren’t immediately taken to our R environment. This is
because our container is running RStudio Server, which is accessed
through a browser.

  - **Mac/Windows:** Get your IP with `$docker inspect <containerid> |
    grep IPAddress`. Then navigate to your IP plus the assigned `8787`
    port.
  - **Linux:** use `localhost:8787` (works on Mac too?)

We ran this as a background process in the terminal by using the `-d`
flag. Without the flag, the terminal stays busy and you can shut down
the container by typing `CTRL+C` in that terminal window.

## Sharing files with a container

Each container runs in a sandboxed environment and can’t access other
directories on your computer unless you give it explicit permission.

To share files with a container, use `-v` (mount volume).

  - Adopts a `LHS:RHS` convention, where `LHS` = `path/on/your/PC` and
    `RHS` = `path/on/the/container`.

Example: I have a folder on my PC, `/home/stephen/myproject`. Make it
available to this “tidyverse” container by running:

  - `docker run -v /home/stephen/myproject:/home/rstudio/myproject -d
    -p 8787:8787 -e PASSWORD=mypassword rocker/tidyverse`

`/home/rstudio` is the default user’s home directory for images in the
“RStudio+” stack. When running a container from this stack, you should
almost always start your RHS with this path root. (Exception: you
assigned a different default user than “rstudio”.)

We must mount under the user’s home directory because RStudio Server
limits how and where users can access files.

Choosing a specific RHS mount point is less important for non-“RStudio+”
containers, but the `/home/rstudio` path won’t work for the r-base
container we ran earlier. That’s because there’s no “rstudio” user; when
you run an r-base container you are logged in as root.

For non-“RStudio+” containers, recommend a general strategy of mounting
external volumes to within the dedicated `/mnt` directory that is
standard on Linux.

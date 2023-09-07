# How to install Ubuntu on WSL with Docker and a graphical environment

**Publication date:** September 07, 2023

**Author:** Luís Vilar Barbosa


## Introduction

This tutorial was designed to explain how to install the Ubuntu distribution on the Windows Subsystem for Linux (both v1 and v2), explain how to install Docker, and explain how to set up an integration that allows Linux applications that need a graphical environment to appear on the Windows graphical environment.

All the presented steps were tested on Windows 10, version 22H2, and Windows 11, also version 22H2.


## Install Ubuntu on WSL v2 (easy version)

The easiest way of installing Ubuntu on WSL v2 is the following:

1. Open `Windows PowerShell` and type:
    ```
    wsl --install
    ```
2. Reboot the machine.
3. The app `Ubuntu` will start automatically after login.
4. The app will start immediately installing Ubuntu and, then, it will ask to create a user account.

    At this stage, it is possible to use a good amount of Linux features with integration with the Windows host.

    As an example, you can open the folder `\\wsl$\Ubuntu` on `File Explorer` to see the files that you have inside the Ubuntu virtual machine and you can perform a change of directory inside the terminal (using `cd /mnt/c/`) to see the files that you have in your Windows host.

    If everything went successfully, you have a minimal virtual machine running Ubuntu on your Windows host with integration with the Windows host. At this point, you have a fully capable Linux machine that is minimal, but it also has systemd enabled on `/etc/wsl.conf`, so we can have services enabled on boot which will be important to start the Docker daemon on boot.

5. Close the app `Ubuntu`.
6. Open `Windows PowerShell` and type:
    ```
    wsl --shutdown
    ```
7. Jump to point 3 of the section `Change Ubuntu to WSL v2 (detailed version)`.


## Install Ubuntu on WSL (detailed version)

1. Enable the features `Windows Subsystem for Linux` and `Virtual Machine Platform` (`Virtual Machine Platform` is only necessary for WSL v2) on `Windows Features`. The features `Hyper-V` and `Windows Hypervisor Platform` are not necessary.
2. Reboot the machine.
3. If you are using Windows 11 as host, go to `Windows PowerShell` and type:
    ```
    wsl --set-default-version 1
    ```

    On Windows 10, WSL version 1 is the default, but, on Windows 11, WSL version 2 is the default and you need to update WSL (point 1 of the section `Change Ubuntu to WSL v2 (detailed version)`) before going to point 5 of this section. This step allows to harmonize the behavior between Windows 10 and 11.

4. Install the app `Ubuntu` through the Microsoft Store.
5. Open the app `Ubuntu`. The app will start immediately installing Ubuntu and, then, it will ask to create a user account.

At this stage, it is possible to use a good amount of Linux features with integration with the Windows host.

As an example, you can open the folder `\\wsl$\Ubuntu` on `File Explorer` to see the files that you have inside the Ubuntu virtual machine and you can perform a change of directory inside the terminal (using `cd /mnt/c/`) to see the files that you have in your Windows host.


## Change Ubuntu to WSL v2 (detailed version)

This will change the Ubuntu distribution from WSL v1 to WSL v2 and this change is important to be able to use Docker because Docker needs the Linux Kernel with full system call compatibility.

More information here: https://docs.microsoft.com/en-us/windows/wsl/compare-versions

1. Open `Windows PowerShell` and type:
    ```
    wsl --update
    ```
    This step is important to ensure that you have a version of WSL that already contains the Linux kernel (so that you don't need to download and install it manually) and that supports systemd.

2. Go to `Windows PowerShell` and type:
    ```
    wsl --set-version "Ubuntu" 2
    ```

    Note: While performing the operation triggered on this step, the Ubuntu machine will be shut down without closing the window of the app `Ubuntu`.

    If everything went successfully, you have a minimal virtual machine running Ubuntu on your Windows host with integration with the Windows host. At this point, you have a fully capable Linux machine that is minimal, but it also has systemd enabled on `/etc/wsl.conf`, so we can have services enabled on boot which will be important to start the Docker daemon on boot.

3. Before starting again the app `Ubuntu`, create a file called `.wslconfig` on your Windows user folder (`%USERPROFILE%` or `~`) with the following content (adjust the values based on your machine characteristics and application necessities):
    ```
    [wsl2]
    memory=8GB
    processors=8
    localhostForwarding=true
    ```

    Here, I limited the memory to 8GB because, usually, it is not necessary to use more and WSL takes some time to release the memory in use by Docker that a good amount of times reaches the limits of the machine. I also set the number of processors to the number available in the host machine because usually the performance is not reduced even if both Windows and Ubuntu are performing intensive tasks and enabled localhost forwarding to be able to use localhost in the Windows host to access a program running inside WSL.

**Since WSL v2 uses a virtual machine, if your files are in the Windows host, they are being accessed through a network share which is slower than having the file inside the virtual machine, so, for high performance, you should try to have all or most of the files you need inside the virtual machine.**


## Install Docker on Ubuntu

1. Open again the app `Ubuntu` which is ready to use.
2. Install Docker by typing `curl -sSL https://get.docker.com | bash`. It will ask for your password 1 time.
3. Allow the current user to execute Docker without using `sudo` by typing the following two commands:
    ```
    sudo usermod -aG docker $USER
    newgrp docker
    ```

    Please, be aware that the `docker` group grants privileges equivalent to the `root` user.

    For more information, go to https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user.

4. If you need to access insecure Docker registries, type `sudo nano /etc/default/docker`, add the following entry (correctly populated), type CTRL+X and type `y` to save the changes to the file:
    ```
    DOCKER_OPTS="--insecure-registry=registry1.example.com --insecure-registry=registry2.example.com"
    ```
5. If you want to install Docker Compose, type:
    ```
    sudo apt install -y docker-compose
    ```

    (If you want the most up-to-date version of docker-compose and don't need automatic updates, you can install pip3 and type `pip3 install docker-compose`).


## Start using Docker

1. Start a new `Ubuntu` window (or use the one already opened in the previous section).
2. Type `docker run hello-world`.

And voilà, now you have a Linux machine with complete integration with the Windows host and with a working Docker installation.


## Graphical environment (bonus)

On a previous version of Windows 10, the integration of a graphical environment would require an X server like VcXsrv installed on the Windows machine to receive the data from the WSL.

Now, similarly to Windows 11, the integration of the graphical environment is active by default, so it is not necessary to follow the steps that were presented on the [version 1 of this article](https://github.com/LuisVilarBarbosa/Articles/blob/main/2022_03_17_Ubuntu_on_WSL_with_Docker_and_graphical_environment_v1.md), but you can always use the steps presented on the version 1 if you want to point the graphical environment of another Linux machine to your Windows machine.

For the reason mentioned above, at this point in the tutorial, you have the ability to launch graphical applications on Ubuntu. If you want to test it, you can install any graphical application, for example, Firefox, and launch it through the terminal.

## Remark

This version 2 of the tutorial was partially inspired on the pull request ["[Update] Add Another option to start docker."](https://github.com/LuisVilarBarbosa/Articles/pull/2) by [Ivaldino Soares](https://github.com/soaresiv).

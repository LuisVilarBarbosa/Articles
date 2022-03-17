# How to install Ubuntu on WSL with Docker and a graphical environment

**Publication date:** March 17, 2022

**Author:** Luís Vilar Barbosa


## Introduction

This tutorial was designed to explain how to install the Ubuntu distribution on the Windows Subsystem for Linux (both v1 and v2), explain how to install Docker, and explain how to set up an integration that allows Linux applications that need a graphical environment to appear on the Windows graphical environment.

All the presented steps were tested on Windows 10, version 21H2.


## Install Ubuntu on WSL

1. Enable the features `Windows Subsystem for Linux` and `Virtual Machine Platform` (`Virtual Machine Platform` is only necessary for WSL 2) on `Windows Features`. The features `Hyper-V` and `Windows Hypervisor Platform` are not necessary.
2. Reboot the machine.
3. Install the app `Ubuntu on Windows` through the Microsoft Store.
4. Open the app `Ubuntu on Windows`. The app will start immediately installing Ubuntu and, then, it will ask to create a user account.

At this stage, it is possible to use a good amount of Linux features with integration with the Windows host.

As an example, you can open the folder `\\wsl$\Ubuntu` on `File Explorer` to see the files that you have inside the Ubuntu virtual machine and you can perform a change of directory inside the terminal (using `cd /mnt/c/`) to see the files that you have in your Windows host.


## Change Ubuntu to WSL v2

This will change the Ubuntu distribution from WSL v1 to WSL v2 and it is important to be able to use Docker because Docker needs the Linux Kernel with full system call compatibility.

More information here: https://docs.microsoft.com/en-us/windows/wsl/compare-versions

1. Open `Windows PowerShell` and type:
    ```
    wsl --set-version "Ubuntu" 2
    ```

    Note: While performing the operation triggered on this step, the Ubuntu machine will be shut down without closing the window of the app `Ubuntu on Windows`.

2. Download and install the kernel package present in the presented URL.
3. Go to `Windows PowerShell` and type again:
    ```
    wsl --set-version "Ubuntu" 2
    ```

    If everything went successfully, you have a minimal virtual machine running Ubuntu on your Windows host with integration with the Windows host. At this point, you have a fully capable Linux machine that is minimal and, for that reason, it doesn't come out of the box with some things like services, but we don't need those for this tutorial.

4. Before starting again the app `Ubuntu on Windows`, create a file called `.wslconfig` on your Windows user folder (`%USERPROFILE%` or `~`) with the following content (adjust the values based on your machine characteristics and application necessities):
    ```
    [wsl2]
    memory=8GB
    processors=8
    localhostForwarding=true
    ```

    Here, I limited the memory to 8GB because, usually, it is not necessary to use more and WSL takes some time to release the memory in use by Docker that a good amount of times reaches the limits of the machine. I also set the number of processors to the number available in the host machine because usually the performance is not reduced even if both Windows and Ubuntu are performing intensive tasks and enabled localhost forwarding to be able to use localhost in the Windows host to access a program running inside WSL.

**Since WSL v2 uses a virtual machine, if your files are in the Windows host, they are being accessed through a network share which is slower than having the file inside the virtual machine, so, for high performance, you should try to have all or most of the files you need inside the virtual machine.**


## Install Docker on Ubuntu

1. Open again the app `Ubuntu on Windows` which is ready to use.
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
5. Configure the Docker daemon to start when `bash` is started, something that happens when you open the app `Ubuntu on Windows`.
    1. Type `sudo visudo`.
    2. Add the following at the end of the file to indicate that the user `unix` does not need to indicate the password to start /etc/init.d/docker, type CTRL+X and type `y` to save the changes to the file:
        ```
        # Docker daemon specification (added by Luis B.)
        unix ALL=(ALL) NOPASSWD: /etc/init.d/docker
        ```
    3. Type `nano ~/.bashrc`, add the following line at the end of the file, type CTRL+X and type `y` to save the changes to the file:
        ```
        sudo /etc/init.d/docker start
        ```
6. If you want to install Docker Compose, type:
    ```
    sudo apt install -y docker-compose
    ```

    (If you want the most up-to-date version of docker-compose and don't need automatic updates, you can install pip3 and type `pip3 install docker-compose`).


## Start using Docker

1. Start a new `Ubuntu on Windows` window.
2. Type `docker run hello-world`.

And voilà, now you have a Linux machine with complete integration with the Windows host and with a working Docker installation.


## Graphical environment (bonus)

**On Windows 11, supposedly, the integration of the graphical environment is active by default, but I did not test it since this tutorial was targeted for Windows 10.**

1. Install VcXsrv on Windows. The binary package is available at https://sourceforge.net/projects/vcxsrv/.
2. To start VcXsrv on user login, go to the folder `%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`, create a new shortcut for the location `"C:\Program Files\VcXsrv"` with the name `vcxsrv.exe`, go to the shortcut properties, and set the `Start in` as `"C:\Program Files\VcXsrv"` and the `Target` as:
    ```
    "C:\Program Files\VcXsrv\vcxsrv.exe" :0 -ac -terminate -lesspointer -multiwindow -clipboard -wgl -dpi auto
    ```

    After creating the shortcut, open it.

3. Do not allow `VcXsrv windows xserver` to create an exception in the firewall, just click `Cancel`.
4. Go to `Windows Defender Firewall with Advanced Security` and:
    1. Create an `Inbound Rule` for the program `C:\program files\vcxsrv\vcxsrv.exe`, with the action `Allow the connection` when in a `Public` network only and called `0-VcXsrv`.
    2. Open the rule and go to the tab `Protocols and Ports`, set the `Protocol type` as TCP, the `Local port` to the `Specific Ports` 6000, and the `Remote port` to `All Ports` and, then, go to the tab `Scope`, set `Local IP address` as `Any IP address` and set `Remote IP address` as `Local subnet`.
    3. Disable the rule with the name `VcXsrv windows xserver` that blocks the connection when in a `Public` network using `TCP` as protocol.
5. On the app `Ubuntu on Windows`, type `nano ~/.bashrc`, add the following line at the end of the file, type CTRL+X and type `y` to save the changes to the file:
    ```
    export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
    ```
6. Start a new `Ubuntu on Windows` window or type `bash` to load the new configuration.

Now, you have the ability to launch graphical applications on Ubuntu. If you want to test it, you can install any graphical application, for example, Firefox, and launch it through the terminal.

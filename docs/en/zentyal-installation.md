---

tags:
  - Zentyal

---

# Zentyal Installation

On this page, we will explain how to install Zentyal 7.0 on an Ubuntu Server 20.04 on an EC2 instance from AWS cloud provider. The objective is simply to install Zentyal 7.0 with only the essential modules and confirm that there are no network incidents after a reboot.

For the installation, we will use [this] script available from Zentyal. It is worth mentioning that we will install Zentyal without a graphical environment, as we want to reduce the resource usage needed by the server.

[this]: https://doc.zentyal.org/en/installation.html#instalacion-sobre-ubuntu-20-04-lts-server-o-desktop

The environment data for the project will be:

* **Server Name:** arthas
* **Domain:** icecrown.es
* **IP:** 10.0.1.200/24
* **Network Type:** Internal
* **Name of an additional administrative user:** djoven

## Requirements

* The operating system **must** be `Ubuntu 20.04 LTS`.
* The server must have a minimum of 4GB of RAM.
* An administrator user (group `sudo`) is required.

## Considerations

* The steps described below are identical in both cloud and on-premise environments.
* In case of not having robust knowledge about Linux, it is recommended to use the commercial version, as support access can be contracted, which can be very useful in case of incidents or version updates.
* If the server is installed in a cloud provider or a location without physical access to the server, the stability of the network module will be critical. Below are some recommendations regarding this:
    * Define the configuration that will be set in advance.
    * Assign a specific IP to the instance's network interface (in case a cloud provider is used).
    * Set the instance IP as static in Zentyal.
    * It is recommended to configure the network card in Zentyal as internal, thus avoiding self-blocking during the initial configuration.

## Previous configuration

Before proceeding to install Zentyal, we will perform the following actions:

1. We connect to the instance through SSH using the private key that we downloaded when creating the Key pair:

    ```bash
    ssh -i KP-Prod-Zentyal.pem ubuntu@arthas.icecrown.es
    ```

2. We set a password for the users; `root` and `ubuntu`:

    ```bash
    sudo passwd root
    sudo passwd ubuntu
    ```

3. We update the server packages:

    ```bash
    sudo apt update
    sudo apt dist-upgrade -y
    ```

4. We create an additional administrative user, which we will use to manage Zentyal from the administration interface:

    ```bash
    sudo useradd -m -d /home/djoven -G sudo -s /bin/bash -c 'Sysadmin' djoven
    sudo passwd djoven
    ```

    !!! warning

        It's important that the user belongs to the `sudo` group, otherwise we won't be able to use it to access the administration interface.

5. We log in with the newly created user:

    ```bash
    su - djoven
    ```

6. We create the necessary directory and file to host our public key to be able to connect via SSH:

    ```bash
    mkdir -v .ssh
    touch .ssh/authorized_keys
    ```

7. Finally, we add the contents of our public key to the newly created file `.ssh/authorized_keys`.

## Instalation

From now on, the server will be ready to install Zentyal 7.0. Here are the steps to follow for its installation:

1. Create a directory where we will store the Zentyal installation script:

    ```bash
    sudo mkdir /opt/zentyal-install
    ```

2. Download the script and give it the appropriate permissions:

    ```bash
    sudo wget -O /opt/zentyal-install/zentyal_installer.sh https://zentyal.com/zentyal_installer.sh
    sudo chmod 0750 /opt/zentyal-install/zentyal_installer.sh
    ```

3. Install Zentyal through the script:

    ```bash
    sudo bash /opt/zentyal-install/zentyal_installer.sh
    ```

    !!! note

        We will answer `n` to the question: '*Do you want to install the Zentyal Graphical environment?*', since we don't want to install the graphical environment.

    The Zentyal packages that the script will install for us are:

    * zentyal (meta-package)
    * zentyal-core
    * zentyal-software

    !!! warning

        At this point, we cannot restart the server until we have installed and configured the network module, otherwise the server will start without an IP address and we will lose access through SSH.

4. Once the script has finished, we log in to the Zentyal administration panel: https://arthas.icecrown.es:8443

    ![Initial wizard - Packages](assets/images/zentyal/01-wizard_packages.png "Initial wizard - Packages")

    !!! info

        If we haven't created the `A` record in the DNS, we will use the public IP address of the instance.

5. We log in with the administrator user that we previously created, in my case it's djoven.

6. In the initial configuration wizard, we only install the [firewall] module, this way the [network] module will be installed as a dependency.

7. We configure the network card as internal and static:

    ![Initial wizard - Network 1](assets/images/zentyal/02-wizard_network-1.png "Initial wizard - Network 1")
    ![Initial wizard - Network 2](assets/images/zentyal/03-wizard_network-2.png "Initial wizard - Network 2")

    !!! warning

        It's possible that when the network is finished configuring, the reported bug [here] will occur. If this is the case, follow the steps described on the `Bug fixing` page (see top navigation menu) or simply modify the URL to: <https://arthas.icecrown.es:8443>

8. Once changes have been saved, we can start managing Zentyal through the dashboard.

    ![Zentyal initial dashboard](assets/images/zentyal/04-dashboard_initial.png "Zentyal initial dashboard")

9. Finally, before proceeding to configure the server, we will perform the following checks to confirm the stability of the server:

    1. We ensure that all modules are enabled (Modules Status).
    2. The machine has access to the Internet:

        ```bash
        ping -c4 8.8.8.8
        ping -c4 google.es
        ```

    3. There have been no errors in the log `/var/log/zentyal/zentyal.log`. Here is an example of the logs without any errors:

        ```bash
        2022/10/23 08:17:51 DEBUG> PAM.pm:83 Authen::Simple::PAM::check - Successfully authenticated user 'djoven' using service 'zentyal'.
        2022/10/23 08:20:29 INFO> install-packages:61 main:: - Starting package installation process
        2022/10/23 08:20:39 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:39 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:40 INFO> Service.pm:965 EBox::Module::Service::restartService - Restarting service for module: network
        2022/10/23 08:20:43 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: network
        2022/10/23 08:20:43 INFO> Base.pm:256 EBox::Module::Base::saveConfig - Saving config for module: firewall
        2022/10/23 08:20:43 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:20:44 INFO> Service.pm:965 EBox::Module::Service::restartService - Restarting service for module: firewall
        2022/10/23 08:20:49 INFO> install-packages:121 main:: - Package installation process finished
        2022/10/23 08:23:15 INFO> Network.pm:89 EBox::Network::CGI::Wizard::Network::_processWizard - Configuring ens5 as 10.0.1.200/255.255.255.0
        2022/10/23 08:23:15 INFO> Network.pm:93 EBox::Network::CGI::Wizard::Network::_processWizard - Adding gateway 10.0.1.1 for iface ens5
        2022/10/23 08:23:15 INFO> Network.pm:108 EBox::Network::CGI::Wizard::Network::_processWizard - Adding nameserver 1.1.1.1
        2022/10/23 08:23:15 INFO> Network.pm:114 EBox::Network::CGI::Wizard::Network::_processWizard - Adding nameserver 9.9.9.9
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:571 EBox::GlobalImpl::saveAllModules - First installation, enabling modules: network firewall webadmin logs audit firewall
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module network
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module firewall
        2022/10/23 08:23:17 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module webadmin
        2022/10/23 08:23:18 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module logs
        2022/10/23 08:23:18 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module audit
        2022/10/23 08:23:19 INFO> GlobalImpl.pm:574 EBox::GlobalImpl::saveAllModules - Enabling module firewall
        2022/10/23 08:23:19 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: network
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: logs
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: audit
        2022/10/23 08:23:23 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: firewall
        2022/10/23 08:23:24 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: sysinfo
        2022/10/23 08:23:25 INFO> GlobalImpl.pm:660 EBox::GlobalImpl::saveAllModules - Saving configuration: webadmin
        2022/10/23 08:23:25 INFO> Base.pm:231 EBox::Module::Base::save - Restarting service for module: webadmin
        2022/10/23 08:24:51 INFO> Index.pm:187 EBox::Dashboard::CGI::Index::masonParameters - dashboard1
        ```

    4. We restart the server to ensure that it is able to start without any network problems.

        ```bash
        sudo reboot
        ```

    5. We verify that we can connect through SSH and to the Zentyal administration interface.

[firewall]: https://doc.zentyal.org/en/firewall.html
[network]: https://doc.zentyal.org/en/firststeps.html#configuracion-basica-de-red-en-zentyal
[here]: https://github.com/zentyal/zentyal/issues/2100

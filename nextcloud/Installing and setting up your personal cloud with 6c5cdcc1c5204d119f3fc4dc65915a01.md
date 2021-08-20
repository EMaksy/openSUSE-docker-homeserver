# Installing and setting up your personal cloud with openSUSE, Docker and Nextcloud

[Eugen Maksymenko](https://github.com/EMaksy) 

## Use Cases

- Regain controls and own your personal data
- Share images videos and data inside your home network
- Have access to your data on the home network even if the internet cloud is down?
- Share your data with other people via the Internet?
- Scale storage on Demand
- Make use out of Open Source Projects without additional costs
- Have a great learning experience

## Target audience

This "How to" guide is targeted to:

- All tech enthusiast out there.
- All advanced Linux Users/Admins and Developers,  so a short command summary will be provided, so you can skip to the juicy part [here](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/Quick%20Guide%20for%20advanced%20user%20bb5c3c3e47fb45a2bd43ecbe055cab08.md)

## Goal of this Article

Provide all the needed Information, to create your own personal Cloud and make it accessible inside your network.

- Use already owned hardware to host your personal server.
- Learn more about openSUSE Linux.
- Install and learn to use Docker container technologies to power your private server.
- Host your own private cloud storage and be in the power of your own data with Nextcloud.

## Conceptional Overview

In this article, we are going to use the following technologies: 

- [openSUSE Leap 15.3](https://get.opensuse.org/leap/#download)

    This is the base for your home server.

    **Why openSUSE?**

    openSUSE Leap uses source and newly also binaries from SUSE Linux Enterprise (SLE), which gives Leap a level of stability unmatched by other Linux distributions, and combines that with community developments to give users, developers and sysadmins the best stable Linux experience available

    openSUSE also offers 5 years of Long term support.

    On this system, you will run multiple Docker containers.

- [Docker](https://docs.docker.com/get-started/overview/)
Multiple Docker containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes
Docker Containers are isolated Environments and by default quite secure.
Containers take up less space than Virtual Machines and are faster to deploy.
Learn about the difference between container and virtual machine  [https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)
- [Maria DB](https://www.mariadbtutorial.com/getting-started/) 
MariaDB is a fast, scalable and robust database.
MariaDB will empower our Nextcloud and serve it as a reliable SQL database.
- [Nextcloud](https://nextcloud.com/)

    The self-hosted productivity platform that keeps you in control of your personal data.

    Nextcloud puts your data under your control.
    Store your documents, calendar, contacts and photos on a server at home.

**Concept summary:**

You will use openSUSE Linux as our Host system. 

You will start two docker containers with one docker-compose file.

Every container will include its own application. 

The first container includes the MariaDB database and the second container includes a Nextcloud application.

Nextcloud and MariaDB will communicate with each other and ensure optimal performance. 

Visualization of the concept:

![concept.png](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/concept.png)

## Hardware Requirements

There are a good deal of different Hardware system options, how to achieve the best performance.

So basically, as long as you meet up with the official openSUSE Leap 15.3  [recommendations](https://en.opensuse.org/Hardware_requirements_15.3)[,](https://doc.opensuse.org/release-notes/x86_64/openSUSE/Leap/15.3/) you should be fine.

**A hardware example could look like this:**

- CPU with Virtualization. Example:  AMD Athlon 200GE
- RAM: 8 GB RAM
- NVM/SSD 128 GB for openSUSE LEAP 15.3 as your host system
- Additional storage drive HDD/SSD/NVM for your Nextcloud data

## Before we start

Follow this guide on an already running system or make a fresh installation.

If you decide for a clean new server,  just follow the official documentation

[Installation Quick Start | Start-Up | openSUSE Leap 15.3](https://doc.opensuse.org/documentation/leap/startup/html/book-startup/art-opensuse-installquick.html)

### Update your system

Open a command line, update your system and install the updates:

Add reason why to update (latest security fixes and bug fixes)

```bash
sudo zypper refresh && sudo zypper update
```

If your system is up-to-date the output should look like this

```bash
All repositories have been refreshed.
Nothing to do.
```

### Prepare a drive/partition for your Nextcloud data.

Mount a drive permanently on your host system, so you can store your Nextcloud data on a separate place than your root user.

Source: [https://www.suse.com/c/manually-partitioning-your-hard-drive-fdisk/](https://www.suse.com/c/manually-partitioning-your-hard-drive-fdisk/)

# Install Docker and create your own Nextcloud container in 3 steps

## STEP 1: Install Docker on openSUSE Leap 15.3

1.  Install all required Docker packages:

    ```bash
    sudo zypper install --no-confirm docker python3-docker-compose
    ```

    Check the version of Docker:

    ```bash
    docker --version
    ```

2. Start and enable Docker:

    ```bash
    sudo systemctl enable --now docker 
    ```

3. To ensure that everything worked out, let's test it with these two commands.
    - Check if Docker is running:

        ```bash
         systemctl is-active docker

        ```

        Expected output

        ```bash
        active
        ```

    - Check if Docker is enabled, so it automatically starts up on system reboot:

        ```bash
        systemctl is-enabled docker
        ```

        Expected output

        ```python
        enabled
        ```

Docker is now installed on your openSUSE Leap 15.3 server. 

## STEP 2: Configure Nextcloud Docker container

### Step 2.1: Create and start a Nextcloud Docker container.

1. In a terminal, create a working directory for our upcoming Nextcloud Docker container:

    ```bash
    mkdir --parents ~/docker/nextcloud
    ```

2. Change directory:

    ```bash
    cd ~/docker/nextcloud
    ```

3. Create a configuration file, `docker-compose.yml` and insert the following lines:

    ```yaml
    version: "3"
      
    volumes:
      nextcloud:
        driver: local
      db:

    services:
      db:
        image: mariadb:latest
        restart: always
        command: --transaction-isolation=READ-COMMITTED --log-bin=ROW --innodb-read-only-compressed=OFF
        volumes:
          - db:/var/lib/mysql
        environment:
          - MYSQL_ROOT_PASSWORD=<INSERT-A-DATABASE-ROOT-PASSWORD>
          - MYSQL_PASSWORD=<INSERT-ANOTHER-PASSWORD-THEN-THE-DATABASE-ROOT-PASSWORD-FOR-BETTER-SECURITY>
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud

      app:
        image: nextcloud:latest
        restart: always
        ports:
          - 8080:80
        links:
          - db
        volumes:
          - nextcloud:/var/www/html
        environment:
          - MYSQL_PASSWORD=<INSERT-ANOTHER-PASSWORD-THEN-THE-DATABASE-ROOT-PASSWORD-FOR-BETTER-SECURITY>
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_HOST=db
    ```

**Attention: Change password values to ensure security of your database.**

- MYSQL_ROOT_PASSWORD=<INSERT-A-DATABASE-ROOT-PASSWORD>
- MYSQL_PASSWORD=<INSERT-ANOTHER-PASSWORD-THEN-THE-DATABASE-ROOT-PASSWORD-FOR-BETTER-SECURITY>

Find the latest iteration of the Nextcloud `docker-compose.yml` file at my GitHub repository

[](https://github.com/EMaksy/openSUSE-docker-homeserver/blob/master/docker-compose/nextcloud-docker/docker-compose.yml)

1. Start the Docker containers:

```bash
sudo docker-compose up --detach
```

1. Check if all containers are up and working:

```bash
sudo docker-compose ps
```

Expected output

```bash
Name                    Command               State                  Ports                
-----------------------------------------------------------------------------------------------
nextcloud_app_1   /entrypoint.sh apache2-for ...   Up      0.0.0.0:8080->80/tcp,:::8080->80/tcp
nextcloud_db_1    docker-entrypoint.sh --tra ...   Up      3306/tcp
```

### Step 2.2:  Mount a storage drive/partition on your Nextcloud Docker Container

Decide where all your Nextcloud data is stored by mounting a partition or even a whole storage drive to your container.

This step is required, if you don't want that your Nextcloud data is stored on your host root drive. 

Also, you can easily make back-ups of your storage place.

1. Shutdown your Nextcloud Docker container.

    ```bash
    sudo docker-compose down
    ```

2. Mount storage device/partition

    ```bash
    sudo mount /dev/<DEVICE-NAME> /var/lib/docker/volumes/nextcloud_nextcloud/_data
    ```

    **Attention: You need to add your Device.
    This is described in "Before we start"**

3. Start Container

    ```bash
    sudo docker-compose up --detach
    ```

## Step 3: Finish your Nextcloud installation

### 1. Access your Nextcloud

a)  from the host system:

```bash
http://localhost:8080/
```

b)  from home network:

```bash
http://<IP-HOST>:8080
```

Your Nextcloud should look like this:

![Untitled](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/Untitled.png)

### 2.  Decide a username for your admin account.

### 3. Pick a strong password for your admin user

If you are planning to make Nextcloud available from the Internet, make sure you pick up an extra strong password.

### 4. Decide if you want to install the recommended apps

If you plan to use Nextcloud only to store, share and upload data, on your network, then you don't require recommended apps. 

### 5. Press the Finish setup button

Installation of your Nextcloud instance has started.

After the installation, Nextcloud will welcome you like this.

![Untitled](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/Untitled%201.png)

### If the browser show you an error or is stuck, don't panic and try this

- On wrong redirections, refresh your web browser with F5
- Retry access your Nextcloud like in STEP 3.1
- Check logs of your Docker container
[https://docs.docker.com/engine/reference/commandline/logs/](https://docs.docker.com/engine/reference/commandline/logs/)

## Conclusion

### Congratulation!!!

- You created a working directory for your Nextcloud Docker container.
- You configured a Docker-compose file for your Nextcloud + MariaDB database container.
- You started a Nextcloud Docker Container.
- You added a storage device/partition to your Nextcloud Docker container for all your private data.
- You created an admin User and installed Nextcloud on your home server
- You can access your Nextcloud from every device in your homework

### Create multiple user

It is possible to create multiple user on your Nextcloud for your whole family and friends.

Source: [https://docs.nextcloud.com/server/latest/admin_manual/configuration_user/user_configuration.html](https://docs.nextcloud.com/server/latest/admin_manual/configuration_user/user_configuration.html)

### Allow user registration on your home network

It is possible to allow an independent user creation. 

This is possible by installing an additional app.

Source: [https://apps.nextcloud.com/apps/registration](https://apps.nextcloud.com/apps/registration)

### Show Nextcloud version

Have an overview about your Nextcloud system

```bash
http://<IP>:8080/settings/admin/overview
```

```bash
http://127.0.0.1:8080/settings/admin/overview
```

In my case, it looks like.

![Untitled](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/Untitled%202.png)

In order to update your Nextcloud instance, you need to shut down your container and restart it again.

The configuration in your docker-compose file will automatically update to the latest version of Nextcloud image

### Consider backup strategy

As a fresh owner of an own cloud, you are also responsible what is happening with your data.

Inform yourself about a backup strategy.

With great power over your data, comes great responsibility.

Source: 

[https://docs.nextcloud.com/server/latest/admin_manual/maintenance/backup.html](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/backup.html)

[https://nextcloud.com/blog/how-to-back-up-nextcloud-with-bareos/](https://nextcloud.com/blog/how-to-back-up-nextcloud-with-bareos/)

### Sync data on multiple clients

If you want to share the same data between multiple machines, then use the Nextcloud sync client.

[https://nextcloud.com/clients/](https://nextcloud.com/clients/)

A use case for this would be to sync up your Password manager database between multiple systems.

Password Manager [https://keepassxc.org/](https://keepassxc.org/)

## Known Issues

- Access through untrusted domain

    All URLs used to access your Nextcloud server must be whitelisted in your config.php file, under the trusted_domains setting, else you get an error message.

    1. Change into Nextcloud Docker directory:

        ```bash
        cd ~/docker/nextcloud
        ```

    2. Shutdown the container:

        ```bash
        sudo docker-compose down
        ```

    3. Edit `config.php` and add your trusted domain

        ```bash
        sudo vim /var/lib/docker/volumes/nextcloud_nextcloud/_data/config/config.php
        ```

        Follow  the official example [https://docs.nextcloud.com/server/22/admin_manual/installation/installation_wizard.html#trusted-domains](https://docs.nextcloud.com/server/22/admin_manual/installation/installation_wizard.html#trusted-domains)

    4. Start the container:

        ```bash
        sudo docker-compose up --detach
        ```

    5. Access Nextcloud on your now trusted domain

- Sync client Problems

    Some users have issues with the Nextcloud sync client. This is a proxy manager Issue and some additional configuration are required. 

    1. Change into Nextcloud Docker directory:

        ```bash
        cd ~/docker/nextcloud
        ```

    2. Shutdown the container:

        ```bash
        sudo docker-compose down
        ```

    3. Edit Nextcloud `config.php` config:

        ```bash
        sudo vim /var/lib/docker/volumes/nextcloud_nextcloud/_data/config/config.php
        ```

    4. Add additional configs:

        ```bash
        'overwriteprotocol' => 'https' 
        ```

        Additional official information:

        [https://docs.nextcloud.com/server/20/admin_manual/configuration_server/reverse_proxy_configuration.html?highlight=reverse#overwrite-parameters](https://docs.nextcloud.com/server/20/admin_manual/configuration_server/reverse_proxy_configuration.html?highlight=reverse#overwrite-parameters)

    5. Start the container:

        ```bash
        sudo docker-compose up --detach
        ```

    6. Sync client should work now.

## Additional sources

- Docker

    [Docker](https://en.opensuse.org/Docker)

    [Orientation and setup](https://docs.docker.com/get-started/)

- MariaDB

    [Introduction to Relational Databases](https://mariadb.com/kb/en/introduction-to-relational-databases/)

- Nextcloud

    [Nextcloud Documentation](https://docs.nextcloud.com/)

    [Nextcloud - Official Image | Docker Hub](https://hub.docker.com/_/nextcloud)

[Quick Guide for advanced user](Installing%20and%20setting%20up%20your%20personal%20cloud%20with%206c5cdcc1c5204d119f3fc4dc65915a01/Quick%20Guide%20for%20advanced%20user%20bb5c3c3e47fb45a2bd43ecbe055cab08.md)
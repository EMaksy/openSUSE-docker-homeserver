# Stage 1:
Install docker on openSUSE

Created: August 20, 2021 2:08 PM

- [x]  To-do
- [ ]  Completed To-do

# Target: Install docker on openSUSE Leap 15.3

1. Update your system and install updates

    ```bash
    sudo zypper refresh && sudo zypper update
    ```

2. Install docker and docker compose.

    ```bash
    sudo zypper install docker python3-docker-compose
    ```

3. Start and enable the docker service

    ```bash
    sudo systemctl enable --now docker 
    ```

## Congratulation, docker is now installed on your host.

You made sure that docker service will automatically start on host reboot
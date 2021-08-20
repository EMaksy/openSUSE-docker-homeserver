# Stage 2:
Run Nextcloud Docker container on the home network

Created: August 20, 2021 2:08 PM

- [x]  To-do
- [ ]  Completed To-do

1. Create a working directory.

    ```bash
    mkdir --parents ~/docker/nextcloud
    ```

2. Change current directory.

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

    Don't forget to adopt the database passwords!

4. Start Container for volume creation 

    ```bash
    sudo docker-compose up --detach
    ```

5. Shutdown Container

    ```bash
    sudo docker-compose down
    ```

6. Mount Drive for data  to this path

    ```bash
    sudo mount /dev/<DEVICE-NAME> /var/lib/docker/volumes/nextcloud_nextcloud/_data
    ```

7. Start Container

    ```bash
    sudo docker-compose up --detach
    ```

8. Access your Nextcloud

    Access WEB UI on host

    ```bash
    localhost:8080
    ```

Access WEB UI on home network:

```bash
<HOST-IP>:8080
```

Create Admin User

![Untitled](Stage%202%20Run%20Nextcloud%20Docker%20container%20on%20the%20home%206ed4e73950fc48128b162eca6b9ee546/Untitled.png)

Nextcloud is installing now, and after a short while you are welcomed by Nextcloud

![Untitled](Stage%202%20Run%20Nextcloud%20Docker%20container%20on%20the%20home%206ed4e73950fc48128b162eca6b9ee546/Untitled%201.png)

### Conclusion

You now are running a Nextcloud container on openSUSE Leap 15.3.

Your container is accessible from the home network.

Enjoy using your personal Nextcloud
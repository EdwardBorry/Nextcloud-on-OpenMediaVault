there're a lot of ways to install nextcloud and one of them is not a very solid or not proper one according to participants, however it works and works pretty fine, in the future i'll change an approach to nextcloud installation by installing it directly on PC (in my case it's RPi4) using docker that's supposed to be as a right and proper approach by people.

********


### Preparation before installation
* Create extra directories for data base and for storing files (nextcloud_app : nextcloud_db respectively (that was just an instance)).
It looks like like that:
![[1.png]]

* Install docker (you can do that directly in OMV), preview of your directories scheme looks like that:
  -- OMV
     -- Docker
         -- Nextcloud

* Go to /Services/Compose/Settings and set the same as i have at screen below:

![[Снимок экрана 2026-05-01 в 16.16.43.png]]![[3]]

 after go to Services/Compose/Files and create there a new one, after set this code there up:
```yaml
version: '3'

services:
  db:
    image: mariadb:10.6
    container_name: nextcloud_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloudpass
    volumes:
      - /srv/dev-disk-by-uuid-e37a1406-b2fc-4857-9f52-b7549902f7fa/nextcloud_d/

  app:
    image: nextcloud
    container_name: nextcloud_app
    restart: always
    ports:
      - 8080:80
    depends_on:
      - db
    volumes:
      - /srv/dev-disk-by-uuid-e37a1406-b2fc-4857-9f52-b7549902f7fa/nextcloudone/
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloudpass
```
(CHANGE VOLUMES ON THE RIGHT DATA)

Now you can deploy it and open in browser as:
```css
http://192.168.1.X:8080
```

*********

If you will try to connect it via vpn or some kind of that you'll get an error like:
```bash
access through untrusted domain. edit the "trusted_domains" setting in config/config.php like the example in config.sample.php
```

In order to fix that type in terminal next:

First, find your Nextcloud container name:
```bash
docker ps
```

Then run these commands (replace nextcloud with your container name):
```bash
docker exec --user www-data nextcloud php occ config:system:get trusted_domains
```

This will show you the current list. Then add your addresses one by one, for example:
```bash
docker exec --user www-data nextcloud php occ config:system:set trusted_domains 1 --value="192.168.1.XX"     # your Pi's IP
docker exec --user www-data nextcloud php occ config:system:set trusted_domains 2 --value="your.duckdns.org"
docker exec --user www-data nextcloud php occ config:system:set trusted_domains 3 --value="nextcloud.local"
```

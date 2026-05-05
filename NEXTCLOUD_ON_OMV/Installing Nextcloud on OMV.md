### Preparation before installation
* Create extra directories directly on OMV for data base and for storing files (nextcloud_db : nextcloud_app respectively (the names were given as an example).
* Create directories for compose files and data compose.

* Install docker (directly in OMV) therefore, as a result the preview of scheme directories should look like that:
  -- OMV
     -- Docker
         -- Nextcloud

* Inside /Services/Compose/Settings choose the directory for compose files in Compose Files section + choose the directory for data compose in Data section.
<img width="1366" height="633" alt="image" src="https://github.com/user-attachments/assets/1e651fc3-b2ad-4559-a108-4646af527021" />

### Nextcloud installation

Move to Services/Compose/Files and create there a new file, after set this code there up:

```yaml
version: '3.8'

services:
  db:
    image: mariadb:10.11
    restart: unless-stopped
    volumes:
      - /srv/dev-disk-by-uuid-ba0d7105-215a-411a-9df2-39983cbf2521/nextcloud_db/ # replace the name of the disk for database to yours
    environment:
      MYSQL_ROOT_PASSWORD: Password
      MYSQL_PASSWORD: Password
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud

  app:
    image: nextcloud:apache
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - /srv/dev-disk-by-uuid-ba0d7105-215a-411a-9df2-39983cbf2521/nextcloud_appdata/ # replace the name of the disk for data storage to yours
    environment:
      MYSQL_HOST: db
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: Password
      MYSQL_DATABASE: nextcloud
    depends_on:
      - db
```

Deploy it and open in browser as:
```css
http://192.168.1.X:8080 # (Nextcloud uses 8080 as a port by default)
```

*********

### Trusted_domain problem fix

If you will try to connect to Nextcloud via untrusted domain/IP, you'll get an issue:
<img width="921" height="546" alt="image" src="https://github.com/user-attachments/assets/d0330e5a-907a-433b-bb72-37a65d2b9c69" />

In order to fix this - type in terminal next:

* Find your Nextcloud container name:
```bash
docker ps
```

* Run the command (replace nextcloud with your container name!):
```bash
docker exec --user www-data nextcloud php occ config:system:get trusted_domains
```

* Add the address of untrusted device (domain):
```bash
docker exec --user www-data nextcloud php occ config:system:set trusted_domains 1 --value="192.168.1.XX"     # replace IP address (192.168.1.XX) to yours
```

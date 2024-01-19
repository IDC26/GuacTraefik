# GuacTraefik
## Setting up Traefik for Apache Guacamole
For setting up all the prerequisites, we firstly need to create a new directory for our docker-compose.yaml file
```linux
$ mkdir GuacTraefik
```
After creating the file, we access it using cd
```linux
$ cd GuacTraefik/
```
We create a file named docker-compose.yaml
```linux
$ touch docker-compose.yaml
```
Inside the GuacTraefik folder we cratea another folder called "init"
```linux
$ mkdir init
```
We are going to generate the initialization file for postgresql inside the folder as follow:
```linux
$ cd init
```
And now we are going to run a one time container which will generate a postgresql file inside the folder:
```linux
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > initdb.sql
```
Now we are going back to the main
```linux
$ cd ..
```
Inside the main folder, we are going to open up the yaml we created earlier.
```linux
$ nano docker-compose.yaml
```
Inside you can either copy my file or paste the following, keep in mind you have to change the password next to POSTGRES_PASSWORD:
```yaml
version: '2.0'

# networks
# create a network 'guacnetwork_compose' in mode 'bridged'
networks:
  guacnetwork_compose:
    driver: bridge

# services
services:
  traefik:
    image: traefik:v2.5
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
  # guacd
  guacd:
    container_name: guacd_compose
    image: guacamole/guacd
    networks:
      guacnetwork_compose:
    restart: always
    volumes:
    - ./drive:/drive:rw
    - ./record:/record:rw
  # postgres
  postgres:
    container_name: postgres_guacamole_compose
    environment:
      PGDATA: /var/lib/postgresql/data/guacamole
      POSTGRES_DB: guacamole_db
      POSTGRES_PASSWORD: 'YOUR_OWN_PASSWORD_HERE'
      POSTGRES_USER: guacamole_user
    image: postgres:13.4-buster
    networks:
      guacnetwork_compose:
    restart: always
    volumes:
    - ./init:/docker-entrypoint-initdb.d:z
    - ./data:/var/lib/postgresql/data:Z

  # guacamole
  guacamole:
    container_name: guacamole_compose
    depends_on:
    - guacd
    - postgres
    environment:
      GUACD_HOSTNAME: guacd
      POSTGRES_DATABASE: guacamole_db
      POSTGRES_HOSTNAME: postgres
      POSTGRES_PASSWORD: 'YOUR_OWN_PASSWORD_HERE'
      POSTGRES_USER: guacamole_user
    image: guacamole/guacamole
    links:
    - guacd
    networks:
      guacnetwork_compose:
    ports:
    - 8082:8080/tcp
    restart: always
    labels:
    - "traefik.enable=true"
    - "traefik.backend=guacamole"
    - "traefik.port=8080"
```
After creating the docker-copmose.yaml file, the only this remaining is to bring up docker as follows:
```linux
$ docker compose up
```
Inside the logs we can notice the tables creating inside the database
Once everythin is completed, we can access the web interace of guacamole
```url
localhost:8082/guacamole
```
And the Web interface of Traefik on
```url
localhost:8080
```

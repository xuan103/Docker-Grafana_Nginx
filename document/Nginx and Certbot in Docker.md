|[Grafana and InfluxDB in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Grafana%20and%20InfluxDB%20in%20Docker.md)|[Table of Contents](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/README.md)|[Setup Grafana](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Setup%20Grafana.md)| 
---| ---| ---|

<h1 id="start">Nginx and Certbot in Docker</h1>

![downloads](https://img.shields.io/github/downloads/atom/atom/total.svg)
![build](https://img.shields.io/appveyor/ci/:user/:repo.svg)
![chat](https://img.shields.io/discord/:serverId.svg)

---

<h2 id="Contents">Table of Contents</h2>

- [Nginx and Certbot in Docker](#start)
    - [Table of Contents](#Contents)
    - [Guide](#Guide)
    - [Architecture](#Architecture)
    - [Nginx as a server](#Nginx)
    - [Setting .conf](#conf)
    - [Create the certificate using Certbot](#Certbot)
    - [Renewing the certificates](#certificates)
    - [Related Posts](#Related)
    - [Appendix and FAQ](#Appendix)

---

<h2 id="Guide">Guide</h2>

1. Create a Docker Compose for Nginx
2. Create the certificate using Certbot
3. Renewing the certificates

---

<h2 id="Architecture">Architecture</h2>

```
.
├── docker-compose.yml
└── volume
    ├── certbot
    │   ├── conf
    │   └── www
    └── nginx
        ├── conf
        └── html
```

---

<h2 id="Nginx">Nginx as a server</h2>
---

```yaml=
version: '3'

services:
  webserver:
    image: nginx:latest
    restart: always
    ports:
      - 80:80
      - 443:443
```
> Run `sudo docker compose up` to start the Nginx, try the server is ok.


```yaml=
version: '3'

services:
  webserver:
    image: nginx:latest
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./volume/nginx/conf/:/etc/nginx/conf.d/:ro
      - ./volume/certbot/www/:/var/www/certbot/:ro
```

> Use the `volumes` feature of Docker. Map the folder located at `/etc/nginx/conf.d/` from the docker container to a folder located at `./volume/nginx/conf/` on local machine. Every file add, remove or update into this folder locally will be updated into the container.

> If you wish to adapt the default configuration, use something like the following to copy it from a running nginx container:

```
$ sudo docker run --name tmp-nginx-container -d nginx 
$ sudo docker cp tmp-nginx-container:/etc/nginx/conf.d/ volume/nginx/conf/
$ sudo docker cp tmp-nginx-container:/usr/share/nginx/html volume/nginx/html
$ sudo docker rm -f tmp-nginx-container
```

---

<h2 id="conf">Setting .conf</h2>

And add the following configuration file into your ./nginx/conf/ local folder. Do not forget to update using your own data.

```
$ nano volume/nginx/conf/default.conf
```

```conf
server {
    listen 80;
    listen [::]:80;

    server_name example.org www.example.org;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://example.org$request_uri;
    }
}
```

> Now reload nginx by doing a rough `sudo docker compose restart` or if want to avoid service interruptions (even for a couple of seconds) reload it inside the container using `sudo docker compose exec webserver nginx -s reload`.

---

<h2 id="Certbot">Create the certificate using Certbot</h2>

Use the docker image for certbot and add it as a service to Docker Compose project.

```yaml=
version: '3'

services:
  webserver:
    image: nginx:latest
    ports:
      - 80:80
      - 443:443
    restart: always
    volumes:
      - ./volume/nginx/conf/:/etc/nginx/conf.d/:ro
      - ./volume/certbot/www:/var/www/certbot/:ro
      - ./volume/certbot/conf/:/etc/nginx/ssl/:ro
  certbot:
    image: certbot/certbot
    volumes:
      - ./volume/certbot/www/:/var/www/certbot/:rw
      - ./volume/certbot/conf/:/etc/letsencrypt/:rw
```

> Might have noticed they have declared the same volume. It is meant to make them communicate together.

> Certbot will write its files into `./certbot/www/` and nginx will serve them on port 80 to every user asking for `/.well-know/acme-challenge/.` That's how Certbot can authenticate our server.

> Now test that everything is working by running `sudo docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d example.org`. Should get a success message like "The dry run was successful".

```yaml=
version: '3'

services:
  webserver:
    image: nginx:latest
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./volume/nginx/conf/:/etc/nginx/conf.d/:ro
      - ./volume/certbot/www/:/var/www/certbot/:ro
  certbot:
    image: certbot/certbot
    volumes:
      - ./volume/certbot/www/:/var/www/certbot/:rw
      - ./volume/certbot/conf/:/etc/letsencrypt/:rw
```

> Restart container using `sudo docker compose restart`. Nginx should now have access to the folder where Certbot creates certificates.

> However, this folder is empty right now. Re-run Certbot without the `--dry-run` flag to fill the folder with certificates:

```shell=
$ sudo docker compose run --rm  certbot certonly --webroot --webroot-path /var/www/certbot/ -d example.org
```

>---

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter'c' to cancel): "example@mail.com"

------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
------------------------------------
(Y)es/(N)o: "y"

------------------------------------
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
------------------------------------
(Y)es/(N)o: "n"
Account registered.
Requesting a certificate for example.org

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/example.org/fullchain.pem
Key is saved at: /etc/letsencrypt/live/example.org/privkey.pem
This certificate expires on 2022-02-19.
These files will be updated when the certificate renews.

NEXT STEPS:
-The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.

------------------------------------
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt: https://letsencrypt.org/donate
 * Donating to EFF: https://eff.org/donate-le
------------------------------------
```

> Since have those certificates, the piece left is the 443 configuration on nginx.

```conf
server {
    listen 80;
    listen [::]:80;

    server_name example.org;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

#    location / {
#        return 301 https://example.org$request_uri;
#    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name example.org;

    ssl_certificate /etc/nginx/ssl/live/example.org/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/example.org/privkey.pem;

    location / {
        proxy_pass http://example.org:3000/;
    }
}
```

> Reloading the nginx server now will make it able to handle secure connection using HTTPS. Nginx is using the certificates and private keys from the Certbot volumes.


---

<h2 id="certificates">Renewing the certificates</h2>

Since have this Docker environment in place, it is easier than ever to renew the Let's Encrypt certificates!

```shell=
$ sudo docker compose run --rm certbot renew
```

This small "renew" command is enough to let your system work as expected. Just have to run it once every three months. 

---

<h2 id="Related">Related Posts</h2>

-  Reference
    -   https://mindsers.blog/post/https-using-nginx-certbot-docker/

<h2 id="Appendix">Appendix and FAQ</h2>

:::info
**Find this document incomplete?** Leave a comment!
:::

###### tags: `Docker` `Nginx` `Certbot` `Let's encrypt`

---

|[Grafana and InfluxDB in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Grafana%20and%20InfluxDB%20in%20Docker.md)|[Table of Contents](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/README.md)|[Setup Grafana](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Setup%20Grafana.md)| 
---| ---| ---|

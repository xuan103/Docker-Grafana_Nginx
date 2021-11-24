|[Grafana and InfluxDB in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Grafana%20and%20InfluxDB%20in%20Docker.md)|[Table of Contents](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/README.md)|[Nginx and Certbot in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Nginx%20and%20Certbot%20in%20Docker.md)| 
---| ---| ---|

---

<h2 id="Contents">Table of Contents</h2>

-    [Setup Grafana](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Setup%20Grafana.md)
        - [Guide](#Guide)
        - [Settings for iframe sharing screen](#iframe)
        - [Anonymous access](#access)

---

<h2 id="Guide">Guide</h2>

1. Grafana dashboard hides the toolbar for easy web embedding
2. Avoid the login problem that the page does not display

---

<h2 id="iframe">Settings for iframe sharing screen</h2>

Edit the grafana.ini file, Uncomment and Change the [ line 241/1071 (22%) ]

> nano /home/user/your_path/grafana.ini

```shell=
# set to true if you want to allow browsers to render Grafana in a <frame>, <iframe>, <embed> or <object>. default is false.
;allow_embedding = false
```

>---

Revise:

>Note: Delete the beginning of ";".

```shell=
# set to true if you want to allow browsers to render Grafana in a <frame>, <iframe>, <embed> or <object>. default is false.
allow_embedding = true 
```

> Now reload nginx by doing a rough `sudo docker compose restart`

---

On Grafana Dashboard find `"Share dashboard or panel" > "Link" > "Link URL" > "Copy"`.

- Hide all toolbars: 
    - Add `"&kiosk"` after the link URL.
        - Ex: localhost:3000/d/vtw5T55nz/dashboard?&kiosk
- Hide left toolbar:
    - Add `"&kiosk=tv"` after the link URL.
        - Ex: localhost:3000/d/vtw5T55nz/dashboard?&kiosk=tv

---

<h2 id="access">Anonymous access</h2>

Edit the grafana.ini file, Uncomment and Change the [ line 387/1071 (36%) ]

> nano /home/user/your_path/grafana.ini

```shell=
#################################### Anonymous Auth ######################
[auth.anonymous]
# enable anonymous access
;enabled = false

# specify organization name that should be used for unauthenticated users
;org_name = Main Org.

# specify role for unauthenticated users
;org_role = Viewer

# mask the Grafana version number for unauthenticated users
;hide_version = false
```

>---

Revise:

> Note: Delete the beginning of ";".

```shell=
#################################### Anonymous Auth ######################
[auth.anonymous]
# enable anonymous access
enabled = true

# specify organization name that should be used for unauthenticated users
org_name = Main Org.

# specify role for unauthenticated users
org_role = Viewer

# mask the Grafana version number for unauthenticated users
hide_version = false

```

> Now reload nginx by doing a rough `sudo docker compose restart`, then open a new webpage and paste the grafana URL.

---

<h2 id="Related">Related Posts</h2>

-  Reference
    -   https://blog.csdn.net/weixin_41621706/article/details/100815603
    -   https://grafana.com/docs/grafana/latest/auth/grafana/

<h2 id="Appendix">Appendix and FAQ</h2>

:::info
**Find this document incomplete?** Leave a comment!
:::

###### tags: `Grafana` `Anonymous access` `iframe`

---

|[Grafana and InfluxDB in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Grafana%20and%20InfluxDB%20in%20Docker.md)|[Table of Contents](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/README.md)|[Nginx and Certbot in Docker](https://github.com/xuan103/Docker-Grafana_Nginx/blob/main/document/Nginx%20and%20Certbot%20in%20Docker.md)| 
---| ---| ---|

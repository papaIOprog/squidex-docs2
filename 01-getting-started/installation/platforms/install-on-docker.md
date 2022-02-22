---
description: Install Squidex on Linux machines with docker and docker-compose.
---

# Install on Docker

## Supported platforms

* Linux with [Docker CE](https://docs.docker.com/install/linux/docker-ce/centos/)
* Windows 10 Pro, Enterprise or Education with [Docker for Windows](https://docs.docker.com/docker-for-windows/install/)
* Windows with [Docker Toolbox](https://docs.docker.com/toolbox/toolbox\_install\_windows/)
* Mac with [Docker for Mac](https://docs.docker.com/docker-for-mac/)

{% hint style="info" %}
Digital Ocean [Droplets](https://www.digitalocean.com/products/droplets) are not supported right now, because their DNS prevents that a container can make a request to itself, which is needed to get OIDC via Identity Server working properly. The issue has been discussed in the [support forum](https://support.squidex.io/t/non-standard-port-installation/1262).
{% endhint %}

## Use the docker-compose setup

We provide a docker-compose configuration:

> [https://github.com/Squidex/squidex-hosting/tree/master/docker-compose](https://github.com/Squidex/squidex-hosting/tree/master/docker-compose)

There are 3 alternatives:

#### Squidex + Caddy

`docker-compose.yml` with the following containers:

* Squidex
* [Caddy ](https://caddyserver.com)as reverse proxy to support HTTPS. Also issues the certificate.
* [MongoDB](https://www.mongodb.com/de)

The caddy proxy uses a custom image to configure the Caddyfile.

{% hint style="info" %}
Recommended setup because of the performance of Caddy and the number of containers.
{% endhint %}

#### Squidex + NGINX

`docker-compose-nginx.yml` with the following containers:

* Squidex
* [NGINX ](https://www.nginx.com)as reverse proxy to support HTTPS
* NGINX sidecar to provision free and secure certificates with [LetsEncrypt](https://letsencrypt.org/de/).
* [MongoDB](https://www.mongodb.com/de)

The NGINX proxy uses a [custom image](https://github.com/Squidex/squidex-hosting/blob/master/docker-compose/proxy-nginx/Dockerfile) to increase the size of the http headers.

{% hint style="info" %}
Recommended setup when you are familiar with Nginx and have special requirements.
{% endhint %}

#### Squidex without Proxy

`docker-compose-noproxy.yml` with the following containers:

* Squidex
* [MongoDB](https://www.mongodb.com/de)

{% hint style="info" %}
Recommended setup if you already have a reverse proxy (e.g. Cloudflare).
{% endhint %}

### 1. Download the files

Download the following files to your server:

* `docker-compose.yml`
* `.env`

### 2. Configure Squidex

Open the `.env` file and set the following variables:

| Variable                | Description                                                                                                                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SQUIDEX_DOMAIN`        | Your domain name, that you want to use for your installation. For example the domain name for the Squidex cloud is `cloud.squidex.io`. If you can Squidex on your local machine it is `localhost`.         |
| `SQUIDEX_ADMINEMAIL`    | The email address of the admin user. You can leave it empty to create a new user with the setup page when you visit your Squidex installation in the browser.                                              |
| `SQUIDEX_ADMINPASSWORD` | The password of the admin user. Must contain a lowercase and uppercase letter, a number and a special character. You can leave it empty to create a new user with the setup page when you visit your Squid |

You can keep the other settings empty for now.

### 3. Data Folder

The data like assets and the MongoDB database files will be stored outside of the docker container to simplify the backups. The default path `/etc/squidex` will be created by docker automatically.

### 4. Run the docker-compose file

```bash
docker-compose up -d
```

### 5. Visit your installation

Squidex should be up and running now. You can visit your installation under&#x20;

[https://${SQUIDEX\_DOMAIN}](https://${squidex\_domain}).&#x20;

You should see the following screen:

![Setup Screen](<../../../.gitbook/assets/image (76).png>)

The setup screen shows a checklist with hints and warnings. As long as there is no error (an red icon), everything is fine.

If no external authentication provider such as Google or Github is configured you will not see the red area.

Create a new administrator account now with an email address and password and you are ready to go. We will not send you an email to this email address, so you can choose whatever email address you want.

## Troubleshooting

Please check the logs first using docker.

```bash
docker ps # Get the container id first
docker logs <CONTAINER-ID> # Read the logs
```

### I get a NET::ERR\_CERT\_AUTHORITY\_INVALID from the browser

You are very likely running under `localhost`. In this case the webserver (caddy) cannot create a valid certificate and will create a self signed certificate. Usually there is a button to continue to localhost:

![Accept self signed certificate with Chrome](<../../../.gitbook/assets/image (73).png>)

{% hint style="info" %}
This screen is taken from Chrome and can look differently for other browsers.
{% endhint %}

### I get a 502 Bad Gateway

In my tests it took sometime to issue the certificate. Probably around 10 minutes.

Also ensure that your DNS server is configured correctly.

### I cannot login and see a IDX20803 error code in my logs

In some cases, especially on CentOS 7, the communication between docker containers on the same host is blocked by the firewall. There is an open [issue on Github](https://github.com/moby/moby/issues/32138) for this problem.

The solution that worked in our cases was to add https as a service to the firewall:

```bash
sudo firewall-cmd --add-service=https --permanent --zone=trusted
sudo firewall-cmd --reload
```

### I see a IDX20803: Unable to obtain configuration from: \<IP> in the logs

This problem is because you use an host name or IP address that is not reachable from the docker itself. You can think about the Squidex being two processes in one application. There is the OpenID Connect Server (Identity Server) that generates the access tokens and the API. When the API receives an access token it makes a request to the Identity Server to validate the token (See following diagram).

![Authentication Flow](<../../../.gitbook/assets/Untitled presentation.png>)

When you use a local host name or IP address such as `localhost` or `127.0.0.1`your are referring to the host name. But containers inside docker cannot resolve the network routes and therefore the authentication flow fails. The solution is to either use another local hostname, that you have to configure in the host file of your Operation system or to use a real hostname, such as a public domain name.

### More issues?

It is very likely a configuration problem and not related to hosting under Docker. Checkout

{% content-ref url="../configuration.md" %}
[configuration.md](../configuration.md)
{% endcontent-ref %}
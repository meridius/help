---
layout: article
title: Certificate Options
categories: [hosting]
featured: false
popular: false
tags: [hosting, docker, install, deploy]
order: 04
---

This article defines the certificate options available to Self-hosted instances of Bitwarden. You will select your certificate option during installation. For installation instructions, see [Install and Deploy On-premise]({% link _articles/hosting/install-on-premise.md %}).

## Generate a Certificate with Let's Encrypt

[Let's Encrypt](https://letsencrypt.org/how-it-works/){:target="_blank"} is a Certificate Authority (CA) that issues trusted SSL certificates free of charge for any domain. The Bitwarden installation script offers the option to generate a trusted SSL certificate for your domain using Let's Encrypt and [Certbot](https://certbot.eff.org){:target="_blank"}.

Certificate renewal checks occur each time Bitwarden is restarted. Using Let's Encrypt will require you to enter an email address for certificate expiration reminders.

Using Let's Encrypt requires ports 80 and 443 to be open on your machine.

### Manually Update a Let's Encrypt Certificate

If you change the domain name of your Bitwarden server, you will need to manually update your generated certificate. Run the following commands to create a backup, update your certificate, and rebuild Bitwarden:

```
./bitwarden.sh stop
mv ./bwdata/letsencrypt ./bwadata/letsencrypt_backup
mkdir ./bwdata/letsencrypt
chown -R bitwarden:bitwarden ./bwdata/letsencrypt
chmod -R 740 ./bwdata/letsencrypt
docker pull certbot/certbot
docker run -i --rm --name certbot -p 443:443 -p 80:80 -v <Full Path from / >/bwdata/letsencrypt:/etc/letsencrypt/ certbot/certbot certonly --logs-dir /etc/letsencrypt/logs
Select 1, then follow instructions
openssl dhparam -out ./bwdata/letsencrypt/live/<your.domain.com>/dhparam.pem 2048
./bitwarden.sh rebuild
./bitwarden.sh start
```

## Use an Existing SSL Certificate

You may alternatively opt to use an existing SSL Certificate, which will require you to have the following files:

- A Server Certificate (`certificate.crt`)
- A Private Key (`private.key`)
- A CA Certificate (`ca.crt`)

You may need to bundle your primary certificate with Intermediate CA certificates to prevent SSL trust errors. All CA Certificates should be included in the CA Certificate Chain file when using a Root CA and Intermediate CA certificate.


Under the default configuration, place your files in `./bwdata/ssl/your.domain`. You may specify a different location for your certificate files by editing the following values in `./bwdata/config.yml`:

```
ssl_certificate_path: <path>
ssl_key_path: <path>
ssl_ca_path: <path>
```

{% callout info %}
The values defined in `config.yml` represent locations inside the NGINX container. Directories on the host are mapped to directories within the NGINX container. Under the default configuration, mappings line up as follows:

The following values in `config.yml`:
```
ssl_certificate_path: /etc/ssl/your.domain/certificate.crt
ssl_key_path: /etc/ssl/your.domain/private.key
ssl_ca_path: /etc/ssl/your.domain/ca.crt
```

Map to the following files on the host:
```
./bwdata/ssl/your.domain/certificate.crt
./bwdata/ssl/your.domain/private.key
./bwdata/ssl/your.domain/ca/crt
```

**You should only ever need to work with files in `./bwdata/ssl/`. Working with files directly in the NGINX container is not recommended.**
{% endcallout %}

### Using Diffie-Hellman Key Exchange

*Optionally*, if using Diffie Hellman Key Exchange to generate ephemeral parameters:

- Include a `dhparam.pem` file in the same directory.
- Set the `ssl_diffie_hellman_path:` value in `config.yml`.

{% callout info %}
You can create your own `dhparam.pem` file using OpenSSL with `openssl dhparam -out ./dhparam.pem 2048`.
{% endcallout %}

## Generate a Self-signed Certificate

You may alternatively opt to use a self-signed certificate, however this is only recommended for testing.

Self-signed certificates will not be trusted by Bitwarden client applications by default. You will be required to manually install this certificate to the trusted store of each device you plan to use Bitwarden with.

Your self-signed certificate (`.crt`) and private key (`private.key`) can be retrieved from the `./bwdata/ssl/self/your.domain` directory.

### Trust a Self-signed Certificate

#### Windows

To trust a self-signed certificate on Windows, run `certmgr.msc` and import your certificate into the Trusted Root Certification Authorities.

#### Linux

To trust a self-signed certificate on Linux, add your certificate to the following directories:

```
/usr/local/share/ca-certificates/
/usr/share/ca-certificates/
```

And run the following commands:

```
sudo dpkg-reconfigure ca-certificates
sudo update-ca-certificates
```

## Use no Certificate

{% callout warning %}
If you opt to use no certificate, you **must front your installation with a proxy that serves Bitwarden over SSL**. This is becacuse Bitwarden requires HTTPS; trying to use Bitwarden without the HTTPS protocol will trigger errors.
{% endcallout %}

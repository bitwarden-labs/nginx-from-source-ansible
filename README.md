# Reference nginx reverse proxy implementation

This repo contains an ansible role for a reference nginx installation for use as a reverse proxy, directing traffic to a Bitwarden server, as well as other example sites and services.

The role uses certbot + letsencrypt to manage certificates via an http challenge, with logic to start and stop nginx when a new certificate is obtained.

Hosted sites are configured via machine variables, and examples are given for both static pages served from the nginx server's disk, as well as proxying to additional services.

There is also a setup section to allow for configuration of basic machine settings.  

As with all ansible roles, the modular nature of the playbook contained should allow for simple customisation and alteration.  A simple example of this would be to obtain certs via DNS challenge instead of HTTP challenge, as used in [this demonstration role](https://github.com/bitwarden-labs/bws-ansible-examples/blob/main/certbot-nginx-vikunja/README-certbot-nginx-vikunja.md).

## Playbook Breakdown

Documentation for each playbook included in the role:

### certbot

This playbook installs certbot using apt, ensuring that directories are available for the generated certs.  It also checks for the existence of a DH params file, and generates this if necessary.

systemd configs are generated for the certbot service itself, as well as for certificate renewal.

### inventory

This folder contains the inventory upon which the roles will be played out - by default this is targetting a single machine called 'nginx-gateway'

nginx-gateway.yml contains the keys used to set the variables used by the rest of the roles, including:

- machine config settings such as hostname and networking config
- letsencrypt account settings
- nginx installation parameters
- nginx location settings

### machine-setup

This playbook sets the machine's hostname, and ensures that the machine can resolve this via /etc/hosts to allow it to serve static content.

It also sets the networking settings via Netplan, and allows for updates if necessary.

### nginx-sites

This playbook contains plays to deploy the nginx config files and (if applicable), static content defined in the inventory.  There is also logic to generate new certificates using letsencrypt when new locations are found.

If a new cert needs generating, there is logic to ensure that nginx is stopped before attempting to do so, freeing up port 80 as is required for the DNS challenge.

### nginx

This playbook contains plays to install nginx from source, using configuration settings defined in the inventory.  In particular, the real-ips module required to pass real-ips onto a backend Bitwarden server is installed.

Upgrades of the nginx server are supported by specifying a different nginx version number in the inventory.

### static-files

This role contains the static files to be served directly from the server, as well as plays to copy them into the correct location.

### users-groups

This role contains plays to ensure that correct users are present on the server, and in particular that the www-data user used by nginx has access to the letsencrypt certificates as part of the letsencrypt group.

### node-exporter

This role contains plays to perform a basic installation of node-exporter for monitoring.

### fail2ban

This role contains plays to perform a basic installation of fail2ban, tailored to the nginx reverse-proxy service.

## Dependancies

This role is tested on Debian 12, but should work on any distribution using apt and systemd.

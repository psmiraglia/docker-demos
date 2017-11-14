# LAMP stack demo

This demo runs a LAMP stack composed of:

-   a load balancer (HAProxy)
-   two instances of a PHP application (phpmyadmin)
-   a SQL database (MariaDB)

The PHP application will be available on port 80 while the load balancer
statistics on port 9000 at URI `/haproxy?stats` (credentials: admin/admin).

## Prerequisites

We assume a clean installation of Ubuntu 16.04 LTS.

1.  Install Docker engine (see [Install using the repository] on Docker pages)

        $ sudo apt-get remove docker docker-engine docker.io
        $ sudo apt-get update
        $ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
        $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        $ sudo apt-get update
        $ sudo apt-get install docker-ce

2.  Install Docker compose (see [Install Compose] on Docker pages)

        $ sudo curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose
        $ sudo chmod +x /usr/local/bin/docker-compose

[Install using the repository]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository
[Install Compose]: https://docs.docker.com/compose/install/#install-compose

## Run the demo

1.  Start the LAMP stack

        $ docker-compose up

2.  Kill the master PHP instances

        $ docker-compose stop php_master

3.  Restart the master PHP instance

        $ docker-compose start php_master

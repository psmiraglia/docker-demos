# Static web site behind a load balancer

This demo runs a stack composed of

*   a load balancer implemented with [HAProxy]

*   a static web site served with [Httpd] and replicated over
    two instances in roundrobin

The whole stack is deployed by using [Docker Compose].

## Prerequisites

*   Online Linux box (e.g. Ubuntu)

## Install Docker engine/compose

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

## Demo steps

1.  Build the Docker images

        $ sudo docker build --tag demo/haproxy ./haproxy
        $ sudo docker build --tag demo/httpd ./httpd

    you should obtain something like that

        $ sudo docker images
        REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
        demo/httpd          latest              3435fec7ae99        4 seconds ago       86.6MB
        demo/haproxy        latest              1722e59f2cc6        19 seconds ago      14.7MB
        httpd               alpine              fe2c83c5d97d        9 days ago          86.6MB
        haproxy             alpine              81866058e8a3        9 days ago          14.7MB

    Alternatively, you can pass the `--build` option to `docker-compose` in
    order to build the images at deploy time.

2.  Deploy and run the stack

        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose up

    or

        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose up --build

    you should see someting like that

        Creating staticwebsitebehindloadbalancer_website_master_1 ... 
        Creating staticwebsitebehindloadbalancer_website_slave_1 ... 
        Creating staticwebsitebehindloadbalancer_website_master_1
        Creating staticwebsitebehindloadbalancer_website_slave_1 ... done
        Creating staticwebsitebehindloadbalancer_loadbalancer_1 ... 
        Creating staticwebsitebehindloadbalancer_loadbalancer_1 ... done
        Attaching to staticwebsitebehindloadbalancer_website_master_1, staticwebsitebehindloadbalancer_website_slave_1, staticwebsitebehindloadbalancer_loadbalancer_1
        website_master_1  | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
        website_master_1  | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.2. Set the 'ServerName' directive globally to suppress this message
        website_master_1  | [Mon Nov 13 15:43:48.888597 2017] [mpm_event:notice] [pid 1:tid 140445102750536] AH00489: Apache/2.4.29 (Unix) configured -- resuming normal operations
        website_master_1  | [Mon Nov 13 15:43:48.888654 2017] [core:notice] [pid 1:tid 140445102750536] AH00094: Command line: 'httpd -D FOREGROUND'
        website_slave_1   | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.3. Set the 'ServerName' directive globally to suppress this message
        website_slave_1   | AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.19.0.3. Set the 'ServerName' directive globally to suppress this message
        website_slave_1   | [Mon Nov 13 15:43:48.935091 2017] [mpm_event:notice] [pid 1:tid 140289125288776] AH00489: Apache/2.4.29 (Unix) configured -- resuming normal operations
        website_slave_1   | [Mon Nov 13 15:43:48.935146 2017] [core:notice] [pid 1:tid 140289125288776] AH00094: Command line: 'httpd -D FOREGROUND'
        loadbalancer_1    | <7>haproxy-systemd-wrapper: executing /usr/local/sbin/haproxy -p /run/haproxy.pid -f /usr/local/etc/haproxy/haproxy.cfg -Ds 
        loadbalancer_1    | [WARNING] 316/154349 (8) : config : log format ignored for proxy 'stats' since it has no log address.
        loadbalancer_1    | [WARNING] 316/154349 (8) : config : log format ignored for frontend 'website-fe' since it has no log address.

    To run `docker-compose` as daemon, use `-d` switch
        
        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose up -d --build

    In this case, the list of running containers can be obtained with

        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose ps
        Name                                    Command               State         Ports       
        --------------------------------------------------------------------------------------------------------------
        staticwebsitebehindloadbalancer_loadbalancer_1     /docker-entrypoint.sh hapr ...   Up      0.0.0.0:80->80/tcp
        staticwebsitebehindloadbalancer_website_master_1   httpd-foreground                 Up      80/tcp            
        staticwebsitebehindloadbalancer_website_slave_1    httpd-foreground                 Up      80/tcp

    or

        $ sudo docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
        9d46165b3049        demo/haproxy        "/docker-entrypoin..."   3 minutes ago       Up 51 seconds       0.0.0.0:80->80/tcp   staticwebsitebehindloadbalancer_loadbalancer_1
        c0a5313159a3        demo/httpd          "httpd-foreground"       4 minutes ago       Up 51 seconds       80/tcp               staticwebsitebehindloadbalancer_website_slave_1
        836dbbdf3721        demo/httpd          "httpd-foreground"       4 minutes ago       Up 51 seconds       80/tcp               staticwebsitebehindloadbalancer_website_master_1

3.  Check the deployed web site.

        $ browser http://127.0.0.1

4.  Update the website

        $ editor httpd/src/index.html

5.  Rebuild everything

        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose stop
        $ sudo docker-compose rm -f
        $ sudo docker-compose up --build

6.  Stop (and start) a replica
        
        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose stop website_master
        
        $ cd /path/to/docker-compose.yaml
        $ sudo docker-compose start website_master

7.  Access the Docker system

        $ sudo docker-compose exec website_slave_1 /bin/sh

## References

*   [Docker Compose]
*   [Docker Engine](https://docs.docker.com/engine)
*   [HAProxu Docker image](https://hub.docker.com/_/haproxy)
*   [HAProxy]
*   [Httpd Docker image](https://hub.docker.com/_/httpd)
*   [Httpd]
*   [Install Docker Compose](https://docs.docker.com/compose/install)
*   [Install Docker Engine](https://docs.docker.com/engine/installation)

[HAProxy]: http://www.haproxy.org
[Httpd]: https://httpd.apache.org/docs/2.4/programs/httpd.html
[Docker Compose]: https://docs.docker.com/compose

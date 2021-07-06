# POC-NAP

### File Structure

```
nginx-plus/
└── etc/
    ├── app_protect/
    │   └── conf/.......................................The default policies and log format provided by App Protect out of box.
    │       ├── NginxDefaultPolicy.json.................Default base policy provided by App Protect
    │       ├── NginxStrictPolicy.json..................Default Strict policy provided by App Protect
    │       └── log_default.json........................Default log format provided by App Protect
    ├── nginx
    │   ├── conf.d
    │   │   ├── health_checks.conf......................Custom Health check match blocks
    │   │   ├── status_api.conf.........................NGINX Plus Live Activity Monitoring available on port 8080 - [Source](https://gist.github.com/nginx-gists/│a51 341a11ff1cf4e94ac359b67f1c4ae)
    │   │   ├── upstreams.conf..........................Upstream configurations
    │   │   └── www.example.com.conf....................HTTP www.example.com Virtual Server configuration
    │   ├── includes
    │   │   ├── add_headers
    │   │   │   └── security.conf.......................Recommended response headers for security
    │   │   ├── nap.d
    │   │   │   └── logformats
    │   │   │       └── config-illegal-requests.json....User-defined custom log format for App Protect
    │   │   ├── proxy_headers
    │   │   │   ├── keepalive.conf......................Common keepalive configurations
    │   │   │   └── proxy_headers.conf..................Common proxy headers
    │   │   └── ssl
    │   │       ├── ssl_a+_strong.conf..................Recommended SSL configuration for Based on SSL Labs A+ (https://www.ssllabs.com/ssltest/)
    │   │       ├── ssl_intermediate.conf...............Recommended SSL configuration for General-purpose servers with a variety of clients, recommended for almost all systems
    │   │       ├── ssl_modern.conf.....................Recommended SSL configuration for Modern clients: TLS 1.3 and don't need backward compatibility
    │   │       └── ssl_old.conf........................Recommended SSL configuration for compatiblity ith a number of very old clients, and should be used only as a last resort
    └── ssl/
        ├── dhparam/
        │   ├── 1024/...................................Diffie-Hellman parameters 1024 key size
        │   │   └── dhparam.pem.........................Diffie-Hellman parameters file (1024 bit)
        │   ├── 2048/...................................Diffie-Hellman parameters 2048 key size
        │   │   └── dhparam.pem.........................Diffie-Hellman parameters file (2048 bit)
        │   └── 4096/...................................Diffie-Hellman parameters 4096 key size
        │       └── dhparam.pem.........................Diffie-Hellman parameters file (4096 bit)
        ├── example.com
        │   ├── example.com.crt.........................Self-signed wildcard certifcate for testing (*.example.com)
        │   └── example.com.key.........................Self-signed private key for testing
        └── nginx
            ├── nginx-repo.crt..........................NGINX Plus repository certificate file (**Use your own license**)
            └── nginx-repo.key..........................NGINX Plus repository key file (**Use your own license**)
```

## Prerequisites:

1. NGINX evaluation license file. You can get it from [here](https://www.nginx.com/free-trial-request/)

2. A Docker host. With [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)

3. **Optional**: The demo uses hostnames: `www.example.com`. For host name resolution you will need to add hostname bindings to your hosts file:

For example on Linux/Unix/MacOS the host file is `/etc/hosts`

```bash
# NGINX Plus demo system (local docker host)
127.0.0.1 www.example.com
```

## Build and run the demo environment

Provided the Prerequisites have been met before running the steps below, this is a **working** environment.

### Build the demo

In this demo we will have a one NGINX Plus ADC/load balancer (`nginx-plus`) and two backend apps (`datacenter1` and `datacenter2`) which is built using [DSVW repository](https://github.com/stamparm/DSVW)

Before we can start, we need to copy our NGINX Plus repo key and certificate (`nginx-repo.key` and `nginx-repo.crt`) into the directory, `nginx-plus/etc/ssl/nginx/`, then build our stack:

```bash
# Enter working directory
cd POC_NAP

# Make sure your Nginx Plus repo key and certificate exist here
ls nginx-plus/etc/ssl/nginx/nginx-*
nginx-repo.crt              nginx-repo.key

# Downloaded docker images and build
docker-compose pull
docker-compose build --no-cache
```

#### Start the Demo stack:

Run `docker-compose` in the foreground so we can see real-time log output to the terminal:

```bash
docker-compose up
```

Or, if you made changes to any of the Docker containers or NGINX configurations, run:

```bash
# Recreate containers and start demo
docker-compose up --force-recreate
```

**Confirm** the containers are running. You should see three containers running:

```bash
$> docker ps
                                                                                                                                                  
CONTAINER ID   IMAGE                         COMMAND                  CREATED              STATUS                                 PORTS                                                              NAMES
a68176aef1d3   POC_NAP_nginx-plus    "sh /entrypoint.sh"      About a minute ago   Up About a minute                      0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp   nginx-plus
4ff6a8e3357a   sebp/elk:792          "/usr/local/bin/star…"   About a minute ago   Up About a minute (health: starting)   5044/tcp, 9200/tcp, 9300/tcp, 9600/tcp, 0.0.0.0:5601->5601/tcp     elk
748727c5eb35   POC_NAP_datacenter2   "python3 dsvw.py"        About a minute ago   Up About a minute                      0.0.0.0:8200->65412/tcp                                            datacenter2
d89fe72f0e56   POC_NAP_datacenter1   "python3 dsvw.py"        About a minute ago   Up About a minute                      0.0.0.0:8100->65412/tcp                                            datacenter1
```

The demo environment is ready in seconds. 
- You can access the demo website on **HTTPS / Port 443** ([`https://localhost`](https://localhost) or [https://www.example.com](https://www.example.com)) 
- The NGINX API can be accessed on **HTTP / Port 8080** ([`http://localhost:8080`](http://localhost:8080))
- The NGINX Dashboard can be accessed on **HTTP / Port 8080** ([`http://localhost:8080/dashboard.html`](http://localhost:8080/dashboard.html))
- The ELK/Kibana Dashboard can be accessed on **HTTP / Port 5601** ([`http://localhost:5601`](http://localhost:5601))

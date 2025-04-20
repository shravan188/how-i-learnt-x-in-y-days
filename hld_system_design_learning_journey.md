

## Day 1

* Important concepts in system design include (refer 5):
    * CAP theorem
    * Idempotency and Concurrency handling
    * Message queues like RabbitMQ, Kafka (part of asynchronism)
    * Load balancer 
    * Proxy and Reverse proxy
    * Caching 
    * Database scaling : Indexing, Materialised views, Denormalization, Vertical scaling, Replication, Sharding
    * API architecture styles : SOAP, RESTful, GraphQL, gRPC, Websocket, Webhook
    * Asynchronism (including task queues and message queues)
    * Microservices patterns : Decomposition, SAGA, Strangler, CQRS (there are others like Service Registry, Circuit breaker, API gateway, Event driven architecture, Database per service, Externalized configuration, Bulkhead pattern, BFF)
    

* Distributed system : Collection of independent computers that appears to user as one computer (we can understand the need for distributed system from Ice Cream problem a real world situation)

* For example we create a simple FastAPI website. If only a few people are using one computer/server is enough. But if millions of people are using, it may not be able to handle the load, so we have multiple computers, each of them having the same FastAPI code, and giving the same response to the user input

* Consistency : Ensures all nodes have the same view of data at a given time. If you're interacting with different servers or services, you'll always get the same view of your data during your session (includes different patterns like weak consistency, eventual consistency and strong consistency, each having its own use case)

* Availability : System's ability to remain operational and accessible despite failures or disruptions

* CAP theorem : A distributed system can only provide two of three guarantees: consistency, availability, and partition tolerance

* Problem statement : Create an API that uploads a large CSV file (>10GB) to a server, then reads the file, normalize the data and store it in a database (refer 1)

### Doubts
1. Why is cache faster than database? (in memory vs disk)
2. What exactly is partition tolerance?
3. Are microservices overrated?

### References
1. https://www.youtube.com/watch?v=CESKgdNiKJw
2. https://github.com/donnemartin/system-design-primer
3. https://fastapi.tiangolo.com/deployment/docker/
4. https://web.archive.org/web/20221030091841/http://www.lecloud.net/tagged/scalability/chrono
5. https://www.youtube.com/watch?v=mI73eTlSqeU&list=PL6W8uoQQ2c63W58rpNFDwdrBnq5G3EfT7 (Concept and Coding By Shrayansh Jain)
6. https://www.reddit.com/r/webdev/comments/83tjhn/are_microservices_overrated/

## Day 2 and 3

* CAP theorem : A distibuted system can provide 2 of the 3 guarantees : consistency, availability and partition tolerance

* Partition tolerance : Means even if the nodes of the distributed system are not able to communicate with each other, the entire system as a whole should still remain functional for the end user (because end user does not see or know they are interacting with a distributed system, for them it appears as a single system)

* Intution behind CAP theorem : Consider a backend server communicating with 2 databases (say P and Q) that are replicas of each other. Consistency means both databases must have same data, availability means the database which is pionged by server should reply
    * CP (Consistency-Parition tolerance) : If communication is lost b/w P and Q, then only way system as a whole can be consistent is backend must always query only P, as Q will not have the latest updates done in P. This implies we have lost availability as Q is not avilable
    * AP (Availability-Partition tolerance) : If communication is lost b/w P and Q, but both of them are available, then we backend server will get different results when querying P versus when querying Q, hence consistency is lost

* Note that Consistency and Availability without Partion tolerance does not make sense to a distributed system (as partition tolerance is must for distributed) and is applicable mainly for a single node system


* Experimented with building a high availablility POstgres cluster on Docker using Patroni, etcd and HAProxy

* Each postgres instance needs its own data directory, but each instance needs to run on the same network




* --rm flag : is used when you need the container to be deleted after the task for it is complete. This is suitable for small testing or POC purposes.

* postgres : postgres is the PostgreSQL database server. In order for a client application to access a database it connects (over a network or locally) to a running postgres instance. The postgres instance then starts a separate server process to handle the connection. One postgres instance always manages the data of exactly one database cluster

* PGDATA : A database cluster is a collection of databases that is stored at a common file system location (the “data area”). When postgres starts it needs to know the location of the data area. The location must be specified by the -D option or the PGDATA environment variable; there is no default. 

```
docker run -it --rm --name postgres-1 --net postgres-replication -e POSTGRES_USER=postgresadmin -e POSTGRES_PASSWORD=admin123 -e POSTGRES_DB=postgresdb -e PGDATA="/data" -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\data:/data -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\config:/config -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\archive:/mnt/server/archive -p 5000:5432 postgres:15.0 -c "config_file=/config/postgresql.conf"



docker run -it --rm --net postgres-replication -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-2\data:/data --entrypoint /bin/bash postgres:15.0


docker run -it --rm --name postgres-2 --net postgres-replication -e POSTGRES_USER=postgresadmin -e POSTGRES_PASSWORD=admin123 -e POSTGRES_DB=postgresdb -e PGDATA="/data" -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-2\data:/data -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-2\config:/config -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-2\archive:/mnt/server/archive -p 5001:5432 postgres:15.0 -c "config_file=/config/postgresql.conf"

docker run -it --rm --name postgres-1 --net postgres-replication -e POSTGRES_USER=postgresadmin -e POSTGRES_PASSWORD=admin123 -e POSTGRES_DB=postgresdb -e PGDATA="/data" -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\data:/data -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\config:/config -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\archive:/archive -v C:\Users\dell\Documents\HLD\postgres-replication\postgres-1\config:/etc/postgresql.conf -p 5000:5432 postgres:15.0 


```


* Error : 
    * docker: Error response from daemon: .\postgres-3\data%!(EXTRA string=is not a valid Windows path).
    * docker: Error response from daemon: create postgres-3/data: "postgres-3/data" includes invalid characters for a local volume name, only "[a-zA-Z0-9][a-zA-Z0-9_.-]" are allowed. If you intended to pass a host directory, use absolute path.
    * Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser.
    * docker: invalid reference format: repository name must be lowercase.
    * postgres: could not access the server configuration file "/\config\postgressql.config": No such file or directory

* Setting up a highly available Postgres cluster using Patroni, etcd and HAProxy
docker network prune
docker network Error response from daemon: Pool overlaps with other one on this address space
md5003c7242487c89ca07282f6e048a4821

* client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:4001: connect: connection refused; error #1: client: endpoint http://127.0.0.1:2379 exceeded header timeout


* Each Patroni instance monitors the health data of a PostgreSQL instance.
The health data needs to be stored somewhere where all other Patroni instances can access it.

* docker network inspect pg_network

### Doubts
1. Why do we need configuration files in Postgres?
2. Whats-the-correct-way-to-mount-a-volume-on-docker-for-windows?
3. What is source command in linux?
4. How to change a leader in etcd cluster?
### References
1. https://www.youtube.com/watch?v=3qRBeZsUa18&list=PL6W8uoQQ2c63W58rpNFDwdrBnq5G3EfT7&index=3
2. https://stackoverflow.com/questions/49726272/what-is-the-rm-flag-doing
3. https://www.postgresql.org/docs/current/app-postgres.html
4. https://forums.docker.com/t/whats-the-correct-way-to-mount-a-volume-on-docker-for-windows/58494
5. https://www.postgresql.fastware.com/postgresql-insider-ha-str-rep
6. https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/storage/databases/postgresql/3-replication

1. https://www.youtube.com/watch?v=lbldTB0GuNY (Extend c drive
2. https://www.bodhost.com/kb/how-to-save-and-exit-in-nano-editor)
3. https://stackoverflow.com/questions/56515128/error-pool-overlaps-with-other-one-on-this-address-space-when-starting-my-proje
4. https://stackoverflow.com/questions/51624598/why-use-etcdcan-i-use-redis-to-implement-configuration-management-service-disco
5. https://etcd.io/docs/v3.4/dev-guide/interacting_v3/
6. https://www.youtube.com/watch?v=uC1WPxFzISQ
7. https://www.med.unc.edu/it/guide/operating-systems/how-do-i-find-the-host-name-ip-address-or-physical-address-of-my-machine/
8. https://stackoverflow.com/questions/1347282/how-can-i-get-a-list-of-all-functions-stored-in-the-database-of-a-particular-sch
9. https://www.cybertec-postgresql.com/en/introduction-and-how-to-etcd-clusters-for-patroni/
10. https://www.crunchydata.com/blog/patroni-etcd-in-high-availability-environments
11. https://stackoverflow.com/questions/47807892/how-to-access-kubernetes-keys-in-etcd
12. https://techsupportpk.blogspot.com/2022/02/set-up-highly-available-postgresql-cluster-docker-ubuntu.html
13. https://www.youtube.com/watch?v=A_t_ytq1lpA / https://github.com/dem-linux/patroni-postgres/blob/main/README.md
14. https://www.crio.do/blog/learn-load-balancer-using-haproxy/
15. https://parottasalna.com/2024/09/10/haproxy-ep-2-tcp-proxy-for-flask-application/









## Day 4
* Goal : To configure HAProxy with Flask

* HTTP is in the application layer of the OSI model, whereas TCP is at the transport layer

* HAProxy as a TCP proxy operates at Layer 4 (Transport Layer) of the OSI model. It forwards raw TCP connections from clients to backend servers.This is ideal for scenarios where:
    * You need to handle non-HTTP traffic, such as databases or other TCP-based applications.
    * You want to perform load balancing without application-level inspection.
    * Your services are using protocols other than HTTP/HTTPS.

* HAProxy can’t read the packets but can identify the ip address of the client

* Errors: 
    * failed to solve with frontend dockerfile.v0: failed to read dockerfile: open /var/lib/docker/tmp/buildkit-mount3676674688/Dockerfile: no such file or directory : DOCKERFILE instead of Dockerfile
    * json: cannot unmarshal array into Go value of type types.ImageInspect : Changed from image:. to build :.  (refer 2)
    * services.haproxy Additional property container is not allowed : Changed container to container_name
    * unknown keyword 'defaults:' in 'global' section
    * parsing [/usr/local/etc/haproxy/haproxy.cfg:10] : 'bind' : missing port number: '*:'
    * Server web2/web2 is DOWN, reason: Layer4 connection problem, info: "Connection refused", check duration: 0ms. 0 active and 0 backup servers left. 0 sessions active, 0 requeued, 0 remaining in queue.

* Docker image : A blue-print of application (an executable package that contains code, runtime, libraries)

* Docker container : Actual running application in an isolated environment

* Some analogy:
    * Image is the app/program, container is the running app/program.
    * Image is the cookie recipe, container is the cookie
    * Image is the bootable pendrive, container is the actual running system

```
## app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_geek():
    return '<h1>Hello from Flask and Docker. Hope you have a great day</h1>'


if __name__ == "__main__":
    app.run(host ='0.0.0.0', port = 5001, debug = True) 

```

```
## Dockerfile
FROM python:3.9-alpine3.20

WORKDIR /flask-app

COPY app.py app.py

RUN pip3 install flask

CMD ["python3","app.py"]

```


```
docker build --tag my-flask-docker .  

docker images 

# map port 4999 of host system to port 5001 of container
docker run -d -p 4999:5001 my-flask-docker

# list all running containers
docker ps

curl localhost:4999 # <h1>Hello from Flask and Docker. Hope you have a great day</h1>

docker stop <container-id>

docker rm <container-id>

```

* Load balancing bw 2 Flask web servers using HAProxy

```
## haproxy/Dockerfile
FROM haproxy:2.2.33

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg

## haproxy/haproxy.cfg

global

defaults
    mode http
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

frontend http-in
    bind *:5001
    default_backend web1

backend web1
    balance roundrobin
    server web1 web1:5001 check
    server web2 web2:5001 check

```


```
# docker-compose.yml
# Dockerfile and app.py are same as in the previous example

version: "2"

services:
  web1:
    build: .
    container_name: web1
    ports:
      - "4900:5001"

  web2:
    build: .
    container_name: web2
    ports:
      - "4901:5001"

  haproxy:
    build: ./haproxy
    container_name: haproxy
    ports:
      - "5001:5001"

```


```
# Folder structure
app.py
│   docker-compose.yml
│   Dockerfile
│
└───haproxy
        Dockerfile
        haproxy.cfg


```

```

docker-compose up 

# docker network prune (remove unused network)

docker-compose down 

```

* Had to chose version 2 of docker compose as for docker-compose version 2 file format, you can build and tag an image for one service and then use that same built image for another service. From 1.6 onwards, we can give path for image key

* ACL (Access Control List) : Examines a statement and returns either true or false. You can use ACLs in many scenarios, including routing traffic, blocking traffic, and transforming messages. Used with an if statement. Tried using ACL for routing traffic as shown below, but did not work

```
# haproxy.cfg (not working as expected)
global

defaults
    mode http
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

frontend http-in
    bind *:5001

    acl has_web1 path_beg /web1
    acl has_web2 path_beg /web2
    use_backend web1 if has_web1
    use_backend web2 if has_web2
    default_backend web1

backend web1
    server web1 web1:5001 check
    
backend web2
    server web2 web2:5001 check


```

* Haproxy has 3 proxies : frontend, backend, listen

* chroot (Linux command) : chroot is an operation on Unix and Unix-like operating systems that changes the apparent root directory for the current running process and its children.

### Doubts
1. HAProxy vs Flask - when to use which?
2. Why can we use HAProxy to handle HTTP traffic?
3. What is the different bw different Python docker images like alpine, slimbookworm etc?
4. Why we need to mention host='0.0.0.0' in app.run?
5. How to use Dockerfile image in docker-compose?
6. What is Difference between frontend/backend and listen in haproxy?
7. What is dig in Ubuntu, and how to run it on windows?
8. Why use pgbouncer?
9. Does docker compose up automatically do docker compose build? Suppose we change a file say haproxy.cfg, do we need to build again? 
10. How are chroot and docker similar? How are they different?

### References
1. https://www.freecodecamp.org/news/how-to-dockerize-a-flask-app/
2. https://github.com/docker/compose/issues/9441
3. https://www.youtube.com/watch?v=PtT32MW2j9c (How to Compose Multiple Web Apps With Docker)
4. https://stackoverflow.com/questions/39209917/difference-between-frontend-backend-and-listen-in-haproxy
5. https://stackoverflow.com/questions/5673335/how-to-configure-haproxy-to-send-get-and-post-http-requests-to-two-different-app
6. https://github.com/jonnylangefeld/docker-load-balance-test/tree/remove-unnecessary-stuff
7. https://parottasalna.com/2024/09/10/haproxy-ep-2-tcp-proxy-for-flask-application/
8. https://www.reddit.com/r/docker/comments/uc99p3/docker_compose_do_i_need_to_run_docker_compose/?rdt=50925


## Day 5
* Haproxy key concepts : Configuration file, defaults, frontend, backend, listen, acl (access control list), resolver, balance(load balancing algorithms), retries, redispatch

* Frontend : A frontend section defines the IP addresses and ports that clients can connect to

* Backend : Backend section defines a pool of servers to which the load balancer will route requests

* Listen : Listen section serves as both a frontend, which listens to incoming traffic, and a backend, which specifies the web servers to which the load balancer sends the traffic. It's paricularly useful for TCP because such configurations are usually simpler than HTTP.

* Below is an example of replacing frontend+backend with listen

```
## haproxy.cfg (old)
global
    log stdout format raw daemon debug

defaults
    log global
    mode http
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

frontend http-in
    bind *:5001
    default_backend web1

backend web1
    balance roundrobin
    server web1 web1:5001 check
    server web2 web2:5001 check


## haproxy.cfg (new)
global
    log stdout format raw daemon debug

defaults
    log global
    mode http
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

listen http-proxy
    bind *:5001
    balance roundrobin
    server web1 web1:5000 check
    server web2 web2:5000 check


```

* Frontend/Backend vs Listen : In a frontend section, you need default_backend web, explicitly defining where traffic should go. In a listen section, the servers inside it are automatically treated as the backend.

* Because of previous point, we have to define a server in listen proxy, else we get `503 Service Unavailable : No server is available to handle this request` (we can add maintenance file for this scenario using errorfile keyword)

* Proxy modes : http and tcp

* Load balancing algorithms available for backend proxy are (refer 9 for detailed design considerations):
    * roundrobin : send traffic in a round robin fashion
    * leastconn : send traffic to the server with the fewest number of connections.
    * static-rr
    * random
    * first : The first server with available connection slots receives theconnection
    * source

* Resolver : Resolvers section lists DNS nameservers that the load balancer will query when it needs to resolve a hostname to an IP address. A DNS nameserver is a specialized server within the Domain Name System (DNS) that translates human-readable domain names (like example.com) into IP addresses (like 192.0.2.1)

* **The key reason the haproxy config file with acl was not working on Day 4 was not because of any error in HAProxy, but it was because the routes were not defined in the Flask application. Once I added /web1 and /web2 routes to the Flask app, it started working as expected**

```
## haproxy/haproxy.cfg
global
    log stdout format raw daemon debug

defaults
    log global
    mode http
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

frontend http-in
    bind *:5001

    acl has_web1 path_beg /web1
    acl has_web2 path_beg /web2
    
    use_backend web1 if has_web1
    use_backend web2 if has_web2
    
    default_backend web1

backend web1
    server web1 web1:5000 check
    
backend web2
    server web2 web2:5000 check

## haproxy/Dockerfile
FROM haproxy:2.2.33

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg


## app.py

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_geek():
    return '<h1>Hello from Flask and Docker. Hope you have a great day</h1>'

@app.route('/web1')
def hello_geek_w1():
    return '<h1>Hello from web1. Hope you have a great day</h1>'

@app.route('/web2')
def hello_geek_w2():
    return '<h1>Hello from web2. Hope you have a great day</h1>'


if __name__ == "__main__":
    app.run(host ='0.0.0.0', port = 5000, debug = True) 


## docker-compose.yml

version: "2"

services:
  web1:
    build: .
    container_name: web1
    
  web2:
    build: .
    container_name: web2    

  haproxy:
    build: ./haproxy
    container_name: haproxy
    ports:
      - "5001:5001"


## docker-compose up --build
```

* HAProxy queries nameservers (192.168.2.10, 192.168.3.10) for backend hostname resolution. The nameserver first checks if it has a cached DNS record for hostname1.example.com. If the cache has expired, it queries authoritative DNS servers to get an updated IP. If hostname1.example.com is managed by AWS Route 53, the nameserver queries AWS's authoritative DNS servers (say it gets back 10.0.1.200).

* Below some key networking concepts have been explored such as
    * Network address and Host address
    * Network and netwrok mask, Subnet and subnet mask
    * Default gateway
    * Domain Name system

* The network address is used to identify the network and is common to all the devices attached to the network. The host (or node) address is used to identify a particular device attached to the network. 

* A subnet is when you divide a network into several sub networks. Let's take the network above (192.168.xxx.xxx) and say we want to split it by office floor or something: 192.168.1.xxx for the first floor, 192.168.2.xxx for the second floor, etc. Each of these is a subnet.

* A subnet mask is like a network mask, but for a subnet. So in the example above, the subnet mask for each floor is 255.255.255.0 because now only the last number can change, the first three are the prefix of that subnet.

* A default gateway is the IP address of the router that will deal with IP's that are outside of a particular network. Now we have a specific machine on the second floor: 192.168.2.14.
    * If this machine sends a message to 192.168.2.10, by applying the subnet mask you know you're in the same network, and the message will be sent directly to that address.
    * If this machine sends a message to 192.168.1.12 (a computer on the first floor), by applying the subnet mask you know you are not in the same network, as you'd expect a 2 in third place. So you can't send it directly to the destination. What you do instead is send it to the default gateway of your network: 192.168.2.1. This gateway is also part of the whole building network. By applying the network mask of the whole building (255.255.0.0) we know the destination IP is in that network, and we can send the message on its merry way to the subnetwork on the first floor.
    * If the source machine sends a message to 173.194.34.0 (google.com), by applying the subnet mask, we know we are not in the right network, so we send the message to the default gateway: 192.168.2.1. That gateway will apply the network mask for the whole building, and will know we are still not in the right network, so it will forward the message to the default gateway for the whole network: 192.168.0.1. This default gateway will recognize the address as an internet address and sends it to the internet connection on its way to the Google address.

* Network mask : To tell what part of IP address is network address.

* DNS, or the Domain Name System, acts as the internet's "phone book," translating human-readable domain names (like "www.example.com") into IP addresses (like "192.0.2.1") that computers use to communicate

* Docker containers communicate with the host machine using various methods, including bridge networks, host networks, and a special DNS name (host.docker.internal), allowing access to services running on the host from within containers

* DNS is not supported in the default bridge network. That is where we can create a custom bridge network and leverage DNS (as shown in 6)

* Tried bashing into system, and then pinging other docker containers using their ip was able to do it successfully
    1. Get the docker network name using `docker network ls` and then inspect it using `docker network inspect <network-name>`, to get ip address of all connected containers
    2. Bash into the container using `docker exec -it <container-id> sh`
    3. Ping the other containers ip addresses using `ping <ip-address>`

* Failover management : if one backend server goes down, traffic is automatically routed to the available server. In Haproxy we do it using `option redispatch` and `backup` as shown below

```
global
    log stdout format raw daemon debug

defaults
    log global
    mode http
    option httplog          # Enable detailed HTTP logging
    timeout connect 5000ms
    timeout client 5000ms
    timeout server 5000ms

listen http_proxy
    bind *:5001
    balance roundrobin      # Load balance traffic

    option redispatch       # Send requests to a healthy server if the chosen one fails
    # Health check settings
    # Health check interval (every 2 seconds). 
    # Server is marked as "DOWN" if it fails 3 consecutive health checks
    # Server is marked as "UP" after 2 successful health checks.
    default-server inter 2s fall 3 rise 2  

    server web1 web1:5000 check
    server web2 web2:5000 check backup

```

* In the above config requests are still round-robin balanced between web1 and web2, although web2 is marked as backup. If we want 

* Errors :
    * Starting frontend GLOBAL: cannot bind UNIX socket [/run/haproxy/admin.sock] (refer 1 for soln)
    * Server web1/web1 is DOWN, reason: Layer4 connection problem, info: "Connection refused", 

### Doubts
1. What is a resolver?
2. What is /16 after ipv4 address?
3. How could I ping my docker container from my host
4. What is iptables?
5. How and from where do the nameservers fetch the updated ip?
6. What is ssl/tls?
7. What if there is no server defined in listen proxy?
8. What is 0.0.0.0 ip address?

### References
1. https://stackoverflow.com/questions/30101075/haproxy-doesnt-start-can-not-bind-unix-socket-run-haproxy-admin-sock
2. https://stackoverflow.com/questions/40729125/layer4-connection-refused-with-haproxy
3. https://www.reddit.com/r/explainlikeimfive/comments/vlyj4/explain_truly_like_im_5_what_a_default_gateway/?rdt=37836
4. https://unix.stackexchange.com/questions/561751/what-does-this-mean-16-after-an-ip-address
5. https://stackoverflow.com/questions/39216830/how-could-i-ping-my-docker-container-from-my-host
6. https://www.youtube.com/watch?v=fBRgw5dyBd4
7. https://stackoverflow.com/questions/17157721/how-to-get-a-docker-containers-ip-address-from-the-host
8. https://stackoverflow.com/questions/57112326/haproxy-get-logs-from-docker-container
9. https://www.haproxy.com/documentation/haproxy-configuration-manual/latest/#balance
10. https://stackoverflow.com/questions/39209917/difference-between-frontend-backend-and-listen-in-haproxy
11. https://www.haproxy.com/blog/failover-and-worst-case-management-with-haproxy


## Day 6

* Worked with Kubernetes for the first time ever on 28th-March-2025 (started learning Kubernetes today)

* Installed kind using Powershell, moved it into C:\kind and then added that location to path

* Docker Desktop for Windows adds its own version of kubectl to PATH. If you have installed Docker Desktop before, you may need to place your PATH entry before the one added by the Docker Desktop installer or remove the Docker Desktop's kubectl.

```
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.27.0/kind-windows-amd64
mkdir C:\kind
Move-Item .\kind-windows-amd64.exe c:\kind\kind.exe
```

* kind uses containerd as a CRI implementation to deal with Pods (and hence - containers)

* Container : Container is a packaged, self-contained unit of software

* Pod : Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. 

* A pod within a node has:
    * A local IP address.
    * One or more Linux containers. For instance, Docker is commonly used as a container runtime.
    * One or more volumes that are associated with these containers are persistent storage resources.

* Node : Machine (physical or virtual) where your containerized applications (pods) reside and run. A node is a fundamental building block of a Kubernetes cluster

* Control plane : Manages clusters and resources such as worker nodes and pods. 

* Node is made up of 3 components : kubelet, a container runtime, and the kube-proxy. 

* Control plane is made up of 5 components : kube-apiserver, kube-scheduler, kube-controller-manager, cloud-controller-manager, etcd (refer 11 for full architecture)

* Kubeconfig : YAML files that configure Kubectl, the default Kubernetes client tool. It contains 3 important things:
    * clusters: https end point to cluster
    * context: cluster + user (use `kubectl config current-context`)
    * users: to authenticate to a cluster

* We can switch b/w different Kubernetes clusters using `kubectl config use-context <different-context>`

* Deployment :  Deployment manages a set of Pods to run an application workload. It tells Kubernetes how to create or modify instances of the pods that hold a containerized application.

* Service :  Service is a method for exposing a network application that is running as one or more Pods in your cluster. It basically enables external traffic exposure to cluster

* ConfigMap: With it we store our environment variables in the cluster. 

* Volume : Container's file system lives only as long as the Container does. So when a Container terminates and restarts, filesystem changes are lost. For more consistent storage that is independent of the Container, you can use a Volume. This is especially important for stateful applications, such as key-value stores (such as Redis) and databases.

*  There are four types of service in Kubernetes.
    * ClusterIP
    * NodePort
    * LoadBalancer
    * ExternalName

* There are 5 types of Kubernetes Volumes:
    * Persistent Volumes
    * Ephemeral Volumes
    * EmptyDir Volumes
    * Kubernetes hostPath Volumes
    * Kubernetes Volumes ConfigMap

* Docker vs K8:
    * Docker is a container technology VS K8s is a management technology.
    * Docker is about automated building VS K8s is about automated managing.
    * Docker is a container runtime that packages applications into containers VS Kubernetes is a container orchestration platform that manages and scales those containers across a cluster of machines
    
* Docker Compose vs K8 : If you are networking containers within the same host go for docker compose VS If you are networking containers across multiple hosts go for kubernetes.

* Hence if we want to run all our containers on a single machine, also know as **single node cluster**, Docker and Docker Compose is sufficient. Only 

* In the Kubernetes cluster we don’t create containers, and we’ll create pods that are abstraction layer over containers. So, We only work with Pods. Generally we have one container per pod, only times that you have more than one container in a pod are time that your application needs some helper containers.

```
kind create cluster

kubectl get nodes

kubectl get pod -v6
# C:\Users\dell\.kube\config

```

* kind was giving lot of errors so downloaded kubectl and enabled kubernetes within docker itself. This enabled the context docker-desktop which is much easier to use (followed That Devops Guy)

```
kubectl config get-contexts
# Returned 2 contexts docker-desktop and kind-deployments, with current being kind-deployments

kubectl config use-context docker-desktop
# Change context to docker-desktop as kind-deplyments not working properly

kubectl get pods
# Output : No resources found in default namespace

kubectl apply -f deployment.yaml

kubectl get deploy
# Output : example-deploy

kubectl get pods
# Output : example-deploy-67b49d65bc-jkj5f   2/2     Running   0          93s

kubectl scale deployment example-deploy --replicas=1

kubectl get pods
# Output : example-deploy-67b49d65bc-jkj5f   1/1     Running   0          93s

kubectl apply -f .\services\service.yaml

kubectl get svc
# Output : example-service   LoadBalancer   10.96.24.178   localhost     80:31022/TCP   115m
# After creating service we can go to localhost to see flask app


kubectl scale deployment example-deploy --replicas=0

```
*  Kubernetes doesn't support stop/pause of current state of pod and resume when needed. However, you can still achieve it by having no working deployments which is setting number of replicas to 0 (as done above in last line).

* Files use to create the kubernetes cluster are as follows (note that we are using prebuilt docker image from docker hub aimvector/python:1.0.4)

```

# deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deploy
  labels:
    app: example-app
    test: test
  annotations:
    fluxcd.io/tag.example-app: semver:~1.0
    fluxcd.io/automated: 'true'
spec: # specs about pod
  selector:
    matchLabels:
      app: example-app
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers: # specs about container within the pod
      - name: example-app
        image: aimvector/python:1.0.4
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "256Mi"
            cpu: "500m"
      tolerations:
      - key: "cattle.io/os"
        value: "linux"
        effect: "NoSchedule"      

# services/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
  labels:
    app: example-app
spec:
  type: LoadBalancer
  selector:
    app: example-app
  ports:
    - protocol: TCP
      name: http
      port: 80
      targetPort: 5000

```
* In deployment.yaml, we have specs of pod and within that specs of the container within the pod


* Tried creating a flask container and using that within deployment.yaml, but it did not work

```
docker build --tag flask-kind:0.1.0 .

docker run -p 5000:5000 flask-kind:0.1.0

kind create cluster --name deployments --image kindest/node:v1.31.1

kubectl apply -f deployment.yml

kubectl get pods

kubectl get nodes

kubectl get deployments

```

* Make sure to separate config from code, keep flask code in src folder, kubernetes configs in kubernetes folder

* Errors faced
    * ERROR: no nodes found for cluster "kind" , on running `kind load docker-image flask-kind:0.1.0`. Reason: kind can't see or use docker images you've built or pulled in Docker For Desktop (refer 18)

    * ERROR: failed to create cluster: could not find a log line that matches "Reached target .*Multi-User System.*|detected cgroup v1", on running `kind create cluster --image=flask-kind:0.1.0`

    * Unable to connect to the server: dial tcp 127.0.0.1:52549: connectex: No connection could be made because the target machine actively refused it, on running `kubectl get pods` and `kubectl apply -f deployment.yaml` : Solution is `kubectl config use-context docker-desktop`

    * Kubernetes service external ip pending


### Doubts
1. What is use of EXPOSE in Dockerfile? Especially when we already have port mapping (not much)
2. Is port mapping done during docker build or docker run?
3. What is diff bw pod and container in Kubernetes?
4. Diff bw docker container and kubernetes container?
5. Because we’re not supposed to pack multiple processes into a single container, we need a higher-level structure that will allow us to tie and wrap containers together and manage them as a single unit. This is the reasoning behind the pods. But why are we not supposed to pack multiple process in single container?
6. Is Docker essentially Kubernetes on a single node?
7. How to connect Flask app on a kubernetes pod to an external database in another network? (use type ExternalName)
8. Are there any advantages of using "kind" instead of the integrated Kubernetes from Docker for learning purposes?
9. Can we automatically restart unhealthy container in docker-compose?
10. What is the kubectl equivalent commands to "minikube service <service name>"
11. Kubernetes: create service vs expose deployment, what is difference?

### References
1. https://www.youtube.com/watch?v=kbeqNY0v0c4
2. https://kind.sigs.k8s.io/docs/user/quick-start
3. https://stackoverflow.com/questions/68172643/finding-the-kubeconfig-file-being-used
4. https://www.quora.com/How-much-time-does-it-take-to-learn-Kubernetes-from-scratch-What-are-the-steps-to-follow-with-examples
5. https://iximiuz.com/en/posts/kubernetes-kind-load-docker-image/
6. https://dhavalgojiya.hashnode.dev/understanding-dockers-expose-keyword-4-port-mapping-scenarios-explained
7. https://www.reddit.com/r/kubernetes/comments/196tgmv/basic_question_about_number_of_container_per_pod/
8. https://www.baeldung.com/ops/kubernetes-pod-vs-container
9. https://www.reddit.com/r/selfhosted/comments/13tcdps/docker_compose_or_kubernetes_for_single_node/
10. https://www.armosec.io/glossary/kubernetes-control-plane/
11. https://kubernetes.io/docs/concepts/architecture/
12. https://spacelift.io/blog/kubeconfig
13. https://stackoverflow.com/questions/50490808/unable-to-connect-to-the-server-dial-tcp-18080-connectex-no-connection-c
14. https://www.youtube.com/watch?v=d1ZMnV4yM1U (That Devops Guy)
15. https://discuss.kubernetes.io/t/connecting-to-an-external-mysql-database/8201
16. https://medium.com/@ManagedKube/kubernetes-access-external-services-e4fd643e5097
17. https://medium.com/@tech_with_mike/how-to-deploy-a-django-app-over-a-kubernetes-cluster-with-video-bc5c807d80e2
18. https://www.reddit.com/r/kubernetes/comments/dc0qk4/are_there_any_advantages_of_using_kind_instead_of/
19. https://stackoverflow.com/questions/38511459/kubernetes-node-vs-hosts-vs-cluster-terminology
20. https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/
21. https://stackoverflow.com/questions/61628052/what-is-the-kubectl-equivalent-commands-to-minikube-service-service-name
22. https://github.com/smriti111/django-postgresql-kubernetes
23. https://stackoverflow.com/questions/54821044/how-to-stop-pause-a-pod-in-kubernetes


## Day 7

* The ImagePullBackOff error is a common error message in Kubernetes that occurs when a container running in a pod fails to pull the required image from a container registry.

* Module vs Package (Python) : The module is a single Python file that can be imported into another module. A package is a collection of Python modules: while a module is a single Python file, a package is a directory of Python modules containing an additional __init__.py file, to distinguish a package from a directory that just happens to contain a bunch of Python scripts. Packages can be nested to any depth, provided that the corresponding directories contain their own __init__.py file.


* Created simple Flask application that I want to containerize and then deploy to a Kubernetes pod
```
## app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_geek():
    return '<h1>Hello from Flask and Docker and Kubernetes. Hope you have a very great day</h1>'


if __name__ == "__main__":
    app.run(host ='0.0.0.0', port = 4000, debug = True)


## Dockerfile
FROM python:3.9-alpine3.20

WORKDIR /flask-app

COPY app.py app.py

RUN pip3 install flask

CMD ["python3","app.py"]

## In terminal run : docker build . -t localhost:4000/my-flask-image

```

* Below are the steps to use a local docker image in the kubernetes deployment.yaml. (trick is to create local registry and push image to that)

```
# create local docker registry
docker run -d -p 4000:5000 --restart=always --name registry registry:2

# run from folder containing the files app.py(flask app) and Dockerfile
docker build . -t localhost:4000/my-flask-image

# push to local registry
docker push localhost:4000/my-flask-image

### Use this image (localhost:4000/my-flask-image) in the deployment.yaml for image key, with imagePullPolicy as IfNotPresent. Also modify port on services.yml

# deploy flask on kubernetes
kubectl apply -f deployment.yaml
kubectl apply -f services\service.yaml
kubectl get deploy
kubectl scale deployment example-deploy --replicas=2 

## visit localhost and you should see the flask app
```


* Once this was done goal was to deploy a flask app on Kubernetes and connect that a Postgres database running outside the Kubernetes cluster

* To use poetry only for package management, follow the steps below. First create the following pyproject.toml file. The key thing is to set `package-mode = false`, since we are using it only for dependency mgmt. In this mode, metadata such as name and version are optional. 

```
## pyproject.toml
[tool.poetry]
package-mode = false

[tool.poetry.dependencies]
python = ">=3.9"

[tool.poetry.group.dev.dependencies]

[tool.poetry.group.test.dependencies]

[tool.poetry.extras]

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"

```

* Poetry will detect and respect an existing virtual environment that has been externally activated. This is a powerful mechanism that is intended to be an alternative to Poetry’s built-in, simplified environment management.To take advantage of this, simply activate a virtual environment using your preferred method or tooling, before running any Poetry commands that expect to manipulate an environment.


* Set up virtual environment for flask app as follows
```
virtualenv venv

# so that poetry uses activated virtual environment
venv\Scripts\activate

poetry install # creates poetry.lock

poetry add flask

poetry add sqlalchemy

poetry add flask-sqlalchemy psycopg2

pip list # check if packages installed
```

* Setting up the postgres database which the Flask application will connect to

```
docker run --name postgres-flask -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 postgres

docker exec -it postgres-flask sh

psql -U postgres

create database parking;
```

* Created the following flask app

```

from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Database configuration (update these values as needed)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://postgres:postgres@localhost:5432/parking'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# Define a Customer model
class Customer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)

# Create the database and table
with app.app_context():
    db.create_all()
    
    # Insert some records if table is empty
    if Customer.query.count() == 0:
        users = [
            Customer(name='Alice', email='alice@example.com'),
            Customer(name='Bob', email='bob@example.com'),
            Customer(name='Charlie', email='charlie@example.com')
        ]
        db.session.add_all(users)
        db.session.commit()

@app.route("/", methods=['GET'])
def get_home_page():
    return "<h2>Hello to flask</h2>"

@app.route('/customers', methods=['GET'])
def get_users():
    users = Customer.query.all()
    return jsonify([{ 'id': user.id, 'name': user.name, 'email': user.email } for user in users])

if __name__ == '__main__':
    app.run(debug=True, port=4000, host ='0.0.0.0')

### python app.py (within the virtual environment)
```

* Also bashed into the Postgres docker container to see if table had been created. Name of table is same as name of model class

```
docker exec -it postgres-flask sh

psql -U postgres

# List all databases
\l 

# Connect to database with name parking
\c parking

# List tables from current schema
\dt

#
select * from customer; # 3 records were shown

# Exit psql
\q

```

* Sqlalchemy create_all does not update tables if they are already in the database. If you change a model’s columns, use a migration library like Alembic with Flask-Alembic or Flask-Migrate to generate migrations that update the database schema.

* Flask SQLAlchemy is a popular ORM tool tailored for Flask applications.

* 192.168.1.1 is the address of your router on your network, through which all your internet traffic goes through. Note that router has 2 IP, a public IP to connect to internet, a private IP to communicate with all devices on the same network. 192.168.1.1 is the private IP

* Network Address Translation (NAT) enables devices within private IP networks to use the internet and cloud by translating (internal) private IP addresses to (external) public IP addresses and vice versa . It allows multiple devices to use the same public IP address and access the Internet and is typically done by a router.

* Two devices on the same WiFi will be seen as having the same IP address externally (to check this connect your phone and laptop to same wifi and check Whats my IP on Google : 103.120.51.59). One of the ways computers on the same network get distinguished in communication with the same public server is by assigning them by the router different port numbers in the communication. Their public IP address is the same, but the port number part is not.

* cURL is a CLI tool that allows you to request and transfer data over a URL using different protocols.

* We can access host from within docker container by using host.docker.internal. Below I bash into an alpine container and from within the container curl the flask app running on port 4000 of my host system, which works successfully

```

docker run -it --rm alpine /bin/ash
apk --no-cache add curl #alpine image doesn't come with curl.
curl http://host.docker.internal:4000 # OUTPUT: <h2>Hello to flask</h2>

```

* Lot of errors installing psycopg2 within Docker container. Finally worked with ref 15 (had to install libpq-dev gcc)

* When a Docker container uses the "host" network mode (--network=host), it doesn't get its own IP address because it shares the host machine's network namespace, meaning it essentially runs directly on the host's network stack

```
### src/app.py
from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# Database configuration (update these values as needed)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://postgres:postgres@host.docker.internal:5432/parking'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

# Define a User model
class Customer(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)

# Create the database and table
with app.app_context():
    db.create_all()
    
    # Insert some records if table is empty
    if Customer.query.count() == 0:
        users = [
            Customer(name='Alice', email='alice@example.com'),
            Customer(name='Bob', email='bob@example.com'),
            Customer(name='Charlie', email='charlie@example.com')
        ]
        db.session.add_all(users)
        db.session.commit()

@app.route("/", methods=['GET'])
def get_home_page():
    return "<h2>Hello to flask</h2>"

@app.route('/customers', methods=['GET'])
def get_users():
    users = Customer.query.all()
    return jsonify([{ 'id': user.id, 'name': user.name, 'email': user.email } for user in users])

if __name__ == '__main__':
    app.run(debug=True, port=4000, host ='0.0.0.0')

### src/requirements.txt
Flask==3.1.0
Flask-JWT-Extended==4.7.1
Flask-SQLAlchemy==3.1.1
SQLAlchemy==2.0.40
psycopg2==2.9.10

### src/Dockerfile
FROM python:3.11.5-slim-bookworm

WORKDIR /flask-app

# libpq-dev and gcc required for psycopg2 to work
RUN apt-get update && \
apt-get -y install libpq-dev gcc

COPY app.py app.py 
COPY requirements.txt requirements.txt

RUN pip install -r requirements.txt

CMD ["python", "app.py"]

### docker build --no-cache --tag flask-postgres:1.0.0 .
### docker run -p 4000:4000 flask-postgres:1.0.0

```

* To access the postgres database running on host from the Flask app running within the container, I had to put database connection string as `postgresql://postgres:postgres@host.docker.internal:5432/parking`

* Note : Without the host ='0.0.0.0' in app.run(), the Flask app within the Docker container was not accessible, although everything else was the same

* Tried a few other things 
```
docker run --network="host" -p 4000:4000 flask-postgres:1.0.0 # did not work
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 0179f405ae23
docker run --add-host host.docker.internal:host-gateway -p 4000:4000 flask-postgres:1.0.0 # did not work
```
* Errors faced
    * The current project's supported Python range (>3.8) is not compatible with some of the required packages Python requirement: flask requires Python >=3.9, so it will not be satisfied for Python >3.8,<3.9.Because no versions of flask match >3.1.0,<4.0.0 and flask (3.1.0) requires Python >=3.9, flask is forbidden.
    * This error originates from the build backend, and is likely not a problem with poetry but one of the following issues with psycopg2 (2.9.10)
    *  pg_config executable not found.pg_config is required to build psycopg2 from source.  Please add the directory containing pg_config to the $PATH or specify the full executable path with the option:
    *  (psycopg2.OperationalError) connection to server at "localhost" (127.0.0.1), port 5432 failed: Connection refused. Is the server running on that host and accepting TCP/IP connections?connection to server at "localhost" (::1), port 5432 failed: Cannot assign requested address
    * Port 4000 is in use by another program. Either identify and stop that program, or start the server with a different port : Reason : Docker registry was also running on same port


```
# Did not work (tried using poetry instead of pip within docker)
FROM python:3.11.5-slim-bookworm

ENV POETRY_VIRTUALENVS_CREATE=false

RUN pip install poetry
RUN apt install libpq-dev gcc

WORKDIR /flask-app

COPY app.py app.py 
COPY pyproject.toml pyproject.toml

RUN poetry install

CMD ["python", "app.py"]
```


### Doubts
1. Why do we maintain database migrations for a Django/Flask app? Is it because sqlalchemy cannot update existing tables?
2. How do we connect a flask app inside Docker to a postgres database outside Docker? How do we connect a flask app inside Kubernetes to a postgres database outside Kubernetes? 
3. What should be the database connection string when we connect from a flask app inside Docker or Kubernetes to a database outside Docker/Kubernetes?
4. What's the difference between 127.0.0.1 and 0.0.0.0?
5. What is 192.168.1.1 and how does it work?
6. Do two computers connected on the same Wi-Fi have the same public IP address?
7. WHat exactly is th add-host flag in DOcker?
8. How to copy folders to docker image from Dockerfile?

### References
1. https://stackoverflow.com/questions/42078080/add-nginx-conf-to-kubernetes-cluster
2. https://stackoverflow.com/questions/57167104/how-to-use-local-docker-image-in-kubernetes-via-kubectl
3. https://www.docker.com/blog/how-to-use-your-own-registry-2/
4. https://stackoverflow.com/questions/7948494/whats-the-difference-between-a-module-and-package-in-python
5. https://stackoverflow.com/questions/22256124/cannot-create-a-database-table-named-user-in-postgresql
6. https://www.youtube.com/watch?v=Y0eAWkddEuM (Flask JWT and migrations)
7. https://superuser.com/questions/801105/do-two-computers-connected-on-the-same-wi-fi-have-the-same-ip-address
8. https://www.reddit.com/r/AskReddit/comments/ov38cj/what_is_19216811_and_how_does_it_work/
9. https://superuser.com/questions/949428/whats-the-difference-between-127-0-0-1-and-0-0-0-0
10. https://stackoverflow.com/questions/35689628/starting-a-shell-in-the-docker-alpine-container
11. https://stackoverflow.com/questions/64300599/why-do-i-get-curl-not-found-inside-my-nodealpine-docker-container
12. https://medium.com/@TimvanBaarsen/how-to-connect-to-the-docker-host-from-inside-a-docker-container-112b4c71bc66
13. https://stackoverflow.com/questions/77727508/problem-installing-psycopg2-for-python-venv-through-poetry
14. https://stackoverflow.com/questions/39985480/unable-to-use-sudo-commands-within-docker-bash-sudo-command-not-found-is-di
15. https://wbarillon.medium.com/docker-python-image-with-psycopg2-installed-c10afa228016
16. https://stackoverflow.com/questions/7023052/configure-flask-dev-server-to-be-visible-across-the-network
17. https://stackoverflow.com/questions/39901311/docker-ubuntu-bash-ping-command-not-found


## Day 8 and 12

* Resources : Key Kubernetes resouces include:
    * Deployment (kubectl get deploy)
    * Pod (kubectl get pods)
    * Service (kubectl get svc)
    * Namespace
    * Ingress
    * StatefulSet
    * ConfigMap

* Cluster: A Kubernetes cluster consists of a control plane plus a set of worker machines, called nodes, that run containerized applications. Every cluster needs at least one worker node in order to run Pods.

* Control Plane: Manages clusters and resources such as worker nodes and pods. The control plane receives information such as cluster activity, internal and external requests, and more. 

* kubelet: Part of node. It ensures that Pods are running, including their containers (node consists of kubelet, container runtime and kube proxy)

* kube-proxy : Maintains network rules on nodes to implement Services

* Container runtime : Software that actually runs the containers i.e. the actual working code like flask or postgres (Kubernetes depreciated Docker as a container runtime after v1.20, using containerd instead) 

* Containerd is the low-level container runtime that Docker is built upon

* Namespace: Help divide or segment resources within a cluster
    
* Context:

* Container : A container image is a ready-to-run software package containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings

* ConfigMap: Used to store environment variables, command line arguments and other non confidential data (like database name). To store confidential data, we use secrets

* ReplicaSet: ReplicaSets allow you to tell Kubernetes to create multiple copies of the same Pod. This helps ensure the availability of applications because if one Pod fails for some reason (such as its host node crashing or being drained), other copies of the Pod remain available. Of course, if you ran just a single Pod instance, Kubernetes would try to restart the Pod in the event it failed. The major advantage of Kubernetes ReplicaSets is that you always have multiple Pod replicas running, so there is no downtime in Pod availability while you wait for Kubernetes to restart a failed Pod.

* A Deployment provides declarative updates for Pods and ReplicaSets.For example when we set .spec.replicas as 3 in deployment, it creates a ReplicaSet that creates 3 replicated pods (refer 6)

* Label: The set of pods that a service targets is defined with a label selector. Hence it is important to give  a pod a label, and in the service spec.selector, use the same label

* While namespaces help group resources, labels are for more than just grouping. For a service to target a pod, it needs to have a label


```
docker run -d -p 4000:5000 --restart=always --name registry registry:2

docker run --name postgres-flask-v2  -e POSTGRES_USER=pgusername -e POSTGRES_PASSWORD=pgpassword -d -p 5432:5432 postgres

docker exec -it 587b229c42c7 sh

psql -U pgusername

create database demo_db;

\c demo_db
\dt
select * from users;

docker build . -t localhost:4000/my-flask-image-v2
docker push localhost:4000/my-flask-image-v2

```
* Most tutorial push to a public Docker registry like Docker Hub, but I pushed it to my local Docker registry

* Manifest file : A YAML or JSON file that describes the desired state of a Kubernetes object (eg. deployment.yaml)

* Manifest file (deployment.yaml)
    * apiVersion : Which Kubernetes API we are using. While there are many apis available (refer 2), the most commonly used ones are:
        * v1 : Typically used with kind service
        * app/v1 : Typically used with kind deployment, stateful sets,
    * kind : Defines the type of resource being created or modified such as Deployment, Service, ConfigMap, Ingress
    * metadata: Contains essential information about the resource, such as its name, namespace, labels, and annotations
        * name: Specifies the name of the resource, allowing it to be uniquely identified within its namespace
        * labels: Enables categorization and grouping of resources based on key-value pairs.
        * namespace: Which namespace resource belongs to
    * spec: Describes the desired state of the resource. It outlines the configuration details and behavior of the resource
        * replicas : Number of copies of the pod deployment will create
        * selector.matchLabels : tells what pods the deployment will apply to.
        * template : Is the PodTemplate i.e. details about the pod



* The first metadata describes the deployment, 
```
### deployment.yaml

apiVersion: "app/v1"
kind: "Deployment"
metadata: 
    name: "flask"
    namespace: "default"
    labels:
        app: "flask"
spec:
    replicas: 1
    selector:
        matchLabels: 
            app: "flask"
    template:
        metadata:
            labels:
                app: "flask"
        spec:
            containers:
                - name: "flask"
                  image: "localhost:4000/my-flask-image-v2"
                  imagePullPolicy: IfNotPresent
                  ports:
                  - containerPort: 3939 

---

apiVersion: "v1"
kind: Service
metadata:
    name: "flask-service"
    namespace: "default"
    labels:
        app: "flask"
spec:
    ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3939
    selector: 
      app:"flask"

    type: "LoadBalancer"


```

* For containers, it is an array because we can theoretically have more than 1 container within a pod

* Only Job, Deployment, Replica Set, and Daemon Set support matchLabels. For other resource types like Service, we target a label using just selector with a map. But they both mean the same thing (refer 8)

* When you create a deployment, the Kubelet reads the PodSpec (a YAML that describes a Pod) and then instructs the container runtime using the CRI (Container Runtime Interface) to spin up containers to satisfy that spec. The container runtime pulls the image from the specified container registry and runs it. If you don’t specify a container registry hostname, Kubernetes will assume the image is in the Default Docker Registry. 

* ImagePullPolicy : Lets you specify how you want the Kubelet to pull an image if there’s any change (restart, update, etc.) to a Pod. Options include:
    * IfNotPresent : Kubernetes will only pull the image when it doesn’t already exist on the node
    * Always : Kubernetes will always pull the latest version of the image from the container registry
    * Never : there will be no attempts to pull the image

* containerPort : containerPort as part of the pod definition is only informational purposes. If you actually want to expose a port as a service within the cluster or node then you have to create a service.

*  Pods are designed as relatively ephemeral (last for short duration), disposable entities, so they can be and often are, changed or replaced

* Since pods keep on changing, their IP address also keeps on changing. Hence we use services to reach the pods (hence important every pod have labels)

* Service : Helps clients reach pods that can fulfill their request, without client having to worry about pods dynamic ip address (the service can be reached at the same place at any point in time, and hence is a stable destination for client to access pod)


* Kubernetes supports five main service types:
    * ClusterIP : Exposes the service on a cluster-internal IP. Used for Pod-to-Pod communication within the same cluster
    * NodePort : provides a way to expose your application to external clients. We can choose a port but it must be in the range of 30000 to 32767
    * LoadBalancer
    * ExternalName and Headless


* On doing `kubectl describe service <service-name>`, the LoadBalancer Ingress gives the external ip of the service (assuming the service is of the right type)

* The manifest file for Postgres is as follows
  * ConfigMap : They require a top-level data field that defines the key-value config pairs to store

```
apiVersion: "v1"
kind: "ConfigMap"
metadata:
  name: "postgres-config"
  namespace: "default"
  labels:
    app: "postgres"
data:
  POSTGRES_DB: "demo_db"
  POSTGRES_USER: "pgusername"
  POSTGRES_PASSWORD: "pgpassword"

---

apiVersion: "apps/v1"
kind: "Deployment"
metadata:
  name: "postgres"
  namespace: "default"
  labels:
    app: "postgres"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: "postgres"
  template:
    metadata:
      labels:
        app: "postgres"
    spec:
      containers:
      - name: "postgres"
        image: "postgres"
        env:
        - name: "POSTGRES_DB"
          valueFrom:
            configMapKeyRef:
              key: "POSTGRES_DB"
              name: "postgres-config"
        - name: "POSTGRES_USER"
          valueFrom:
            configMapKeyRef:
              key: "POSTGRES_USER"
              name: "postgres-config"
        - name: "POSTGRES_PASSWORD"
          valueFrom:
            configMapKeyRef:
              key: "POSTGRES_PASSWORD"
              name: "postgres-config"
      ports:
        - containerPort: 5432
          name: "postgres"

      volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/db-data
    
    volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pv-claim
  

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postges-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---

apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
    - port: 5432 # since targetPort not specified, targetPort = port = 5432 
  selector:
    app: postgres
  


```

* ConfigMap help **set configuration data separately from application code** (similar to why we have a separate configuration.py in Flask). Configuration data includes thing like database username, password, server etc whereas the application code includes operations like reaing from/writing to database/

* Kubernetes stores ConfigMaps in its etcd datastore. You shouldn’t directly edit etcd data; instead, use the Kubernetes API server and Kubectl to create and manage your ConfigMaps (to back up ConfigMap values, follow the guidance to set up backups for etcd)

* To use to values (or rather key-value pairs) defined in the ConfigMap, we use either valueFrom or envFrom with configMaKeyRef 

* envFrom vs keyFrom : envFrom imports all key-value pairs from a ConfigMap or Secret, setting them as environment variables, while valueFrom allows you to select specific keys from a ConfigMap or Secret and use their values for individual environment variables

* Volumes: A volume is essentially a directory with data, accessible to multiple containers in a pod, where data is stored for the life of the pod. Provide a way for containers in a pod to access and share data via the filesystem. We need volumes for 2 reasons:
  * Data Persistence : When container crashes/stops, all of the files that were created or modified during the lifetime of the container are lost. 
  * Shared Storage : To share files across containers in a pod

* Mounting : "Mount" is the Posix term for taking some sort of storage and attaching it to a filesystem at a given point. If you hook a new harddrive up to a computer running Linux, all you'll see at first is a new device under /dev. You have to "mount" it for it to be accessible at /some/directory where you can now browse the files etc. This is something Windows, Mac, and even desktop Linux users don't ever really have to deal with as it's done automatically - insert a new USB thumb drive into a laptop and chances are it'll get automatically mounted somewhere and you just start using it. When you do the "safely remove USB" thing in Windows, essentially you're unmounting the drive. Once it's detached from the filesystem programs can't be reading or writing to the files on the drive, which is why it's now safe to remove.

* Volume Mount : Volume mount refers to the process of making a volume (a directory or file on the host machine) accessible to a container, allowing data to be shared and persisted even after the container is stopped or restarted

* MountPath : Where inside the container the volume is

* Note : The name of the volumeMount (spec.container.volumeMounts) should match the name of the volume (spec.volumes) for the mount to be successful (refer example below). Also volume and volumeMounts go hand in hand. You can not create a volume without mounting it or mount a volume that has not been created. 

```
spec:
  volumes:
    name: xyz
  containers:
  - name: my-app
    image: nginx
    volumeMounts:
    - name: xyz # match volume name
      mountPath: /app/config


```

* Ephemeral volume types have a lifetime linked to a specific Pod, but persistent volumes exist beyond the lifetime of any individual pod

* Persistent Volume (PV) is a piece of storage in the cluster. It is a resource in the cluster just like a node is a cluster resource. They have a lifecyle independent of any pods that use them

* PersistentVolumeClaim (PVC) is a request for storage by a user (unlike persistent volume which is provisioned by admin). PVCs consume PV resources, just like how pods consume node resources. Claims can request specific size and access modes 

* StorageClass : The kind (class) of storage. Helps cluster administrators to offer users a variety of PersistentVolumes without exposing to users the details of how those volumes are implemented.  

* Persistent Volume is to Persistent Volume Claim what Node is to Pod

* Types of persistent volumes include:
  * hostPath :  HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
  * local : local storage devices mounted on nodes
  * nfs - Network File System (NFS) storage
  * glusterfs - GlusterFS storage. (not available starting v1.26)

* Types of access modes in PVC include:
  * ReadWriteOnce (RWO) : This access mode lets a single node mount the volume as read-write at a time.
  * ReadOnlyMany (ROX) : This access mode lets multiple nodes mount the volume as read-only simultaneously.
  * ReadWriteMany (RWX) : This access mode lets multiple nodes mount the volume as read-write simultaneously.
  * ReadWriteOncePod (RWOP) : This access mode allows the volume to be mounted as read-write by a single pod. This is a more restrictive version of RWO, ensuring that only one pod can write to the volume at any time.

* Some example use cases : RWO is useful for databases, ROX for creating database read replica so as to offload read queries from the primary database instance, RWX is useful for applications requiring a shared writable storage environment

* The lifecycle of PVC includes
  * Provisioning
  * Binding
  * Using
  * Reclaim

*  If a Kubernetes service definition doesn't specify a targetPort, the service will default to using the value of the port field as the target port for forwarding traffic to the pods. 

*

* Istio : Ingress controller

```
kubectl config get-contexts

kubectl get pods

kubectl apply -f deployment.yaml
kubectl get pods

# get the logs
kubectl logs flask-55d96d6986-qqv48

# get all pods with the label of app as flask
kubectl get pods -l app=flask
kubectl get pods -l app=postgres

kubectl get pod

kubectl describe 

```

* To solve the external ip pending issue for the flask service, these are the things I tried
    1. Added containerPort under container specs. Did not work
    2. Deleted another service which was having the external ip as localhost : `kubectl delete service example-service`, then retried deploying. Did not work
    3. Changed type for `flask-service` from LoadBalancer to NodePort and back to LoadBalancer. After doing this, somehow the external ip was assigned as localhost

* "CrashLoopBackOff" status : Indicates a pod is stuck in a restart loop because a container within it fails to start successfully and is repeatedly restarted.

* cURL is mainly used for making HTTP request. It can make an HTTP request to an HTTP server and it can get an HTTP response from an HTTP server. **cURL can't communicate with a database**. It can communicate with an HTTP server that runs a server side program (which could be written in PHP) which gets data from a database, formats it in an HTTP response and sends that.

* Package index files : Contain information about the packages a package repository provides. APT stores them in /var/lib/apt/lists

* APT (Advanced package tool) : A command-line utility used for managing software packages on Debian-based systems like Ubuntu and Debian

* Reason for using `apt-get update` : To refresh the package index files on a Linux system, so that system has the latest information about available software packages and their versions. Run this before using `apt-get install`. Not doing so might throw error - `Unable to locate package` (faced this when trying to install ping)

* I wanted to bash into a Kubernetes pod and then try pinging the other pods and services, so this is how I did it (based on second answer of ref 26 and 29)

```
# to get pods internal ip
kubectl get pods -o wide

kubectl get pods

kubectl get service
# OUTPUT : 
# flask-service   LoadBalancer   10.109.19.11    localhost     80:32708/TCP   20h
# postgres        ClusterIP      10.104.70.127   <none>        5432/TCP       20h

## kubectl exec -it flask-574c865cc-ljxm4 -n default sh
kubectl exec -it <pod-name> -- sh

# Install curl as it is not present by default
apt-get update
apt-get install curl

curl flask-service # OUTPUT : Hello World! I am from docker!
exit

## Bash into postgres pod
kubectl exec -it postgres-55944994d8-5qk8x -- sh

# Get linux os used
cat /etc/*-release

# Install curl and ping
apt-get update
apt-get install curl
apt-get install iputils-ping

# Tried pinging postgres service but failed
ping postgres
# OUTPUT 
# PING postgres.default.svc.cluster.local (10.104.70.127) 56(84) bytes of data
# 15 packets transmitted, 0 received, 100% packet loss, time 14589ms

# Ping other pods within cluster
# To get cluster ip address, in another terminal type kubectl get pods -o wide
ping 10.1.0.35 # ping was succeeful

```

* URL of service is in the below format: `<service-name>.<namespace>.svc.cluster.local:<service-port>`. In the above case postgres.default.svc.cluster.local. But note that if both services are in same namespace then just use service name, rest can be ignored. Note that at the end of the day, this URL is just an IP address

* As mentioned in previous point, Kubernetes creates DNS records for Services and Pods. You can contact Services with consistent DNS names instead of IP addresses.(refer 28)

* In the SQLAlchemy string `postgresql://pgusername:pgpassword@postgres:5432/demo_db`, the postgres refers to the postgres service in deployment.yaml. Since we are in the same namespace, we can access the postgres service by just typing postgres (instead of postgres.default.svc.cluster.local)


* Errors:
    * sqlalchemy.exc.NoSuchModuleError: Can't load plugin: sqlalchemy.dialects:postgres. Reason: The URI should start with postgresql:// instead of postgres://. SQLAlchemy used to accept both, but has removed support for the postgres name.

    * sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) could not translate host name "postgres" to address: No such host is known. Reason : In `SQLALCHEMY_DATABASE_URI = 'postgresql://pgusername:pgpassword@postgres:5432/demo_db'`, sqlalchemy could not figure out what postgres:5432 is hence changed to localhost:5432

    * Kubernetes service external ip pending
    * kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
    * Unable to locate package curl : Solution : First run `apt-get update`, then `apt get install curl`
    * Package 'ping' has no installation candidate : Solution : `apt-get update` followed by `apt-get install iputils-ping`

### Doubts
1. What is a context in Kubernetes?
2. What is the difference between Kubernetes cluster and namespaces?
3. Why do we need a selector within kind:Deployment, since deployment will already know which pod to target/deploy?
4. What is the difference bw pod and container?(refer Veeramalla)
5. How to use kubectl create secret?
6. WHat is the advantage of having the deployments in multiple file vs everything in 1 file?
7. Does the ip address of pod keep on changing kubernetes? How about node ip address?
8. How to know which cluster a service belongs to?
9. What is difference bw NodePort, LoadBalancer and Ingress?
10. I tried bashing into a Kubernetes pod and then trying to ping a service, but it did not work. Why?
11. How to make 2 Kubernetes services talk to each other? For example, I have a Flask app running on 1 node, how do I connect to the Postgres service using sqlalchemy?
12. What is the difference bw envFrom and valueFrom in the context of configmap in Kubernetes? When do we use each of them? (refer 32)
13. What happens to volume if pod crashes?
14. Where is Kubernetes storage location of a Persistent Volume? Where is it when we run Kubernetes on Docker? How do we the contents stored within a peristent volume?
15. What is the difference bw deployment with persistent volume claim and a Statefulset?
16. Should name of volumeMount match volume name?

### References
1. https://stackoverflow.com/questions/42078080/add-nginx-conf-to-kubernetes-cluster
2. https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html
3. https://kubernetes.io/docs/concepts/architecture/
4. https://www.linkedin.com/pulse/understanding-kubernetes-yaml-files-exploring-kind-spec-anantharamu
5. https://www.groundcover.com/blog/kubernetes-replicaset
6. https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
7. https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
8. https://medium.com/@zwhitchcox/matchlabels-labels-and-selectors-explained-in-detail-for-beginners-d421bdd05362
9. https://spacelift.io/blog/kubernetes-imagepullpolicy
10. https://stackoverflow.com/questions/55741170/container-port-pods-vs-container-port-service
11. https://spacelift.io/blog/kubernetes-deployment-yaml
12. https://medium.com/@mudasiryounas/kubernetes-docker-flask-postgres-sqlalchemy-gunicorn-deploy-your-flask-application-on-57431c8cbd9f
13. https://github.com/mudasiryounas/kubernetes-docker-flask-postgres-sqlalchemy-gunicorn-tutorial/tree/master
14. https://fahadahammed.medium.com/comparing-kubernetes-manifest-application-methods-single-file-vs-multiple-files-1e460fb0f393
15. https://stackoverflow.com/questions/64227258/all-in-one-file-kubernetes-manifest-and-one-file-per-resource-kubernetes-file
16. https://stackoverflow.com/questions/52362514/when-will-the-kubernetes-pod-ip-change
17. https://kodekloud.com/blog/clusterip-nodeport-loadbalancer/
18. https://stackoverflow.com/questions/44021319/what-is-the-best-way-to-transfer-data-from-database-using-curl
19. https://medium.com/@danielepolencic/learn-why-you-cant-ping-a-kubernetes-service-dec88b55e1a3
20. https://askubuntu.com/questions/216287/unable-to-install-files-with-apt-get-unable-to-locate-package
21. https://askubuntu.com/questions/550393/what-is-a-package-index-file
22. https://askubuntu.com/questions/216287/unable-to-install-files-with-apt-get-unable-to-locate-package
23. https://superuser.com/questions/718916/problems-installing-ping-in-docker
24. https://stackoverflow.com/questions/30746888/how-to-know-a-pods-own-ip-address-from-inside-a-container-in-the-pod
25. https://stackoverflow.com/questions/26988262/best-way-to-find-the-os-name-and-version-on-a-unix-linux-platform
26. https://stackoverflow.com/questions/59558303/how-to-find-the-url-of-a-service-in-kubernetes
27. https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/
28. https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
29. https://stackoverflow.com/questions/45720084/how-to-make-two-kubernetes-services-talk-to-each-other
30. https://www.digitalocean.com/community/tutorials/how-to-inspect-kubernetes-networking
31. https://spacelift.io/blog/kubernetes-configmap
32. https://stackoverflow.com/questions/66352023/when-should-i-use-envfrom-for-configmaps 
33. https://humanitec.com/blog/handling-environment-variables-with-kubernetes
34. https://kubernetes.io/docs/concepts/storage/persistent-volumes/
35. https://www.reddit.com/r/kubernetes/comments/ilwvlu/volumes_and_volume_mounts_still_confuse_me/
36. https://www.baeldung.com/ops/kubernetes-access-modes-persistent-volumes
37. https://www.kubermatic.com/blog/keeping-the-state-of-apps-1-introduction-to-volume-and-volumemounts/

## Day 9


* The ingress-nginx version that we download should match the Kubernetes version. Kubernetes version can be obtained using `kubectl version --short`, and get number corresponding to server version

* From ref 3, we know Ingress-nginx v1.9.5 is compatible with Kubernetes v1.25.4

* When you delete a Kubernetes namespace, all resources within that namespace are also deleted. 

* Built a simple nginx ingress using ref 4

```
kubectl version --short
# OUTPUT
# Client Version: v1.25.4
# Kustomize Version: v4.5.7
# Server Version: v1.25.4


kubectl create namespace ingress-nginx

kubectl get namespaces

# Deploy ingress nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml


kubectl get all -n ingress-nginx

kubectl get pods -n ingress-nginx
# Must show 1 running pod : ingress-nginx-controller-xxxx

# Use curl (or alternatively go from browser)
curl https://localhost/ 

kubectl -n ingress-nginx get svc

## Deleted many other services in between
kubectl get namespaces

kubectl apply -f service-a.yaml
kubectl apply -f service-b.yaml

kubectl port-forward svc/service-a 80

kubectl apply -f routing-by-path.yaml

kubectl get svc -n ingress-nginx
# ingress-nginx-controller LoadBalancer   10.98.104.70     localhost     80:30363/TCP,443:32604/TCP   175m
# ingress-nginx-controller-admission   ClusterIP      10.108.122.244   <none>   443/TCP   175m


# Went to following urls in browser
# 127.0.0.1 => "/" on service-a
# http://127.0.0.1/path-a.html => "/path-a.html" on service-a
# http://127.0.0.1/path-b.html => "/path-b.html" on service-a
# http://127.0.0.1/path-b => "/" on service-b
# http://127.0.0.1/path-b/path-b.html => "/path-b.html" on service-b

```

* Initially routing-by-path was not working as expected. Then removed the host (spec.rules.host) and post that it started working as expected (got idea from ref 7)

```
#### routing-by-path.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: service-a
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-a
            port:
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: service-b
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /path-b(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: service-b
            port:
              number: 80
---

#### service-a.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-a
data:
  path-a.html: |
    "/path-a.html" on service-a
  path-b.html: |
    "/path-b.html" on service-a
  index.html: |
    "/" on service-a  
  404.html: |
    service-a 404 page
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-a-nginx.conf
data:
  nginx.conf: |
    user  nginx;
    worker_processes  1;
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;
    events {
        worker_connections  1024;
    }

    http {
        sendfile        on;
        server {
          listen       80;
          server_name  localhost;

          location / {
              root   /usr/share/nginx/html;
              index  index.html index.htm;
          }

          error_page 404 /404.html;
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-a
  labels:
    app: service-a
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-a
  template:
    metadata:
      labels:
        app: service-a
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: "/usr/share/nginx/html/"
        - name: config
          mountPath: "/etc/nginx/"
      volumes:
      - name: html
        configMap:
          name: service-a
      - name: config
        configMap:
          name: service-a-nginx.conf
---
apiVersion: v1
kind: Service
metadata:
  name: service-a
spec:
  selector:
    app: service-a
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80


#### service-b.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-b
data:
  path-a.html: |
    "/path-a.html" on service-b
  path-b.html: |
    "/path-b.html" on service-b
  index.html: |
    "/" on service-b  
  404.html: |
    service-b 404 page
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: service-b-nginx.conf
data:
  nginx.conf: |
    user  nginx;
    worker_processes  1;
    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;
    events {
        worker_connections  1024;
    }

    http {
        sendfile        on;
        server {
          listen       80;
          server_name  localhost;

          location / {
              root   /usr/share/nginx/html;
              index  index.html index.htm;
          }

          error_page 404 /404.html;
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-b
  labels:
    app: service-b
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-b
  template:
    metadata:
      labels:
        app: service-b
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: "/usr/share/nginx/html/"
        - name: config
          mountPath: "/etc/nginx/"
      volumes:
      - name: html
        configMap:
          name: service-b
      - name: config
        configMap:
          name: service-b-nginx.conf
---
apiVersion: v1
kind: Service
metadata:
  name: service-b
spec:
  selector:
    app: service-b
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

```


* Warning : path /path-b(/|$)(.*) cannot be used with pathType Prefix


### Doubts
1. When we delete namespace do all resources get deleted?
2. What is a port forward using kubectl?

### References
1. https://stackoverflow.com/questions/65193758/enable-ingress-controller-on-docker-desktop-with-wls2
2. https://stackoverflow.com/questions/61616203/nginx-ingress-controller-failed-calling-webhook
3. https://github.com/kubernetes/ingress-nginx/tree/main
4. https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/ingress/controller/nginx
5. https://www.youtube.com/watch?v=72zYxSxifpM (That Devops Guy)
6. https://stackoverflow.com/questions/38230452/what-is-command-to-find-detailed-information-about-kubernetes-masters-using-ku
7. https://github.com/ankur6ue/Flask_Ingress/blob/main/ingress.yaml
8. https://www.telesens.co/2021/05/14/accessing-a-custom-flask-microservice-from-a-kubernetes-ingress-controller/


## Day 10

* Learnt about Flask and caching

* Extensions are extra packages that add functionality to a Flask application

* Deserialization is the process whereby a lower-level format (such as a series of bytes or a string that has been transferred over a network, or stored in a data store) is translated into a readable object or other data structure. In JavaScript, for example, you can deserialize a JSON string to an object by calling the function JSON

* Marshmallow is useful because it deserializes the json objects passed to your REST API, and raises consistent 400 error messages when the user violates the API. Like Pydantic, it also helps validate data

* Migrations are meant to keep the database in sync with the Models. Migrations also allow you to change your database in a controlled way: Any change you make to your database will be represented in a migration and may be "played back" on a different machine, thus making it easy to version track the database. This is also important because database must be persistent. When you add a new field, old the existing data must not get erased

* __name__ is a variable that exists in every Python module, and is set to the name of the module

* Scripts vs Module : A script is a Python file that's intended to be run directly. When you run it, it should do something. A module is a Python file that's intended to be imported into scripts or other modules.

* A slug is an alternative to a name that would otherwise not be acceptable for various reasons - e.g. containing special characters, too long, mixed-case, etc. - appropriate for the target usage
```
{
   "title": "Hello World",
   "slug": "hello_world"
}

```

* In Declarative mapping (Declarative ORM), we use classes to define database tables. In Imperative Mapping (Classical ORM) we define the class separately from the table mapping., and use Table objects explicitly. We mostly use Declarative mapping

* The documentation of sqlalghemy is really bad (from a Youtube comment)

* There are 3 types of relationships possible bw 2 tables:
  * 1 to 1
  * 1 to many
  * many to many


* back_populates defines how two related models reference each other within the Python code. **It does not enforce referential integrity at the database level** Instead it allows SQLAlchemy to maintain in-memory synchronization between two related objects at the ORM level (not at database level)

```
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child", back_populates="parent")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
    parent = relationship("Parent", back_populates="children")

parent = Parent()
child = Child()
child.parent = parent
print(parent.children)
# OUTPUT : [Child(...)]. Without back_populate this would have been empty

```
* Note : Mapped classes give you the ability to work with tables as if they are objects in memory; along the same lines, relationships give you the ability to work with foreign keys as if they are references in memory.

* Model vs Schema : 
  * Models interact with the database, while Schemas format and validate API data. 
  * Models are used for storing & querying data in a database, whereas Schemas help validate input data from the user

* Marshmallow in Python is like Zod in Typescript. Both are schema validation libraries

* Flask-Restful, an extension for Flask that simplifies building RESTful APIs. Its advantages are: it provides a Resource class that allows you to define HTTP methods (GET, POST, PUT, DELETE) as class methods, making it easier to organize and manage your API endpoints

* While converting from flask to flask-restful, classes are formed on basis of which endpoint they are using, same endppoint (for example /hello) would result in same class

```
### Using flask
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/hello', methods=['GET'])
def hello():
    return jsonify({"message": "Hello, World!"})

if __name__ == '__main__':
    app.run(debug=True)


### Using flask-restful

from flask import Flask, jsonify
from flask_restful import Api, Resource

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {"message": "Hello, World!"}

api.add_resource(HelloWorld, "/hello")

if __name__ == '__main__':
    app.run(debug=True)

##### For a full fledged CRUD example with create, update and delete, check out Chatgpt

```

* One reason for using Flask over FastAPI is Flask maintainers are really good, better compared to FastAPI maintainers (https://www.reddit.com/r/Python/comments/1g83rjr/why_people_still_using_flask_after_fastapi_release/)

* The folder structure for the project is as follows

```
|   app.py
|   config.py
|   extensions.py
|   poetry.lock
|   pyproject.toml
|   roles.sql
|   users.sql
|
+---api
|   |   views.py
|   |   __init__.py
|   |
|   +---resources
|   |   |   user.py
|   |   |   __init__.py
|   |
|   +---schemas
|   |   |   user.py
|   |   |   __init__.py
|   |
+---migrations
+---models
|   |   users.py
|   |   __init__.py
|   |
+---instance
|       db.sqlite3


```


* The Flask code is as belows

```
### extensions.py
from flask_marshmallow import Marshmallow
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_caching import Cache

db = SQLAlchemy()
migrate = Migrate
ma = Marshmallow()
cache = Cache()

### config.py
import os
FLASK_RUN_HOST = os.environ.get("FLASK_RUN_HOST","0.0.0.0")
FLASK_RUN_PORT = os.environ.get("FLASK_RUN_PORT", 9000)
SQLALCHEMY_DATABASE_URI = "sqlite:///db.sqlite3"
CACHE_TYPE = "RedisCache"

### app.py
from flask import Flask
from extensions import db, cache
from api.views import register_routes  # Import function to register API routes

app = Flask(__name__)
app.config.from_object("config")
register_routes(app)
db.init_app(app)
cache.init_app(app)

if __name__ == '__main__':
    app.run(
        host=app.config.get("FLASK_RUN_HOST"),
        port=app.config.get("FLASK_RUN_PORT"),
        debug=True
    )


### models/users.py
from extensions import db

class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.String, nullable=False)
    email = db.Column(db.String, nullable=False)
    age = db.Column(db.Integer, nullable=False)
    # define a many-to-many relationship between User table and Role table, using an association table named "user_role"
    # By using back_pupulates, when we assign a role to user using user.roles = role, when we print(role.users), 
    # we will get all users with that role instead of an empty list
    roles = db.relationship("Role", secondary="user_role", back_populates="users")

class Role(db.Model):
    __tablename__ = "roles"

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.String(30), nullable=False)
    slug = db.Column(db.String(30), nullable=False, unique=True)

    users = db.relationship("User", secondary="user_role", back_populates="roles")

class UserRole(db.Model):
    __tablename__ = "user_role"

    user_id = db.Column(db.Integer, db.ForeignKey("users.id"), primary_key=True)
    role_id = db.Column(db.Integer, db.ForeignKey("roles.id"), primary_key=True)

### api/views.py
from flask import Blueprint, jsonify
from flask_restful import Api
from marshmallow import ValidationError
from api.resources.user import UserList, UserResource, RoleList, HomePage

def register_routes(app):
    api = Api(app)
    api.add_resource(HomePage, "/")
    api.add_resource(UserList, "/users")
    api.add_resource(UserResource, "/users/<int:user_id>")
    api.add_resource(RoleList, "/roles")


### api/resources/user.py

from flask import request
from flask_restful import Resource
from api.schemas.user import UserSchema, RoleSchema
from extensions import db, cache
from models.users import User, Role

class HomePage(Resource):
    def get(self):
        return "Welcome to homepage"

class UserList(Resource):
    def get(self):
        users = User.query.all()
        schema = UserSchema(many=True)
        return {"results": schema.dump(users)}

    def post(self):
        schema = UserSchema()
        user = schema.load(request.json)
        db.session.add(user)
        db.session.commit()

        return {"msg": "User created", "user": schema.dump(user)}


class UserResource(Resource):
    def get(self, user_id):
        user = cache.get(f"user_id_{user_id}")
        if user is None:
            user = User.query.get_or_404(user_id)
            cache.set(f"user_id_{user_id}", user)
        schema = UserSchema()

        return {"user": schema.dump(user)}

    def put(self, user_id):
        schema = UserSchema(partial=True)
        user = User.query.get_or_404(user_id)
        user = schema.load(request.json, instance=user)

        db.session.add(user)
        db.session.commit()

        return {"msg": "User updated", "user": schema.dump(user)}

    def delete(self, user_id):
        user = User.query.get_or_404(user_id)

        db.session.delete(user)
        db.session.commit()

        return {"msg": "User deleted"}


class RoleList(Resource):
    @cache.cached(60)
    def get(self):
        roles = Role.query.all()
        schema = RoleSchema(many=True)
        return {"results":schema.dump(roles)}


### api/schemas/user.py
from marshmallow import validate, validates_schema, ValidationError
from marshmallow.fields import String
from extensions import ma
from models.users import User, Role

class UserSchema(ma.SQLAlchemyAutoSchema):
    name = String(required=True, validate=[validate.Length(min=3)], error_messages={
        "required": "The name is required",
        "invalid": "The name is invalid and needs to be a string",
    })
    email = String(required=True, validate=[validate.Email()])

    @validates_schema
    def validate_email(self, data, **kwargs):
        email = data.get("email")

        if User.query.filter_by(email=email).count():
            raise ValidationError(f"Email {email} already exists.")

    class Meta:
        model = User
        load_instance = True
        exclude = ["id"]


class RoleSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Role

```
* Created the database and seeded some data using the following commands

```
### creating the database in the command prompt
flask shell
from app import db
from models.users import User
from models.users import Role
from models.users import UserRole
db.create_all()

### seeding the database (fillinf/populating it with some data)
sql = open("users.sql", "r")
statement = sql.read()
from sqlalchemy import text
db.session.execute(text(statement))
db.session.commit()

```

* The SQL used for seeding the tables are as follows
```
INSERT INTO roles (name, slug)
VALUES 
    ("Admin", "admin"),
    ("Account Manager", "account_manager"),
    ("Regional Manager", "regional_manager")

INSERT INTO users (name, email, age)
VALUES
    ('John Doe', 'john@google.com',  25),
    ('Jane Smith',  'jane@msn.com',  30),
    ('Bob Johnson',  'bob@hotmail.com',  40),
    ('Jack Edwards',  'jak@gmail.com',  40),
    ('Julie Banks',  'julie@gmail.com',  40)


```
* Tested the application at 127.0.0.1:9000, 127.0.0.1:9000/users, 127.0.0.1:9000,roles. Also tested using Postman with a POST request to 


* Since we cache the data for roles, if we insert a new role and then try to retrive all roles we will not the get the latest data.

* We can check the data within redis using the following redis cli commands
```
#### Important redis cli commands

# activate redis cli
redis-cli

# see all keys present
keys *

# get value corresponding to key (get <key-name>)
GET flask_cache_view//roles

# delete a key
DEL flask_cache_view//roles

# set <key-name> <key-value> to set a value eg. set age 26

# check if key exists, 0 means no longer present
EXISTS flask_cache_view//roles

# Check time to live, -1 means key will never expire, we can make it expire using EXPIRE
TTL flask_cache_view//roles

```
* Time to Live (TTL) : You can control the freshness of your cached data by applying a time to live (TTL) or expiration to your cached keys. After the set time has passed, the key is deleted from the cache, and access to the origin data store is required along with reaching the updated data. In our case, we have put `@cache.cached(60)` so it will expire after 60 seconds

* Errors:
  * TypeError: models.users.User() argument after ** must be a mapping, not User

### Doubts
1. Why do we need to store database migrations?
2. What is the difference bw Python script and Python module?
3. Why do we pass __name__ to the Flask class?
4. What us the difference bw Declarative and Imperative style mentioned in SQLAlchemy docs?
5. When do we use back populate in SQLAlchemy?
6. How to avoid circular dependency issues with backpopulate?
7. Instead defining a foreign key sufficient, just like we do at the database level? Why the whole need for relationship?
8. Is Marshmallow a simple way of doing jsonify?
9. What is the difference bw schema and model in Flask?
10. Is there a way to test a SQLAlchemy connection?
11. What are Flask blueprints?
12. How to prevent caching in Flask?
13. Can we cache a webpage in redis?
14. How do we force refresh of cache, so that it reflects latest data?
15. What are use cases for data structures like list and set in Redis? What is redis expiration algorithm?

### References
1. https://www.reddit.com/r/flask/comments/zkf9kn/why_use_marshmallow_with_rest_api_and_orm/
2. https://github.com/demoskp/flask-caching-tutorial/tree/master
3. https://stackoverflow.com/questions/39975996/django-migrations-what-benefits-do-they-bring
4. https://blog.miguelgrinberg.com/post/why-do-we-pass-name-to-the-flask-class
5. https://stackoverflow.com/questions/4357007/what-does-slug-mean
6. https://stackoverflow.com/questions/39869793/when-do-i-need-to-use-sqlalchemy-back-populates
7. https://www.youtube.com/watch?v=aAy-B6KPld8&t=515s (Arjan Codes)
8. https://stackoverflow.com/questions/46462152/is-it-necessary-to-use-relationship-in-sqlalchemy
9. https://stackoverflow.com/questions/51335298/concepts-of-backref-and-back-populate-in-sqlalchemy
10. https://stackoverflow.com/questions/53269323/flasks-jsonify-function-inconsistent-with-flask-marshmallow
11. https://stackoverflow.com/questions/58896928/how-to-connect-to-sqlite-from-sqlalchemy
12. https://www.youtube.com/watch?v=jgpVdJB2sKQ&t=258s (Redis CLI commands)
13. https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/cache-validity.html
14. https://stackoverflow.com/questions/36302972/how-to-update-redis-after-updating-database


## Day 11
* First, lets talk sentinel. Sentinel manages the failover, it doesn't configure Redis for HA. It is an

* Kubernetes enables Pods to communicate with each other across Nodes in a cluster. This Pod network is implemented through networking plugins such as Flannel, Calico, Canal, Weave Net.

* To find the network plugin, type `kubectl get pods -n kube-system`

* Some common Ingress controllers include:
  * NGINX Ingress Controller - The default Ingress controller. It uses NGINX as a reverse proxy and load balancer.
  * Traefik - A cloud native edge router that works as an Ingress controller. It can be configured through Kubernetes Manifests.
  * HAProxy Ingress - Uses the HAProxy load balancer as an Ingress controller. Provides high availability.
  * GCE - The Ingress controller provided by Google Kubernetes Engine. Uses a GCP load balancer.
  * Istio Ingress - Provides Ingress capabilities as part of the Istio service mesh.

* Redis is essentially a Python dictionary (https://realpython.com/python-redis/)

* We need atleast 3 redis instances 

* Was unable to create a high availability Redis cluster following That Devops Guy tutorial, hence created a simple Redis pod using ref 8


```
### redis-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379


### redis-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379


```


* To test also created a Redis client pod
```
### redis-client-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-client
spec:
  containers:
  - name: redis-client
    image: redis:latest
    command: ["redis-cli"]
    tty: true

```



* Ran the following commands in terminal

```
kubectl -n redis get all

kubectl delete namespace redis

kubectl create namespace redis1

kubectl -n redis1 get all
# No resources found in redis1 namespace.

kubectl -n redis1 apply -f redis-deployment.yaml

kubectl -n redis1 get all

kubectl -n redis1 apply -f redis-service.yaml
# pod/redis-56bc99cc99-mtf52   1/1     Running   0          2m12s
# service/redis   ClusterIP   10.103.20.226   <none>        6379/TCP   22s
# deployment.apps/redis   1/1     1            1           2m12s
# replicaset.apps/redis-56bc99cc99   1         1         1       2m12s


kubectl -n redis1 apply -f redis-client-pod.yaml

kubectl -n redis1 exec -it redis-client -- redis-cli -h redis 

kubectl get pods -n redis1 -o wide

```

* Errors:
  * Docker Failed to start : delete %appdata%\Docker\settings.json and let Docker to create a new one
  * Redis pods not starting, logs just showing Defaulted container "redis" out of: redis, config (init). Was not able to resolve

### Doubts
1. What is local volume standard storage?
2. What is an initContainer?
3. What is the difference bw host path and local volume?
4. How to delete all the resources within a namespace?
5. When I do kubectl get nodes, it shows only 1 control plane?
6. Can all pods in Kubernetes communicate with each other, even across different nodes and different namespaces?
7. How to identify network plugin in Kubernetes cluster?

### References
1. https://stackoverflow.com/questions/31143072/redis-sentinel-vs-clustering
2. https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/storage/redis/kubernetes
3. https://stackoverflow.com/questions/51209870/minikube-vs-kubernetes-in-docker-for-windows
4. https://forums.docker.com/t/solved-docker-failed-to-start-docker-desktop-for-windows/106976
5. https://weng-albert.medium.com/local-volume-vs-hostpath-en-8c214c1dca74
6. https://stackoverflow.com/questions/63490278/kubernetes-persistent-volume-hostpath-vs-local-and-data-persistence
7. https://stackoverflow.com/questions/47128586/how-to-delete-all-resources-from-kubernetes-one-time
8. https://medium.com/@harshaljethwa19/redis-deploying-redis-on-kubernetes-building-chat-applications-with-redis-pub-sub-on-kubernetes-f81a56ec0273
9. https://medium.com/@achanandhi.m/how-to-identify-the-network-plugin-in-your-kubernetes-cluster-1f0d1dcdd937
10. https://zeet.co/blog/kubernetes-networking-101-your-straightforward-guide-to-connecting-pods-and-services


## Day 13 and 14 (incomplete, go through all the files in end-to-end project and type each line by hand)

* Network identity : refers to the concept that each pod gets its own unique IP address, enabling pods to communicate with each other directly (also known as pod per ip model)

* Stateful set : StatefulSet runs a group of Pods, and maintains a sticky identity for each of those Pods (useful for managing applications that need persistent storage or a stable, unique network identity)

* Each pod in the StatefulSet receives a stable network identity by assigning them unique, persistent hostnames (derived from the StatefulSet name and ordinal index such as db-0, db-1) that remain consistent even if a pod is rescheduled or restarted

* In Deployments, pods are interchangeable (mainly for stateless application like a Flask application), whereas in Stateful Set, each pod has unique identity and are not interchangeable. 

* The IP address of a pod in Stateful set can change (especially upon pod restart or rescheduling), but the DNS record and hostname (which together form network identity) will remain the same. We can n go to the pod of a stateful set using the following DNS : podname-{replica-index}.{serviceName}.default.svc.cluster.local

* As done earlier, if we want to ping a pod,we cannot do it directly from host system. Instead we can
  * Run the ping command from a worker node.
  * Deploy another container (preferably in a different namespace) and ping from there.
  * Forward the IP range of the worker nodes to your client device with the DNS e.g. using something like sshuttle

* Headless service : Headless service allows direct access to individual pods without a load balancer or a single virtual IP address, instead exposing the pods' IP addresses through DNS records, useful for scenarios requiring direct pod communication

* Whenever the clusterIP is set to none that means that it is a headless service. StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. Because unlike the stateless deployments the statefulset pods will have Stable, unique network identifiers(pod names).

* The only thing that I found that works is that headless (with clusterIP: None) service creates the <pod-name>.<service-name>.<namespace>.svc.<cluster-suffix> entries (plus the <service-name>.<namespace>.svc.<cluster-suffix> as A records for all endpoints). Which means I need two services for the StatefulSet. Took quite a bit of trial and error to find out

* By default, Kubernetes pod and service DNS names are only resolvable from within the cluster, not from your local machine. Instead we have to run a temporary pod within the same cluster and then can do a nslookup


```
### flask-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-ingress







```



```

docker-compose up --build

## tested POST request using POSTMAN, this is the equivalent cURL command
curl --location 'http://localhost:5002/orders' \
--header 'Content-Type: application/json' \
--data '{
    "id" : "o553",
    "item" : "Shoes",
    "quantity" : 2,
    "user_id" : "u101"
}'


curl --location --request GET http://localhost:5002/orders/o553

docker-compose up --build

curl --location 'http://localhost:5001/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id" : "u101",
    "name" : "Shoes",
    "email" : "john@gmail.com"
}'

curl --location --request GET http://localhost:5001/users/u101
## OUTPUT: {"email":"john@gmail.com","id":"u101","name":"Shoes"}


```


```
kubectl get nodes

kubectl get namespaces

kubectl create namespace final-proj-1

kubectl -n final-proj-1 apply -f redis-statefulset.yaml

kubectl -n final-proj-1 apply -f client-pod.yaml

kubectl -n final-proj-1 describe pod client-pod

## Change the current namespace
kubectl config set-context --current --namespace=final-proj-1

## Check the current namespace
kubectl config view --minify | grep namespace

kubectl describe pod alpine

kubectl logs apline

kubectl apply -f apline.yaml

kubectl exec -it alpine -- sh

kubectl get pods -o wide 
ping 10.1.0.88

apk update
apk get curl
```

```
docker build --tag user-service . 

docker tag user-service ksshr/user-service

docker login

docker push ksshr/user-service

#kubectl apply -f kubernetes\user-deployment.yaml
kubectl apply -f user-deployment.yaml

kubectl port-forward svc/user-service 80
# OUTPUT : Forwarding from 127.0.0.1:80 -> 5001
# OUTPUT : Forwarding from [::1]:80 -> 5001
# Visited 127.0.0.1 in the browser successfully (image pull working as expected)
# I had commented the init container

docker build --tag order-service .
docker tag order-service ksshr/order-service
docker login
docker push ksshr/order-service

kubectl apply -f order-deployment.yaml
kubectl port-forward svc/order-service 80

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml

kubectl get all -n ingress-nginx 

kubectl get pods -n default
kubectl -n default delete service/service-b
kubectl -n default delete service/service-a
kubectt delete pod service-a-bb574c9fb-sd9zf



kubectl get ingress
kubectl get service ingress-nginx-controller --namespace=ingress-nginx




```



* Spiderpool is an underlay and RDMA network solution for the Kubernetes.

* CrashLoopBackOff means that Kubernetes was able to successfully pull the image and start your container, but then the container process exited with an error. Faced this error when trying to deploy a simple Alpine container (more details in errors section)

* In Kubernetes, every pod gets its own ip address from 10.*, that is usable only within the cluster

* port-forward : This feature of kubectl simply tunnels the traffic from a specified port at your local host machine to the specified port on the specified pod. API server then becomes, in a sense, a temporary gateway between your local port and the Kubernetes cluster. Used mainly for testing/debugging (in a way similar to port mapping in docker)

* The flask-ingress.yaml you’re using defines an Ingress resource, but to actually expose it, you need a running Ingress Controller, like NGINX.

* Issue : NGINX Ingress Controller was using leftover config or not seeing your new flask-ingress at all.

```

kubectl get ingressclass
# Add ingressClassName: nginx to your Ingress (flask-ingress.yaml)
kubectl apply -f flask-ingress.yaml
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
kubectl describe ingress flask-ingress
# Rules:
#  Host        Path  Backends
#  ----        ----  --------
#  *
#              /users    user-service:80 (10.1.0.109:5001)
#              /orders   order-service:80 (10.1.0.110:5002)

kubectl get ingress -A
kubectl delete ingress service-a -n default 
kubectl delete ingress service-b -n default 
kubectl get ingress -A

kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80

```
* When trying to go to http://127.0.0.1:8080/users, it is giving URL not found error, same for http://127.0.0.1:8080/orders. Create a simple route in order_service.py using `@app.route('/', methods=['GET'])` and that route worked. However even when calling http://127.0.0.1:8080/orders/orders, the above route was called. So no matter what I type after http://127.0.0.1:8080/orders, only the / route is called

* Capture Group : 

* Service and Ingress resource types that define two ways to control the inbound traffic in a Kubernetes cluster. 

* Load Balancer vs Ingress controller : Load balancer is configured to distribute traffic to the pods in your Service on a given port. The LoadBalancer only works at layer 4 - the Service is unaware of the actual applications, and can't make any additional routing considerations. Ingress controllers work at layer 7, and can use more intelligent rules to distribute application traffic. A common use of an Ingress controller is to route HTTP traffic to different applications based on the inbound URL.

* Creating a Postgres deployment with multiple replicas pointing to the same persistent volume (PV) can cause corrupted data, as two instances can try to write the same data at the same time. Moreover create a simple StatefulSet and set replicas: 2 is wrong as each StatefulSet pod gets its own storage and its own PersistentVolumeClaim and Kubernetes doesn't know how to trigger synchronization in whatever application the pod is running. A good way to solve this is to use Statefulsets and configure Replication on the database level, for example using Postgres native replication or Patroni. And use the volumeClaimTemplate system so each pod gets its own distinct storage (refer 24, 25, 26)

* Each Pod in the StatefulSet is assigned its own Persistent Volume (PV) and Persistent Volume Claim (PVC)

* Checkout https://kubernetes-tutorial.schoolofdevops.com/13_redis_statefulset/

```
git init
# Add a .gitignore

git add kubernetes/
git add user-service/  
git commit -m "add all files"  
git remote add origin https://github.com/shravankshenoy/End-to-End-Kubernetes-Devops-Project.git
git pull origin main --allow-unrelated-histories 
git push origin master:main 

```

* Created an architecture diagram like https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project using 19. Took inspiration from eks workshop (ref 27)

* Potential next steps
  * Helm chart version of all this
  * Separate namespaces
  * Secrets for DB credentials
  * Metrics + liveness/readiness probes

* Errors faced:
  * error: src refspec main does not match any
  error: failed to push some refs to 'https://github.com/shravankshenoy/End-to-End-Kubernetes-Devops-Project.git'.Updates were rejected because the remote contains work that you do not have locally
  *  pg_config is required to build psycopg2 from source.  Please add the directory containing pg_config to the $PATH or specify the full executable path with the option or with the pg_config option in 'setup.cfg'. Solution : RUN apt-get update && apt-get -y install libpq-dev gcc OR pip install psycopg2-binary
  * sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) connection to server at "postgres" (172.18.0.2), port 5432 failed: Connection refused. Is the server running on that host and accepting TCP/IP connections?
  * psycopg2.errors.UndefinedTable: relation "users" does not exist 
  * Status CrashLoopBackOff : Alpine does not have bash by default so we need to change /bin/bash to /bin/sh, which is present in Alpine:
  * The Pod "client-pod" is invalid: spec: Forbidden: pod updates may not change fields other than spec.containers[*].image, spec.initContainers[*].image, spec.activeDeadlineSeconds, spec.tolerations (only additions to existing tolerations) or spec.terminationGracePeriodSeconds (allow it to be set to 1 if it was previously negative) : Reason: To change anything else (like command, tty, stdin, name), you need to recreate the pod. We can do that by `kubectl delete pod client-pod` followed by `kubectl apply -f client-pod.yaml`
  * Warning: path /users(/|$)(.*) cannot be used with pathType Prefix

### Doubts
1. How to test a docker image? And how to manage test data with docker images?
2. What is depends_on in docker compose?
3. What is the significance of version in docker compose file?
4. What does `Base.metadata.create_all(engine)` do and how is it useful for local testing?
5. Should the selector in service match metadata or spec.template.metadata.labels? What is the purpose of the metadata in deploment/statefulset?
6. What is network identity in Kubernetes?
7. Does the ip address of a statefulset pod never change
8. When do we set clusterIp to none?
9. How can I expose a Statefulset with a load balancer?
10. How to ping a statefulset pod kubernetes?
11. How does `kubectl port-forward` used in day 9 works?
12. Is kubernetes port-forward similar to Docker port mapping?
13. How to force Kubernetes to repull an image? (delete the pod)
13. How do I push a local Git branch (or master branch) to main branch in the remote?
14. What's the purpose of using Zookeeper rather than just databases for managing distributed systems?
15. What is the difference between Helm and Kustomize?
16. How do we test microservices? What are different kind of tests involved?
17. How is alb (AWS Load Balancer) different from Nginx Ingress controller?
18. Is ingress-nginx really a load balancer or not?
19. Since each pod of a stateful set has its own PVC, will the data in the different PVC be in sync?
20. If I declare 2 replicas of PostgreSQL StatefulSet pods in k8s, are they the same database or they just share the volume? If neither, then isnt it better to use deployment with PVC?


### References
1. https://www.reddit.com/r/docker/comments/jub4fs/how_do_folks_manage_test_data_with_docker_images/
2. https://github.com/GoogleContainerTools/container-structure-test
3. https://stackoverflow.com/questions/35104097/how-to-install-psycopg2-with-pg-config-error
4. https://superuser.com/questions/1646555/how-to-get-the-hostname-of-a-service-in-kubernetes
5. https://serverfault.com/questions/1119093/when-and-how-we-should-specify-clusterip-inside-the-service-yaml-file
6. https://stackoverflow.com/questions/52707840/what-is-a-headless-service-what-does-it-do-accomplish-and-what-are-some-legiti
7. https://stackoverflow.com/questions/52274134/connect-to-other-pod-from-a-pod
8. https://spacelift.io/blog/crashloopbackoff
9. https://github.com/infrabricks/kubernetes-standalone/blob/master/examples/alpine.yml
10. https://stackoverflow.com/questions/75325649/what-is-the-purpose-of-headless-service-in-kubernetes
11. https://stackoverflow.com/questions/45142855/bin-sh-apt-get-not-found
12. https://stackoverflow.com/questions/59559438/retrieve-the-full-name-of-a-service-in-kubernetes
13. https://stackoverflow.com/questions/51468491/how-does-kubectl-port-forward-create-a-connection
14. https://medium.com/@haroldfinch01/how-do-i-force-kubernetes-to-re-pull-an-image-cf2b8c4854bc
15. https://www.linkedin.com/pulse/difference-between-cache-vs-database-githin-nath
16. https://stackoverflow.com/questions/36312640/whats-the-purpose-of-using-zookeeper-rather-than-just-databases-for-managing-di
17. https://stackoverflow.com/questions/60519939/what-is-the-difference-between-helm-and-kustomize
18. https://www.lambdatest.com/blog/testing-microservices/
19. https://app.diagrams.net/
20. https://aws.amazon.com/blogs/containers/exposing-kubernetes-applications-part-3-nginx-ingress-controller/
21. https://learn.microsoft.com/en-us/answers/questions/295210/is-ingress-nginx-really-a-load-balancer-or-not
22. https://www.eksworkshop.com/docs/introduction/getting-started/microservices
23. https://www.reddit.com/r/kubernetes/comments/ph64go/statefulsets_storage_sync/
24. https://stackoverflow.com/questions/68516778/if-i-declare-2-replicas-of-postgresql-statefulset-pods-in-k8s-are-they-the-same
25. https://www.reddit.com/r/kubernetes/comments/z7jrob/can_you_create_a_postgres_deployment_with/
26. https://github.com/zalando/postgres-operator
27. https://www.eksworkshop.com/docs/introduction/getting-started/microservices
28. https://stackoverflow.com/questions/47837087/nginx-ingress-rewrite-target
29. https://stackoverflow.com/questions/46720563/kubernetes-ingress-load-balancer-rewrites-everything-to-index-why
30. https://stackoverflow.com/questions/59491324/is-there-a-way-to-kubectl-apply-all-the-files-in-a-directory

## Day 15 (incomplete)

* Fixtures : Fixtures are functions, which will run before each test function to which it is applied

```
## test_maths.py

import pytest

@pytest.fixture
def input_value():
    input = 39
    return input

def test_divisible_by_3(input_value):
    assert input_value % 3 == 0

## pytest test_maths.py

```

### Doubts
1. Is fixture a decorator?

### References
1. https://www.tutorialspoint.com/pytest/pytest_fixtures.htm


## Day 16 and 17

* CQRS is an architectural pattern that separates the responsibility of executing commands that change data (write operations) from the responsibility of retrieving data (read operations).

* Event sourcing is a software architectural pattern where changes to an application's state are stored as a sequence of events, rather than directly updating the current state.

* Event Sourcing vs SCD2 : Event Sourcing focuses on capturing and storing all changes as a sequence of events, while SCD2 focuses on managing historical changes to dimension data in a data warehouse. Event Sourcing is a software architectural pattern, often used in microservices and complex systems, while SCD2 is a data warehousing technique

* Pull only : Kafka does not actively push notifications to consumers; instead, consumers proactively pull messages from Kafka topics. The Consumer Design section in the Apache Kafka documentation explains why "consumer pulling" was choosed over "broker pushing".

* consumer.subscribe() in Kafka does indeed cause the consumer to continuously poll the broker for new messages.

* The following code was used to create a Python Kafka producer and consumer. Note that consumer and producer python code was run in separate terminals (in one terminal `python kafka_producer.py` multiple times in second terminal `python kafka_consumer.py`)

```
########## kafka_producer.py

from confluent_kafka import Producer, Consumer
from confluent_kafka.admin import AdminClient, NewTopic

import socket
import json
import uuid
import random
import datetime


### MESSAGE DETAILS

order_id = uuid.uuid4().hex[-8:]

order_details = {
        'status': 100, 
        'timestamp': int(datetime.datetime.now().timestamp() * 1000), 
        'order': {
            'extra_toppings': ['Onion', 'Olives'], 
            'sauce': 'Tomato', 
            'cheese': 'Provolone', 
            'main_topping': 'Pepperoni', 
            'customer_id': 'd60c39c7744941eb9227c3e47fa791c9', 
            'username': 'user' + str(random.randint(1000, 9999))
        }
    }


#### KAFKA PRODUCER CONFIG

HOSTNAME = socket.gethostname()

producer_common_config = {
            "partitioner": "murmur2_random",
        }

producer_extra_config={
        "client.id": f"""pizza_client_webapp_{HOSTNAME}""",
    }

config_kafka = {
    'bootstrap.servers': 'localhost:9092'
    }


### KAFKA PRODUCER

PRODUCER = Producer(
            {
                **producer_common_config,
                **config_kafka,
                **producer_extra_config,
            }
        )


PRODUCER.produce(
            "pizza-destroyed",
            key=order_id,
            value=json.dumps(order_details).encode(),
        )

PRODUCER.flush()


########## kafka_consumer.py


from confluent_kafka import Consumer
import socket
import json

HOSTNAME = socket.gethostname()

### KAFKA CONSUMER CONFIG

consumer_common_config = {
            "enable.auto.commit": False,
            "auto.offset.reset": "earliest",
            "max.poll.interval.ms": 3000000,
        }

config_kafka = {
    'bootstrap.servers': 'localhost:9092'
    }

consumer_extra_config={
        "group.id": f"""pizza_assemble_{HOSTNAME}""",
        "client.id": f"""pizza_client_assemble_{HOSTNAME}""",
    }

CONSUMER = Consumer({
    **consumer_common_config,
    **config_kafka,
    **consumer_extra_config
})

CONSUMER.subscribe(["pizza-ordered"])

try:
    while True:
        event = CONSUMER.poll(1.0)

        if event is None:
            continue
        if event.error():
            print(f'Error while consuming: {event.error()}')
        else:
            # Parse the received message
            order_id = event.key().decode()
            try:
                order_details = json.loads(event.value().decode())
                order = order_details.get("order", dict())
                print(order)
            except Exception as e:
                print(e)
                            
except KeyboardInterrupt:
    pass
finally:
    # Close the consumer gracefully
    CONSUMER.close()


######## docker-compose.yml

version: '2'

services:
  zookeeper:
    image: wurstmeister/zookeeper:latest
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka:latest
    ports:
      - "9092:9092"
    expose:
      - "9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: "my-topic:1:1"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

```

* The following commands were used to inspect the messages stored within Kafka topic

```
docker ps ## The container id used below is docker container id, not zookeeper container id

## To see all the messages in the topic "pizza-ordered"
docker exec -it cea71d503965 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic pizza-ordered

## To see message in a particular partition at a particular offset
docker exec -it cea71d503965 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic pizza-ordered --partition 0 --offset 2 --max-messages 1

## To see all messages in a particular partition from a particular offset
docker exec -it cea71d503965 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic pizza-ordered --partition 0 --offset 2

## To see number of partitions and replications for a pizza-ordered topic
docker exec -it cea71d503965 /opt/kafka/bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic pizza-ordered
## Output : Topic: pizza-ordered    TopicId: 5ZDrQLomSTyUfWCjNZoGJw PartitionCount: 1       ReplicationFactor: 1

## To see number of partitions and replications for a pizza-destroyed topic
docker exec -it cea71d503965 /opt/kafka/bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic pizza-destroyed
## Output : Topic: pizza-destroyed  TopicId: LwqER9X-QCOuFxFnKYi2kw PartitionCount: 2       ReplicationFactor: 1  

```

* Deserialization : Kafka brokers deal exclusively in bytes. So event data Kafka receives and stores is just raw byte streams. When the consumer get the data, the first step is to deserialize the payload i.e. convert it from bytes to usable data (such as jsonor any format the program can use)

* poll() : To get the messages from Kafka to consumer. When a consumer fetches data from a Kafka broker, it stores it in its local cache

* Kafka stores messages in topics, and each topic can have multiple partitions. Each consumer reads messages from one partition of a topic.

* Offset commit : An integer which keeps track of the messages that consumers read. For ex, if a consumer has read 5 messages from a partition, offset commit is 4. So next time onwards, the consumer consumes messages with offsets 5 onwards

* 4 ways to commit offset by consumer
  * Auto commit (default) : at every five seconds Kafka commits the largest offset returned by the poll() method 
  * Manual sync commit
  * Manual async commit
  * Commit specific offset

* Auto commit drawbacks : There is a very high chance of data loss in case of application failure. Let’s say poll() returns 100 messages, which the consumer stores in its cache, and the consumer processes 60 messages when the auto-commit happens. Then, due to some failure, the consumer crashes (due to which cache is lost). When a new consumer goes live to read messages, it commences reading from offset 101, resulting in the loss of messages between 61 and 100 (refer 10 for different scenarios)

* Exact once processing :  Each message is consumed exactly once, not zero times, not more than 1 time

* Partitioning : Dividing a topic into multiple ordered sequences of records/messages enabling scalability, parallelism. Each ordered sequence of message is called a partition. 

* Every message is characterized by 2 things : partition number and offset number

* Partition rebalancing : The process by which Kafka redistributes partitions across consumers to ensure that each consumer is processing an approximately equal no. of partitions

* auto.offset.reset configuration : Defines how Kafka consumers should behave when no initial committed offsets are available for the partitions assigned to them

* max.poll.records : Set to whatever number you like such that at most you will get that many records on each poll.

* max.poll.interval.ms : Places an upper bound on the amount of time that the consumer can be idle before fetching more records

* Polling vs Fetching : Polling is like asking: “Is it ready yet?” Fetching is like saying: “Give me what’s ready!” 

* Kafka partition key : The producer uses this key to ensure that related messages are sent to the same partition, enabling ordering guarantees within that partition. For example all requests placed by a particular user go to the same partition (say partition 0). This is important, because if they go to different partition (say partition 1), and the consumer/service reading from partition 1 reads before consumer/service reading from partition 0, then the whole order is spoilt. Say a consumer first places an order and then cancels or updates it, then reversing the order will cause an error

* Consumer groups : Help process data in parallel and making sure at the same time that no message is read twice. If all the consumer instances have the same consumer group, then the records will effectively be load balanced over the consumer instances. 

* For real world use case of consumer group refer 14 (essentially when we have a Payment service as producer, Notification and Shipping service as consumers, and there are multiple instances of the Notification and Shipping service) 

* To increase the number of partitions for a topic using Python, we have to use the AdminClientClass
(to test this, I created 2 consumers by run the kafka_consumer code in 2 terminals, each consumer was assigned to 1 of the 2 partitions)
```
from confluent_kafka import Producer, Consumer
from confluent_kafka.admin import AdminClient, NewTopic

import socket
import json
import uuid
import random
import datetime


### MESSAGE DETAILS

order_id = uuid.uuid4().hex[-8:]

order_details = {
        'status': 100, 
        'timestamp': int(datetime.datetime.now().timestamp() * 1000), 
        'order': {
            'extra_toppings': ['Onion', 'Olives'], 
            'sauce': 'Tomato', 
            'cheese': 'Provolone', 
            'main_topping': 'Pepperoni', 
            'customer_id': 'd60c39c7744941eb9227c3e47fa791c9', 
            'username': 'user' + str(random.randint(1000, 9999))
        }
    }


#### KAFKA PRODUCER CONFIG

HOSTNAME = socket.gethostname()

producer_common_config = {
            "partitioner": "murmur2_random",
        }

producer_extra_config={
        "client.id": f"""pizza_client_webapp_{HOSTNAME}""",
    }

config_kafka = {
    'bootstrap.servers': 'localhost:9092'
    }


### KAFKA PRODUCER

PRODUCER = Producer(
            {
                **producer_common_config,
                **config_kafka,
                **producer_extra_config,
            }
        )

### TOPIC WITH MULTIPLE PARTITIONS
admin = AdminClient({"bootstrap.servers": 'localhost:9092'})
pizza_destroyed_topic = NewTopic("pizza-destroyed", num_partitions=2)
topic_results = admin.create_topics([pizza_destroyed_topic])
print(topic_results)
# OUTPUT: {'pizza-ordered': <Future at 0x275b8ec2c90 state=running>}

cluster_metadata = admin.list_topics()
print(cluster_metadata.topics)

PRODUCER.produce(
            "pizza-destroyed",
            key=order_id,
            value=json.dumps(order_details).encode(),
        )

PRODUCER.flush()

```

* Errors
  * The session is unavailable because no secret key was set. Set the secret_key on the application to something unique and secret.
  * The method is not allowed for the requested URL. (because only GET request defined for that endpoint but tried doing a POST request to )
  * TypeError: expected list of topic unicode strings , CONSUMER.subscribe("pizza-ordered") to 
CONSUMER.subscribe(["pizza-ordered"])



### Doubts
1. What is diff bw event sourcing and SCD2?
2. How does Kafka notify a consumer?
3. What is long polling in Kafka?
4. What is the need of consumer group in kafka? What if 1 consumer goes down?
5. WHat is the difference bw fetching and polling in Kafka? (check 7)
6. What is partition rebalancing and when is it useful?
7. What is heartbeat in Kafka?
8. What is kafka strimzi operator?
9. What is replication factor?
10. What is future?
11. Why do we need consumer groups? (refer 14)
12. Can consumer from different gorups read from same partition? (yes, refer 15)
13. How to create multiple instances in google cloud platform/app engine multiple services?
14. Why do we need to send message from a particular consumer to the same partition? In other words what is the use case of Kafka partion key?
15. How important is it to preserve event order in a microservice based architecture?
16. How to ensure data consistency in a microservice architecture?
17. What is database per service pattern?
18. What is Ray and how can it help create distributed consumer?
19. Is it possible to create a kafka topic with dynamic partition count?

### References
1. https://stackoverflow.com/questions/52273832/how-to-push-notify-consumers-from-kafka-message-bus
2. https://stackoverflow.com/questions/44239027/how-to-view-kafka-messages
3. https://stackoverflow.com/questions/51760214/how-to-restart-docker-for-windows-process-in-powershell
4. https://www.baeldung.com/kafka-commit-offsets
5. https://stackoverflow.com/questions/37943372/kafka-consumer-poll-behaviour
6. https://www.confluent.io/blog/kafka-producer-and-consumer-internals-4-consumer-fetch-requests
7. https://learn.conduktor.io/kafka/kafka-consumer-important-settings-poll-and-internal-threads-behavior/
8. https://quix.io/blog/kafka-auto-offset-reset-use-cases-and-pitfalls
9. https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html
10. https://newrelic.com/blog/best-practices/kafka-consumer-config-auto-commit-data-loss
11. https://stackoverflow.com/questions/59291018/how-to-get-message-from-a-kafka-topic-with-a-specific-offset
12. https://www.redpanda.com/guides/kafka-performance-kafka-rebalancing
13. https://stackoverflow.com/questions/64441716/how-to-create-or-set-number-of-partitions-for-a-topic
14. https://codingharbour.com/apache-kafka/what-is-a-consumer-group-in-kafka/
15. https://stackoverflow.com/questions/35561110/can-multiple-kafka-consumers-read-same-message-from-the-partition
16. https://blog.devgenius.io/preserving-event-order-in-a-microservices-based-architecture-7679723312aa
17. https://stackoverflow.com/questions/75390937/what-is-the-need-of-consumer-group-in-kafka
18. https://www.confluent.io/learn/kafka-partition-key
19. https://github.com/ftisiot/flask-apache-kafka-demo/tree/main (https://www.youtube.com/watch?v=hfi_ALPlsOQ)
20. https://medium.com/@jhansireddy007/how-to-parallelise-kafka-consumers-59c8b0bbc37a
21. https://stackoverflow.com/questions/35437681/kafka-get-partition-count-for-a-topic
22. https://stackoverflow.com/questions/32761598/is-it-possible-to-create-a-kafka-topic-with-dynamic-partition-count?rq=3
23. https://stackoverflow.com/questions/64441716/how-to-create-or-set-number-of-partitions-for-a-topic
24. https://www.youtube.com/watch?v=j1KQY33IqRc (The Python AdminClient Class)
25. https://stackoverflow.com/questions/33677871/is-it-possible-to-add-partitions-to-an-existing-topic-in-kafka-0-8-2
26. https://github.com/Sumanshu-Nankana/kafka-python (https://www.youtube.com/watch?v=gLMXVzEkahc)
27. https://github.com/ifnesi/python-kafka-microservices/tree/main
28. https://pandeyshikha075.medium.com/getting-started-with-confluent-kafka-in-python-579b708801e7



### Extras
1. https://www.youtube.com/watch?v=RHwglGf_z40&t=1529s - Patroni
2. https://www.youtube.com/watch?v=fvpq4jqtuZ8 - Connecting to database outside Kubernetes
3. Check out Signoz 
4. Check out - Best Practices for using Docker in Production
 https://www.youtube.com/watch?v=8vXoMqWgbQQ&t=34s
5. Implement different microservice architectures described here - https://redis.io/solutions/microservices/

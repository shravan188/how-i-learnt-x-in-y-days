

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

### References
1. https://www.youtube.com/watch?v=CESKgdNiKJw
2. https://github.com/donnemartin/system-design-primer
3. https://fastapi.tiangolo.com/deployment/docker/
4. https://web.archive.org/web/20221030091841/http://www.lecloud.net/tagged/scalability/chrono
5. https://www.youtube.com/watch?v=mI73eTlSqeU&list=PL6W8uoQQ2c63W58rpNFDwdrBnq5G3EfT7 (Concept and Coding By Shrayansh Jain)


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

### Extras
1. https://www.youtube.com/watch?v=RHwglGf_z40&t=1529s - Patroni

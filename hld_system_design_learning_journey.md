

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

### Doubts
1. HAProxy vs Flask - when to use which?
2. Why can we use HAProxy to handle HTTP traffic?
3. What is the different bw different Python docker images like alpine, slimbookworm etc?
4. Why we need to mention host='0.0.0.0' in app.run?
5. How to use Dockerfile image in docker-compose?
6. What is Difference between frontend/backend and listen in haproxy?
7. What is dig in Ubuntu, and how to run it on windows?
8. Why use pgbouncer?
### References
1. https://www.freecodecamp.org/news/how-to-dockerize-a-flask-app/
2. https://github.com/docker/compose/issues/9441
3. https://www.youtube.com/watch?v=PtT32MW2j9c
4. https://stackoverflow.com/questions/39209917/difference-between-frontend-backend-and-listen-in-haproxy
5. https://stackoverflow.com/questions/5673335/how-to-configure-haproxy-to-send-get-and-post-http-requests-to-two-different-app
6. https://github.com/jonnylangefeld/docker-load-balance-test/tree/remove-unnecessary-stuff
7. https://parottasalna.com/2024/09/10/haproxy-ep-2-tcp-proxy-for-flask-application/




### Extres
1. https://www.youtube.com/watch?v=RHwglGf_z40&t=1529s - Patroni
# Data Engineering Zoomcamp (by DataTalks Club)

## Day 1
### Duration : 1.5 hours
* Started the Zoomcamp on 16-Jan-2024

* To run an Ubuntu container in docker : `docker run -it ubuntu bash`.The docker run command automatically pulls and runs the image without the need to run docker pull first. 

```
# to run python 3.9 in containerized environment
docker run -it python:3.9

# when we want to run pip install, we cannot do it directly in the python shell but have to do it in the bash terminal, hence we specify entrypoint as bash
docker run -it --entrypoint=bash python:3.9

```

* Dockerfile : A text file which contains sequence of steps to build a docker image. You can imagine as if this is a terminal where you are typing all your commands, but instead of typing it one at a time, you are typing it all at once and then running it all using `docker build` followed by `docker run` . Below is example of a simple dockerfile

```
### Sample DOCKERFILE
### The folder we are running from just has 2 files excluding DOCKERFILE - hello.py (a simple flask app) and requirements.txt
FROM debian:buster

COPY . /myproject

RUN pip install flask # RUN pip install -r /myproject/requirements.txt

CMD ["python","myproject/app.py"]

```
* Once we create DOCKERFILE, we build it to create docker image, and then run the image to have the container as shown below

```
# build the dockerfile using docker build .
# by using tag, it is easier to refer to image, compared to using the image id
docker build --tag hello-flask:1.0 .

# see list of images available
docker images

# run the image with port binding
docker run -p 5000:5000 <imageid/tag>

```

* Base image : Most Docker images aren’t built from scratch. Instead, you take an existing image, called base image, and use it as the basis for your image using the FROM command in your Dockerfile

* In regular command line, if you cd somewhere, it stays there until you change it. However, in a Dockerfile, *each RUN command starts back at the root directory*. That's a gotcha for docker newbies, and something to be aware of. So not only does WORKDIR make a more obvious visual cue to someone reading your code, but it also keeps the working directory for more than just the one RUN command(similar to cd in normal terminal).

```
### DOCKERFILE with WORKDIR (compare with previous dockerfile)

FROM debian:buster

WORKDIR /myproject

COPY . .

RUN pip install flask # RUN pip install -r requirements.txt

CMD ["python","app.py"]

```

* You should use WORKDIR instead of proliferating instructions like RUN cd … && do-something (RUN cd /myproject && pip install -r requirements.txt), which are hard to read, troubleshoot, and maintain.

* Layer caching : Common layers are not pulled multiple times. For example, if we run docker pull python:3.9 and then docker pull python:3.10, then the layers in 3.10 image common to 3.9 image are not pulled but used directly from 3.9 image. To see how to leverage caching to build more efficient DOCKERFILE see reference 3

* ENTRYPOINT : To use when you want to append some additional command to command entered in docker run.
Assume following ENTRYPOINT in DOCKERFILE
`ENTRYPOINT [ "npm", "init" ]`
Now, If I run docker run -t node install
It will append the `npm init` to `install` and overall command run is `npm init install`



### Doubts
1. What is the difference bw docker-compose run and docker-compose up?
2. To build a Python application is docker, what is difference b/w FROM ubuntu and FROM python:3.9
3. Do all docker images need an OS? Are all Docker images based on linux? Is the python:3.9 image based on linux?
4. If containers do not have a guest OS, then why do we base the container on an OS image such as `FROM ubuntu`?
5. Why use WORKDIR in the DOCKERFILE?
6. In dockerfile with workdir, shouldn't workdir command come after copy?
7. Is the default shell for python:3.9 image a python shell instead of bash terminal?

### References
1. https://stackoverflow.com/questions/46708721/do-all-docker-images-have-minimal-os
2. https://serverfault.com/questions/755607/why-do-we-use-a-os-base-image-with-docker-if-containers-have-no-guest-os
3. https://www.youtube.com/watch?v=1d-LRIZRf5s
4. https://stackoverflow.com/questions/51066146/what-is-the-point-of-workdir-on-dockerfile
5. https://www.youtube.com/watch?v=U1P7bqVM7xM
6. https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile
7. https://www.geeksforgeeks.org/how-to-run-a-python-script-using-docker/



## Day 2
### Duration : 2.25 hours

### Learnings
* Environment variable : a user-definable value that can affect the way running processes will behave on a computer. In docker environment variables will be redeclared in each new containers

* ENV (keyword in dockerfile): Set environment variables with some default value.

* ENV values from the Dockerfile will be overridden via the CLI (using -e flag during docker run or env files)

* ARG (keyword in dockerfile): Create build-time variables

* ENV vs ARG :  ARG is for building your Docker image i.e. a variable defined using ARG can be assigned only during `docker build` and not during `docker run`. ENV is for future running containers. ENV is mainly meant to provide default values for your future environment variables, and these environment variables can be assigned new values during `docker run`

* Docker Volume : Volumes are persistent data stores for containers, created and managed by Docker. In simple words, when you stop and start a container, the data inserted previously is not lost (hence used to solve issue of data persistence)
```
# volume-name is path of data in host system
# mount-path is path of data in docker container
docker run --volume <volume-name>:<mount-path>
```

* The command to create a postgres container with volume is as follows (will be used to store nyc taxi data)

```
# the same thing can also be done using docker-compose.yml
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v C:\Users\Dell\Documents\ny_taxi_postgres_data:/var/lib/postgresql/data
    -p 5432:5432
    postgres:13

```

* Errors faced
    * ``docker: invalid reference format`  Cause: `-v ($pwd)/ny_taxi_postgres_data:var/lib/postgresql/data` and `-e POSTGRES_USER = "root"` Solution : remove space around = 
    * `invalid mount config for type "volume": invalid mount path: 'var/lib/postgresql/data' mount path must be absolute`.
    * `docker: Error response from daemon: create $(pwd)/ny_taxi_postgres_data: "$(pwd)/ny_taxi_postgres_data" includes invalid characters for a local volume name, only "[a-zA-Z0-9][a-zA-Z0-9_.-]" are allowed. If you intended to pass a host directory, use absolute path`. Solution : Give full path instead of $(pwd) in Windows,
    * `ModuleNotFoundError: No module named 'psycopg2' while running upload_data.py ` Solution: pip install psycopg2


```
pip install pgcli

pgcli -h localhost -p 5432 -u root -d ny_taxi

# test connection to running database
SELECT 1; 
```

* Following is the Python code to insert tripdata from csv file into Postgres database

```
### upload_data.py 
import pandas as pd

from sqlalchemy import create_engine
from time import time

df = pd.read_csv('yellow_tripdata_2021-01.csv', nrows=100)

print(pd.io.sql.get_schema(df, name='yellow_taxi_data', con=engine))

# postgresql://user:password@hostname/database_name
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')

df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100000)

while True: 
    t_start = time()

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
    
    df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')

    t_end = time()

    print('inserted another chunk, took %.3f second' % (t_end - t_start))

# Using pgcli we see total of 1369765 records using select count(*) from yellow_taxi_data;

```

### Doubts
1. Where exactly in the DOCKERFILE are the environment variables used when we type `docker run -e POSTGRES_USER=root`
2. What is difference bw ENV and ARG in dockerfile? 


### References
1. https://vsupalov.com/docker-arg-env-variable-guide/
2. https://docs.docker.com/reference/dockerfile/
3. https://stackoverflow.com/questions/33935807/how-to-define-a-variable-in-a-dockerfile
4. https://stackoverflow.com/questions/41916386/arg-or-env-which-one-to-use-in-this-case
5. https://stackoverflow.com/questions/34809646/what-is-the-purpose-of-volume-in-dockerfile
6. https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
7. https://github.com/DataTalksClub/nyc-tlc-data
8. https://stackoverflow.com/questions/45682010/docker-invalid-reference-format
9. https://stackoverflow.com/questions/9353822/connecting-postgresql-with-sqlalchemy

## Day 3

### Duration : 3 hours
### Learnings
* Pgcli : Pgcli is a command line interface for Postgres, built using Python

* Port : A virtual point where network connections start and end. 

* Port number : A 16-bit integer that serves as a unique identifier for a specific proces/service/application on a networked device.

* Port mapping : By default, when we create or run a container using docker create or docker run, containers on bridge networks don't expose any ports to the outside world. The -p flag makes a port available to services outside the bridge network.
```
# -p HOST_PORT:CONTAINER_PORT
-p 8080:80	Map port 8080 on the Docker host to TCP port 80 in the container.
```

* Inspect Postgres database using pgcli as shown below


```
# Connect to running database by providing : Host, port, username, database
pgcli -h localhost -p 5432 -u root -d ny_taxi

select * from information_schema.tables;


select column_name, data_type from information_schema.columns where table_name='yellow_taxi_data'


```

* Pgadmin : A tool for managing Postgres. Provides graphical interface to create and update database objects (tables, schema etc)

* PgAdmin 4 docker container has exposed port 80 and 443 by default (refer DOCKERFILE of pgadmin). Hence we map port 8080 of our computer to port 80 of pgadmin

```

docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    dpage/pgadmin4

```


* Docker Network : A virtual network created by Docker to enable communication between Docker containers. If two processes are running on same machine/container, then no need of network, but two different containers are like 2 separate machines, hence we need to create a network. If two containers are running on the same host they can communicate with each other without the need for ports to be exposed to the host machine.

* TO connect pgadmin to postgres container, we need to create a docker network and connect both the containers to the same docker network
as shown below (environment variables have been e)
```
# create a network
docker network create pg-network

# connect an existing container to the network
docker network connect pg-network competent_germain

# docker rename competent_germain pg_database_1

# see details about network like driver, list of connected containers, etc
docker inspect pg-network

# connect a new container with network using network flag
docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    --network pg-network \
    --name pg-admin \
    dpage/pgadmin4

```

* Steps for using pgadmin are
    1. Login using email and password listed above
    2. Click on Add New Server
    3. Give a name to connection in General tab
    4. In Connection tab, enter following details
        * Host name: IP address of postgres container
        * Port : 5432
        * Username : username of postgres database
        * Password : password of postgres database
    5. Click on Save
    6. Select Query Tool icon at left top 


* `Error : Unable to connect to server, name does not resolve in pgadmin4` Solution: Make sure the postgres container is running. Get the id of the container and do `docker inspect <container-id>`. Copy the IPv4 address and paste it into the Host Name field

* Data serialization : Process of converting an object into a stream of bytes to more easily save or transmit it. 

* YAML (YAML Ain't Markup Language) : A human-readable and human-writable data interchange format for storing and transmitting the information

* YAML has two top-level elements, an object and array
    * Object : a collection of key-value pairs. Each key is followed by :
    * Array : ordered list of values, with each item preceded by -
 JSON and YAML are very similar

* Service : abstract definition of a resource within an application for example a database or a web app frontend or a web app backend

* A service can be run by one or multiple containers. With docker you can handle containers and with docker-compose you can handle services.

* Using docker-compose, we can create the same containers above instead of having to create each container separately, as shown below. Docker compose saves us from writing kilometre long run statement from the terminal, and can be considered as a wrapper around docker cli which makes our lives easier

```
# docker-compose.yml
services:
    pgdatabase:
        image: postgres:13
        environment:
            - POSTGRES_USER=root
            - POSTGRES_PASSWORD=root
            - POSTGRES_DB=ny_taxi
        volumes:
            - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
        ports:
            - "5432:5432"
    
    pgadmin:
        image: dpage/pgadmin4
        environment:
            - PGADMIN_DEFAULT_EMAIL=
            - PGADMIN_DEFAULT_PASSWORD=root
        ports:
            - "8080:80"
        
# To run : docker-compose up -d
# To down: docker-compose down
```

* Useful docker-compose commands

```
# Stop services only
docker-compose stop

# Stop and remove containers, networks..
docker-compose down 

# Down and remove volumes
docker-compose down --volumes 
```
* If we do not use network top level element, docker compose automatically creates a network

* By default docker just manages the volume to host mapping for you i.e. where in the host system the volume is stored.

* (Is this correct?)We add a volume for pgadmin so that we don't have to continually add the connection every time you bring it up. The pgadmin_conn_data volume is separate from your PostgreSQL volume (ny_taxi_postgres_data). The PostgreSQL volume is used to persist data from your PostgreSQL database, while the pgadmin_conn_data volume is used to persist data from your PgAdmin application. 

* Using network top level element in dcoker-compose we can name the network name in the compose file (useful when we want to access it from another container)
```
docker volume ls

docker volume inspect volume-name

docker network ls

```



* 3 major docker concepts : container, network, volume

### Doubts
1. What is difference bw process, service and application?
2. What is differnece bw physical and virtual network?
3. What is the use case for docker network drivers other than bridge (like host, none, etc)?
4. How to add a docker container which has already been created to a new docker network?
5. Why ports in docker-compose wrapped in double quotes? In general when to and not to use double quotes in docker-compose yml file?
6. What is difference bw service and container in docker-compose?
7. For volumes why no need of full path in docker compose for pgdatabase?
8. What is the difference bw volume and bind mount?
9. What is difference bw stop and down in docker-compose command?

### References
1. https://docs.docker.com/engine/network/
2. https://superuser.com/questions/209654/whats-the-difference-between-an-application-a-process-and-a-service
3. https://github.com/pgadmin-org/pgadmin4/blob/master/Dockerfile
4. https://stackoverflow.com/questions/48046585/docker-pgadmin-4
5. https://www.geeksforgeeks.org/basics-of-docker-networking/
6. https://stackoverflow.com/questions/50721424/how-to-add-containers-to-same-network-in-docker
7. https://stackoverflow.com/questions/57109494/unable-to-connect-to-server-pgadmin-4
8. https://deepsource.com/glossary/yaml
9. https://stackoverflow.com/questions/35565770/difference-between-service-and-container-in-docker-compose
10. https://www.reddit.com/r/docker/comments/11fm9zr/help_me_understand_dockercompose_named_volumes/
11. https://kinsta.com/blog/docker-compose-volumes/
12. https://stackoverflow.com/questions/46428420/docker-compose-up-down-stop-start-difference


## Day 4
### Duration : 1 hour

* Cloud Infrastructure : Hardware/software in cloud such as storage, server, compute, networking etc

* Infrastructure as Code(IaC) : Practice where infrastructure is managed and provisioned using code rather than manual processes. By storing infrastructure code in version control system, we can ensure infrastructure changes are trackable

* Terraform : IaC tool which allows users to provision and manage infrastructure resources across various cloud platforms (and even on prem)

* HCL (HashiCorp Configuration Language) : Language used in Terraform

* State file : Stores info about infrastructure's configuration and status (i.e. things like resources that have been created, their current properties) Named `terraform.tfstate `

* Important Terraform CLI commands:
    * `terraform fmt`: Format terraform code
    * `terraform init` : Beginning of project or we decide to add more providers or change the version of existing ones 
    * `terraform plan` : Compares the code with the state file to identify and highlight resources that will be created, updated, or deleted if we choose to execute the current version of the code. ALso goes through configuration files and identify syntax errors/version mismatch
    * `terraform apply` : Actually execute code to create/delete resources on cloud
    * `terraform destroy`

* Basic workflow : init -> plan -> apply

* Plugin :  A piece of software that extends functionality of existing software/provides additional functionality eg. Grammarly, OneTab, Sceenshot of entire page(Chrome), Adblocker

* Provider : Plugin that enables interaction with an API and thus allows it to manage infra on any platform eg. AWS Provider to manage resources on AWS

* Resource :  Infrastructure objects (like virtual networks or compute instances)

* Module : Container for multiple resources that are used together (can include resources from the same provider or different providers)

* Below is code to create a simple GCP Cloud Storage Bucket using Terraform
```
### main.tf

terraform {
    required_providers{
        google = {
            source="hashicorp/google"
            version="5.5.5"
        }
    }
}

provider "google" {
    credentials = "./keys/my-creds.json"
    project = "gcp-project-id"
    region = "us-central1"
}

resource "google_storage_bucket" "demo-bucket" {
    name = "bucket-name" # must be globally unique across all of gcp
    location = "US"
    force_destroy = true

}

```


### Doubts
1. What does declarative syntax mean? And how is it different from imperative?
2. How exactly is terraform cloud agnostic and be used to automate multi cloud deployment? How does it map say cloud storage in GCP to S3 in AWS? 
3. What is the folder structure for terraform?

### References
1. https://spacelift.io/blog/terraform-tutorial
2. https://dzone.com/articles/an-introduction-to-terraforms-core-concepts
3. https://www.reddit.com/r/Terraform/comments/17xcpvq/can_someone_help_me_explain_when_is_terraform/

Folder structure - main.tf (https://spacelift.io/blog/terraform-files)

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

Error because billing account not linked - The billing account for the owning project is disabled in state absent,
curl

## Day 4 and 5
### Duration : 1 + 1 hour
* Apache Airflow and Kestra are an open-source data orchestration and scheduling platforms to manage complex data flows

* Orchestration (remember orchestra where coordination is imp.) : Coordination and management of multiple applications and services, stringing together multiple tasks in order to execute a larger workflow or process

* Data orchestration : Programmatically author, schedule, and monitor workflows.

* Task : Step/action to be performed eg. download data, transform data. Two types
    * Flowable tasks: Tasks which orchestrate the flow eg. io.kestra.plugin.core.flow.Parallel
    * Runnable tasks: Tasks which perform actual work/take some action eg. io.kestra.plugin.scripts.shell.Commands to execute shell commands


* Some properties of task include: id, type,  

* Namespace : Logical grouping of flows eg. based on team

* Flow : Container/Grouping for set of tasks, their inputs and outputs and associated error handling. Defines the order in which tasks are executed and how they are executed (i.e. parellel, sequential)

* Variable : Key value pair to reuse value across tasks

* Namespace variable : Variables whose scope is limited to specific namespace. Namespace variables have to be defined via Kestra UI and not via YML file

* Trigger : Mechanism to automate execution of a flow (schedule driven or event driven)

* States: Control status of your workflow execution

* Template engine : A template engine enables you to use static template files in your application. At runtime, the template engine replaces variables in a template file with actual values

* Pebble : Java templating engine, similar to Jinja in Python

* Expression : Used to dynamically pass data to the workflow in real time. Uses Pebble, hence expression must be wrapped in {{}}. 

* Some variables which exist by default and need not be created explicitly are `flow`, `inputs`, `outputs`, `tasks`. In example below the inputs variable is used

```
## inputs.name is an expression which allows Kestra to use the name passed by end user in real time in the logging task
id: myflow
namespace: company.team

inputs:
    - id: name
      type: STRING

tasks:
    - id: hello
      type: io.kestra.plugin.core.log.log
      message: "Hello {{ inputs.name }}"


```

* If a variable uses a Pebble expression, we must use render() function when using that variable

```
id: myflow
workspace: company.team

variables:
    file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"


tasks:
    - id: set_label
      type: io.kestra.plugin.core.execution.Labels
      labels:
        file: "{{render(vars.file)}}" # since file uses Pebble expression, use render function
        taxi: "{{inputs.taxi}}"

```

* There are multiple types of tasks available, and each has its own set of  properties. Refer Kestra documentation

```
id: postgres-taxi
namespace: zoomcamp

inputs:
    - id: taxi
      displayName: Select taxi type
      values: ['yellow','green']
      defaults: 'yellow'

    - id: year
      displayName: Select year
      values: ['2019','2020']
      defaults: '2019'

    - id: month
      displayName: Select month
      values: ['01','02', '03', '04', '05', '06', '07']
      defaults: '01'


variables:
    file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
    
```

* No need of remembering all the task types, just know the high level logic to implement, rest can be gotten from documentation

* In staging table, we create a unique row id for each row by concatenating all the column values and using md5(instead of using uuid)


### Doubts
1. What is purpose of staging table?
2. What are static files (in context of Jinja)?
3. What is difference between static and dynamic task?
4. What is truncate table in SQL? What is update statement in SQL?
5. What is the difference bw merge and insert in SQL?

### References
1. https://medium.com/geekculture/airflow-vs-prefect-vs-kestra-which-is-best-for-building-advanced-data-pipelines-40cfbddf9697
2. https://expressjs.com/en/guide/using-template-engines.html
3. https://kestra.io/docs/expressions


## W1 Homework (Day 4,5,6)

* Worked on Week 1 Homeowrk

* Question 1
```
-- 24.3.1
docker run -it --entrypoint=bash python:3.12.8
pip -V
```

* Question 2

We can use IP address of host instead of `host.docker.internal`. But in reality, host.docker.internal is not just a hostname that directly points to the IP address of your host. It will point to the “host gateway” which can be configured but it is 172.17.0.1 by default which is the gateway of the default docker bridge. On pinging it will return something like 192.168.65.254 depending on the subnet of Docker Desktop and not 172.17.0.1 because in Docker Desktop the ---host-gateway-ip parameter is set to a proxy ip and forward any request sent to any port to your host machine’s localhost probably using unix sockets and TCP sockets together.

```
docker-compose up -d

pgcli -h localhost -p 5433 -u postgres -d ny_taxi
```


* Question 3
```
-- 104830 ()
select count(*) 
from green_taxi_data
where lpep_pickup_datetime::date >= '2019-10-01' and lpep_pickup_datetime::date < '2019-11-01' and trip_distance <= 1
number not matching any option because only considered pickup time and not drop off time

-- 12 rows not in 2019
select count(*)
from green_taxi_data
where date_part('year', lpep_pickup_datetime::date) <> '2019'

In 2019 Nov data why are there 2008 data? What do we do with dirty data?

-- 104,802; 198,924; 109,603; 27,678; 35,189
select 
	sum(case when trip_distance <= 1 then 1 else 0 end) as "Up to 1 mile",
	sum(case when trip_distance > 1 and trip_distance <=3 then 1 else 0 end) as "1 - 3 miles",
	sum(case when trip_distance > 3 and trip_distance <=7 then 1 else 0 end) as "3 - 7 miles",
	sum(case when trip_distance > 7 and trip_distance <=10 then 1 else 0 end) as "7 - 10 miles",
	sum(case when trip_distance > 10 then 1 else 0 end) as "Over 10 miles"
from green_taxi_data
where lpep_pickup_datetime::date >= '2019-10-01' and 
lpep_pickup_datetime::date < '2019-11-01' and
lpep_dropoff_datetime::date >= '2019-10-01' and 
lpep_dropoff_datetime::date < '2019-11-01' 


-- 104,802; 198,924; 109,603; 27,678; 35,189
-- notice the use of single quotes since these are values and not column names
select 
	case when trip_distance <= 1 then 'Up to 1 mile'
		when trip_distance > 1 and trip_distance <=3 then  '1 - 3 miles'
		when trip_distance > 3 and trip_distance <=7 then  '3 - 7 miles'
		when trip_distance > 7 and trip_distance <=10 then '7 - 10 miles'
		when trip_distance > 10 then 'Over 10 miles'
		end as trip_segment,
	count(*) as trip_count
from green_taxi_data
where lpep_pickup_datetime::date >= '2019-10-01' and 
lpep_pickup_datetime::date < '2019-11-01' and
lpep_dropoff_datetime::date >= '2019-10-01' and 
lpep_dropoff_datetime::date < '2019-11-01' 
group by trip_segment


```
* Question 4

```

-- 2019-10-31, trip distance was 515.89
select lpep_pickup_datetime::date
from green_taxi_data
where trip_distance = (select max(trip_distance) from green_taxi_data)
```

* Question 5

```
-- East Harlem North, East Harlem South, Morningside Heights
select "PULocationID", sum(total_amount)
from green_taxi_data
where lpep_pickup_datetime::date = '2019-10-18'
group by "PULocationID"
having sum(total_amount) > 13000
order by sum(total_amount) desc

-- East Harlem North, East Harlem South, Morningside Heights
with top_pickups as 
(
    select 
        "PULocationID", 
        sum(total_amount)
    from green_taxi_data
    where lpep_pickup_datetime::date = '2019-10-18'
    group by "PULocationID"
    having sum(total_amount) > 13000
    order by sum(total_amount) desc
)
select 
    * 
from 
    top_pickups tp 
left join 
    taxi_zone_lookup tzl
on tp."PULocationID" = tzl."LocationID"

-- JFK Airport
with EastHarlemPU as (
    select *
    from 
    green_taxi_data
    where "PULocationID" = (select "LocationID" from taxi_zone_lookup where "Zone"='East Harlem North')
)
select 
	"Zone"
from 
	EastHarlemPU ehp 
left join 
    taxi_zone_lookup tzl
on ehp."DOLocationID" = tzl."LocationID"
where ehp.tip_amount = (select max(tip_amount) from EastHarlemPU)
```

* Question 6
```
terraform init, terraform apply -auto-approve, terraform destroy
```

* Errors encountered
    * The container name "/pgadmin" is already in use by container "48c18c4276460544bb186b48247a63ff9776514adbbb142429ea14b6a09a0fb5". You have to remove (or rename) that container to be able to reuse that name - Cause: There was already another container with the name pgadmin
    * Unable to connect to server.Connection to server 172.26.0.3 failed (when trying to connect to Postgres from pgadmin New Server connection menu)
    * DtypeWarning: Columns (3) have mixed types. Specify dtype option on import or set low_memory=False.



* We can get date from timestamp by using type casting

* PostgreSQL converts all names (table name, column names etc) into lowercase if you don't prevent it by double quoting them. Hence if we have table or column names with upper case characters, we must put them in double quotes so that postgres does not convert them to lowercase

* Single quotes for string values, double quotes for column, table names

* The terraform apply command performs a plan just like terraform plan does, but then actually carries out the planned changes to each resource using the relevant infrastructure provider's API. It asks for confirmation from the user before making any changes, unless it was explicitly told to skip approval.


### Doubts
1. How to extract date from datetime in Python?
2. WHat does negative trip_dotsance and total_amount mean?
3. What does host.docker.internal mean?
4. How to check netowrk request to docker container? Why when we enter name of container in pgadmin server connection, why is it not working, but host.docker.internal is working?
5. How to get date from timestamp in postgres
6. How to get year from date in postgres? Why is year function not working?
7. Should all column names be in double quotes?

### References
1. https://forums.docker.com/t/host-docker-internal-in-production-environment/137507/2
2. https://stackoverflow.com/questions/31697828/docker-name-is-already-in-use-by-container
3. https://stackoverflow.com/questions/6133107/extract-date-yyyy-mm-dd-from-a-timestamp-in-postgresql
4. https://stackoverflow.com/questions/36203613/how-to-extract-year-from-date-in-postgresql
5. https://stackoverflow.com/questions/55297807/when-do-postgres-column-or-table-names-need-quotes-and-when-dont-they

## Milestone : Submitted Homework 1 on 24-Jan-2025

## Day 7
### Duration : 2 hours
### Learnings
* Learnt about basic GCP concepts

* Project : A project is logical container for organizing and managing resources (provides a secure and isolated environment for deploying applications, storing data, and managing access controls)

* Folder : Folder resources optionally provide an additional grouping mechanism and isolation boundaries between projects. They can be seen as sub-organizations within the organization resource. Folder resources can be used to model different legal entities, departments, and teams within a company.

* Service account : A service account is an account for an application or machine instead of an human user

* IAM : IAM enables us to manage access control by defining who (identity) has what access for which resource(role)

* Identity : Identity represents a human user or programmatic workload that can be authenticated and then authorized to perform actions

* Principal (aka members) : A principal represents an identity (in simple words person) that can access a resource.

* Role :  A set of permissions that allows you to perform specific actions on Google Cloud resources (can be further classified into basic, predefined and custom)

* Policy : Policy defines and enforces what roles are granted to which principals (has allow and deny policy)

* In IAM, permission to access a resource isn't granted directly to the end user. Instead, permissions are grouped into roles, and roles are granted to authenticated principals. For example principal user@example.com is granted the role `roles/bigquery.resourceAdmin` (i.e. BigQuery Admin). Note that if the allow policy is attached to a project, the principals gain the specified roles within the project.

* Steps to generate credentials for Service account : Service Account > Actions > Manage keys > Create New key > JSON . Never ever share service account key as anyone can use our resources and mine bitcoins 

```
# main.tf
terraform {
    required_providers{
        google={
            source="hashicorp/google"
            version="5.6.0"
        }
    }
}

provider "google"{
    credentials = "./my-creds.json" // service account creds
    project = "terraform-demo-448805"
    region = "us-central1"
}

# demo-bucket need not be globally unique, but name must be globally unique
resource "google_storage_bucket" "demo-bucket" {
  name          = "terraform-demo-448805-terra-bucket"
  location      = "US"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 3
    }
    action {
      type = "Delete"
    }
  }

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}

### Commands to run
# terraform init 
# terraform plan 
# terraform apply
```
* Refer 4 to download terraform. Unzip terraform and and copy terraform.exe file to folder where you want to run the terraform commands 

* terraform init creates .terraform.lock.hcl lock file, and it records the provider selections made

* terraform apply creates a terraform.tfstate file 

* State files : Terraform state files contain each and every detail of any resources along with their current status whether it is “ACTIVE”, “DELETED” or “PROVISIONING” etc.

* IaC like Terraform saves us hours of clickops

* OpenTofu is a good alternative for Terraform and will be getting more popular in the coming years

* Errors faced: 
    * googleapi: Error 403: The billing account for the owning project is disabled in state absent, accountDisabled

### Doubts
1. What is the difference bw principal and role with an example?
2. Which is better - terraform or directly using GCP API to programmatically create resources on GCP?
3. What is difference bw project and folder in GCP, and why need for 2 separate concepts?

### References
1. https://cloud.google.com/iam/docs/overview
2. https://cloud.google.com/iam/docs/roles-overview
3. https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/storage_bucket.html
4. https://www.youtube.com/watch?v=dA6WqakJOts
5. https://developer.hashicorp.com/terraform/install
6. https://www.reddit.com/r/Terraform/comments/17xcpvq/can_someone_help_me_explain_when_is_terraform/
7. https://www.reddit.com/r/googlecloud/comments/1e7umtt/terraform_vs_api/
8. https://www.reddit.com/r/devops/comments/cb7rr8/gcp_api_vs_terraform/
9. https://eitca.org/cloud-computing/eitc-cl-gcp-google-cloud-platform/introductions/the-essentials-of-gcp/examination-review-the-essentials-of-gcp/what-is-the-role-of-a-gcp-project-and-what-resources-can-you-provision-within-it/
10. https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy

## Day 8
### Duration : 1.5 hours
### Learnings

* To install Kestra using Docker Compose, refer to this docker-compose.yml file ; https://github.com/kestra-io/kestra/blob/develop/docker-compose.yml

```
# docker-compose.yml (to run kestra)
# along with this you need another docker-compose.yml to run the postgres database which will store the data, this yml file was place in another folder
volumes:
  postgres-data:
    driver: local
  kestra-data:
    driver: local

services:
  postgres:
    image: postgres
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: kestra
      POSTGRES_USER: kestra
      POSTGRES_PASSWORD: k3str4
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      interval: 30s
      timeout: 10s
      retries: 10

  kestra:
    image: kestra/kestra:latest
    pull_policy: always
    # Note that this setup with a root user is intended for development purpose.
    # Our base image runs without root, but the Docker Compose implementation needs root to access the Docker socket
    # To run Kestra in a rootless mode in production, see: https://kestra.io/docs/installation/podman-compose
    user: "root"
    command: server standalone
    volumes:
      - kestra-data:/app/storage
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/kestra-wd:/tmp/kestra-wd
    environment:
      KESTRA_CONFIGURATION: |
        datasources:
          postgres:
            url: jdbc:postgresql://postgres:5432/kestra
            driverClassName: org.postgresql.Driver
            username: kestra
            password: k3str4
        kestra:
          server:
            basicAuth:
              enabled: false
              username: "admin@kestra.io" # it must be a valid email address
              password: kestra
          repository:
            type: postgres
          storage:
            type: local
            local:
              basePath: "/app/storage"
          queue:
            type: postgres
          tasks:
            tmpDir:
              path: /tmp/kestra-wd/tmp
          url: http://localhost:8080/
    ports:
      - "8080:8080"
      - "8081:8081"
    depends_on:
      postgres:
        condition: service_started

```

* Common types of orchestration in the industry:
    * Data Pipeline orchestration :  ETL workflow where data is extracted from a source, transformed, and then loaded into a database. The orchestrator ensures these steps happen in sequence
    * CI/CD pipeline orchestration : CI/CD pipeline involving tasks like compiling code, running tests, deploying to a staging environment, and triggering manual approval for production deployment. Orchestrator ensures that each task runs in the correct order 
    * Cloud Infra orchestration : When deploying a new environment in the cloud, an orchestrator manages the provisioning of servers, databases, and network configurations. It ensures that all resources are created in the right order.


* TRUNCATE : Truncate statement in SQL is used to empty all data from a table (but will not delete table)

* DELETE : Delete statement deletes a table

```
DROP TABLE table_name;

TRUNCATE TABLE table_name;

```

* UPDATE : To update value of an already existing row. SET keyword helps select columns, WHERE clause helps select rows we want to update
```
UPDATE customers
SET contact_name = "Rohan", city = "Bangalore"
WHERE customer_id = 109

```
* MERGE : MERGE statement in SQL is used to perform insert, update, and delete operations on a target table based on the results of JOIN with a source table. This allows users to synchronize two tables by performing operations on one table based on results from the second table.

```
-- target table is the final table which we will use, source table is more of a temo/staging table
MERGE target_table t
USING source_table s
ON t.product_id = s.product_id
-- for those rows for which product already exist in original table
WHEN MATCHED
THEN UPDATE t.product_price = s.product_price
-- for those rows for which product does not exist in original table
WHEN NOT MATCHED BY t
THEN INSERT (product_id, product_name, product_price) VALUES (s.product_id, s.product_name, s.product_price)
-- for those rows for which product does not exist in updated list/table delete as those products no longer exist
WHEN NOT MATCHED BY s
THEN DELETE

```
* Merge statement was introduced in Postgres 15, prior to that upsert with on conflict used to be used to do the same thing

* Ran a simple Kestra flow to create a staging table by typing the following code in the editor (go to Kestra UI -> Flows -> Create)
```
id: 02_postgres_taxi
namespace: zoomcamp
description: |
  The CSV Data used in the course: https://github.com/DataTalksClub/nyc-tlc-data/releases

inputs:
  - id: taxi
    type: SELECT
    displayName: Select taxi type
    values: [yellow, green]
    defaults: yellow

  - id: year
    type: SELECT
    displayName: Select year
    values: ["2019", "2020"]
    defaults: "2019"

  - id: month
    type: SELECT
    displayName: Select month
    values: ["01", "02", "03", "04", "05", "06", "07", "08", "09", "10", "11", "12"]
    defaults: "01"

variables:
  file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
  staging_table: "public.{{inputs.taxi}}_tripdata_staging"
  table: "public.{{inputs.taxi}}_tripdata"
  data: "{{outputs.extract.outputFiles[inputs.taxi ~ '_tripdata_' ~ inputs.year ~ '-' ~ inputs.month ~ '.csv']}}"

tasks:
  - id: set_label
    type: io.kestra.plugin.core.execution.Labels
    labels:
      file: "{{render(vars.file)}}"
      taxi: "{{inputs.taxi}}"

  - id: extract
    type: io.kestra.plugin.scripts.shell.Commands
    outputFiles:
      - "*.csv"
    taskRunner:
      type: io.kestra.plugin.core.runner.Process
    commands:
      - wget -qO- https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{{inputs.taxi}}/{{render(vars.file)}}.gz | gunzip > {{render(vars.file)}}

  - id: if_yellow_taxi
    type: io.kestra.plugin.core.flow.If
    condition: "{{inputs.taxi == 'yellow'}}"
    then:
      - id: yellow_create_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              tpep_pickup_datetime   timestamp,
              tpep_dropoff_datetime  timestamp,
              passenger_count        integer,
              trip_distance          double precision,
              RatecodeID             text,
              store_and_fwd_flag     text,
              PULocationID           text,
              DOLocationID           text,
              payment_type           integer,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              congestion_surcharge   double precision
          );

pluginDefaults:
  - type: io.kestra.plugin.jdbc.postgresql
    values:
      url: jdbc:postgresql://host.docker.internal:5432/postgres-zoomcamp
      username: kestra
      password: k3str4



```

* Tested that table was created by using pg cli

```
pgcli -h localhost -p 5432 -u kestra -d postgres-zoomcamp 

SELECT 1 FROM yellow_tripdata;

```

* Errors encountered
    * yellow_create_table.then[0].url: must not be null - Cause : had not added the pluginDefaults field, which is required to connect to postgres database
    * Connection to host.docker.internal:5432 refused - Cause : postgres table to store data had not been created




### Doubts
1. What is the difference bw automation and orchetration?
2. Can cron jobs replace an orchestrator?
3. What is the difference bw merge, update and upsert?

### References
1. https://www.reddit.com/r/dataengineering/comments/xwgkil/ask_dataengineering_does_anyone_do_orchestration/?rdt=54460
2. https://kestra.io/blogs/2024-09-18-what-is-an-orchestrator
3. https://www.crunchydata.com/blog/a-look-at-postgres-15-merge-command-with-examples


## Day 9

### Duration : 1 hour
### Learnings

* To create a unique id for each row, we concatenate all the columns and then do a MD5 hash. We do not use a uuid so that every time we run the code, we want the same id to be generated, which will not be the case with uuid as a different uuid is generated each time, hence the same row will be assigned different uuid if we run the insertion code on the same file twice, leading to duplicates

```
-- create a unique id for each row
UPDATE yellow_tripdata_staging
SET unique_row_id = md5(
    COALESCE(CAST(VendorID AS text),'') ||
    COALESCE(CAST(tpep_pickup_datetime AS text), '') || 
    COALESCE(CAST(tpep_dropoff_datetime AS text), '') || 
    COALESCE(PULocationID, '') || 
    COALESCE(DOLocationID, '') || 
    COALESCE(CAST(fare_amount AS text), '') || 
    COALESCE(CAST(trip_distance AS text), '')      
    )

```

* 


```
id: 02_postgres_taxi
namespace: zoomcamp
description: |
  The CSV Data used in the course: https://github.com/DataTalksClub/nyc-tlc-data/releases

inputs:
  - id: taxi
    type: SELECT
    displayName: Select taxi type
    values: [yellow, green]
    defaults: yellow

  - id: year
    type: SELECT
    displayName: Select year
    values: ["2019", "2020"]
    defaults: "2019"

  - id: month
    type: SELECT
    displayName: Select month
    values: ["01", "02", "03", "04", "05", "06", "07", "08", "09", "10", "11", "12"]
    defaults: "01"

variables:
  file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
  staging_table: "public.{{inputs.taxi}}_tripdata_staging"
  table: "public.{{inputs.taxi}}_tripdata"
  data: "{{outputs.extract.outputFiles[inputs.taxi ~ '_tripdata_' ~ inputs.year ~ '-' ~ inputs.month ~ '.csv']}}"

tasks:
  - id: set_label
    type: io.kestra.plugin.core.execution.Labels
    labels:
      file: "{{render(vars.file)}}"
      taxi: "{{inputs.taxi}}"

  - id: extract
    type: io.kestra.plugin.scripts.shell.Commands
    outputFiles:
      - "*.csv"
    taskRunner:
      type: io.kestra.plugin.core.runner.Process
    commands:
      - wget -qO- https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{{inputs.taxi}}/{{render(vars.file)}}.gz | gunzip > {{render(vars.file)}}

  - id: if_yellow_taxi
    type: io.kestra.plugin.core.flow.If
    condition: "{{inputs.taxi == 'yellow'}}"
    then:
      - id: yellow_create_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              tpep_pickup_datetime   timestamp,
              tpep_dropoff_datetime  timestamp,
              passenger_count        integer,
              trip_distance          double precision,
              RatecodeID             text,
              store_and_fwd_flag     text,
              PULocationID           text,
              DOLocationID           text,
              payment_type           integer,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              congestion_surcharge   double precision
          );

      - id: yellow_create_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.staging_table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              tpep_pickup_datetime   timestamp,
              tpep_dropoff_datetime  timestamp,
              passenger_count        integer,
              trip_distance          double precision,
              RatecodeID             text,
              store_and_fwd_flag     text,
              PULocationID           text,
              DOLocationID           text,
              payment_type           integer,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              congestion_surcharge   double precision
          );

      - id: yellow_truncate_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          TRUNCATE TABLE {{render(vars.staging_table)}};

      - id: yellow_copy_in_to_staging_table
        type: io.kestra.plugin.jdbc.postgresql.CopyIn
        format: CSV
        from: "{{render(vars.data)}}"
        table: "{{render(vars.staging_table)}}"
        header: true
        columns: [VendorID,tpep_pickup_datetime,tpep_dropoff_datetime,passenger_count,trip_distance,RatecodeID,store_and_fwd_flag,PULocationID,DOLocationID,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,improvement_surcharge,total_amount,congestion_surcharge]

      - id: yellow_add_unique_id_and_filename
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          UPDATE {{render(vars.staging_table)}}
          SET 
            unique_row_id = md5(
              COALESCE(CAST(VendorID AS text), '') ||
              COALESCE(CAST(tpep_pickup_datetime AS text), '') || 
              COALESCE(CAST(tpep_dropoff_datetime AS text), '') || 
              COALESCE(PULocationID, '') || 
              COALESCE(DOLocationID, '') || 
              COALESCE(CAST(fare_amount AS text), '') || 
              COALESCE(CAST(trip_distance AS text), '')      
            ),
            filename = '{{render(vars.file)}}';

      - id: yellow_merge_data
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          MERGE INTO {{render(vars.table)}} AS T
          USING {{render(vars.staging_table)}} AS S
          ON T.unique_row_id = S.unique_row_id
          WHEN NOT MATCHED THEN
            INSERT (
              unique_row_id, filename, VendorID, tpep_pickup_datetime, tpep_dropoff_datetime,
              passenger_count, trip_distance, RatecodeID, store_and_fwd_flag, PULocationID,
              DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount,
              improvement_surcharge, total_amount, congestion_surcharge
            )
            VALUES (
              S.unique_row_id, S.filename, S.VendorID, S.tpep_pickup_datetime, S.tpep_dropoff_datetime,
              S.passenger_count, S.trip_distance, S.RatecodeID, S.store_and_fwd_flag, S.PULocationID,
              S.DOLocationID, S.payment_type, S.fare_amount, S.extra, S.mta_tax, S.tip_amount, S.tolls_amount,
              S.improvement_surcharge, S.total_amount, S.congestion_surcharge
            );

  - id: if_green_taxi
    type: io.kestra.plugin.core.flow.If
    condition: "{{inputs.taxi == 'green'}}"
    then:
      - id: green_create_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              lpep_pickup_datetime   timestamp,
              lpep_dropoff_datetime  timestamp,
              store_and_fwd_flag     text,
              RatecodeID             text,
              PULocationID           text,
              DOLocationID           text,
              passenger_count        integer,
              trip_distance          double precision,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              ehail_fee              double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              payment_type           integer,
              trip_type              integer,
              congestion_surcharge   double precision
          );

      - id: green_create_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.staging_table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              lpep_pickup_datetime   timestamp,
              lpep_dropoff_datetime  timestamp,
              store_and_fwd_flag     text,
              RatecodeID             text,
              PULocationID           text,
              DOLocationID           text,
              passenger_count        integer,
              trip_distance          double precision,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              ehail_fee              double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              payment_type           integer,
              trip_type              integer,
              congestion_surcharge   double precision
          );

      - id: green_truncate_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          TRUNCATE TABLE {{render(vars.staging_table)}};

      - id: green_copy_in_to_staging_table
        type: io.kestra.plugin.jdbc.postgresql.CopyIn
        format: CSV
        from: "{{render(vars.data)}}"
        table: "{{render(vars.staging_table)}}"
        header: true
        columns: [VendorID,lpep_pickup_datetime,lpep_dropoff_datetime,store_and_fwd_flag,RatecodeID,PULocationID,DOLocationID,passenger_count,trip_distance,fare_amount,extra,mta_tax,tip_amount,tolls_amount,ehail_fee,improvement_surcharge,total_amount,payment_type,trip_type,congestion_surcharge]

      - id: green_add_unique_id_and_filename
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          UPDATE {{render(vars.staging_table)}}
          SET 
            unique_row_id = md5(
              COALESCE(CAST(VendorID AS text), '') ||
              COALESCE(CAST(lpep_pickup_datetime AS text), '') || 
              COALESCE(CAST(lpep_dropoff_datetime AS text), '') || 
              COALESCE(PULocationID, '') || 
              COALESCE(DOLocationID, '') || 
              COALESCE(CAST(fare_amount AS text), '') || 
              COALESCE(CAST(trip_distance AS text), '')      
            ),
            filename = '{{render(vars.file)}}';

      - id: green_merge_data
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          MERGE INTO {{render(vars.table)}} AS T
          USING {{render(vars.staging_table)}} AS S
          ON T.unique_row_id = S.unique_row_id
          WHEN NOT MATCHED THEN
            INSERT (
              unique_row_id, filename, VendorID, lpep_pickup_datetime, lpep_dropoff_datetime,
              store_and_fwd_flag, RatecodeID, PULocationID, DOLocationID, passenger_count,
              trip_distance, fare_amount, extra, mta_tax, tip_amount, tolls_amount, ehail_fee,
              improvement_surcharge, total_amount, payment_type, trip_type, congestion_surcharge
            )
            VALUES (
              S.unique_row_id, S.filename, S.VendorID, S.lpep_pickup_datetime, S.lpep_dropoff_datetime,
              S.store_and_fwd_flag, S.RatecodeID, S.PULocationID, S.DOLocationID, S.passenger_count,
              S.trip_distance, S.fare_amount, S.extra, S.mta_tax, S.tip_amount, S.tolls_amount, S.ehail_fee,
              S.improvement_surcharge, S.total_amount, S.payment_type, S.trip_type, S.congestion_surcharge
            );
  
  - id: purge_files
    type: io.kestra.plugin.core.storage.PurgeCurrentExecutionFiles
    description: This will remove output files. If you'd like to explore Kestra outputs, disable it.

pluginDefaults:
  - type: io.kestra.plugin.jdbc.postgresql
    values:
      url: jdbc:postgresql://host.docker.internal:5432/postgres-zoomcamp
      username: kestra
      password: k3str4


```
* Cron job : A cron job is a Linux command used for scheduling tasks to be executed sometime in the future

* Cron format (* * * * *) : set of five fields in a line, indicating when the job should be executed. Fields in order are : minute, hour, day of the month, month, day of the week. * indicates run for every occurence of that unit of time

* Trigger : Triggers automatically start your flow based on events. A trigger can be a scheduled date (schedule trigger), completion of another flow (flow trigger), a new file arrival, a new message in a queue etc

* Backfill : Backfills are replays of missed schedule intervals between a defined start and end date.Useful in the case when we have old data, since triggers will work only going forward, but for older data (i.e. data in the past) we will need to use backfill

* Ran the new flow below. Some changes in the scheduled flow include
    * Addition of trigger field
    * No longer having year or month as input as that can be obtained from trigger
    * Concurrency is set 1 to avoid multiple flows working on the same table at the same time (alternatively can create staging table for each month)
```

id: 02_postgres_taxi_scheduled
namespace: zoomcamp
description: |
  Best to add a label `backfill:true` from the UI to track executions created via a backfill.
  CSV data used here comes from: https://github.com/DataTalksClub/nyc-tlc-data/releases

concurrency:
  limit: 1

inputs:
  - id: taxi
    type: SELECT
    displayName: Select taxi type
    values: [yellow, green]
    defaults: yellow

variables:
  file: "{{inputs.taxi}}_tripdata_{{trigger.date | date('yyyy-MM')}}.csv"
  staging_table: "public.{{inputs.taxi}}_tripdata_staging"
  table: "public.{{inputs.taxi}}_tripdata"
  data: "{{outputs.extract.outputFiles[inputs.taxi ~ '_tripdata_' ~ (trigger.date | date('yyyy-MM')) ~ '.csv']}}"

tasks:
  - id: set_label
    type: io.kestra.plugin.core.execution.Labels
    labels:
      file: "{{render(vars.file)}}"
      taxi: "{{inputs.taxi}}"

  - id: extract
    type: io.kestra.plugin.scripts.shell.Commands
    outputFiles:
      - "*.csv"
    taskRunner:
      type: io.kestra.plugin.core.runner.Process
    commands:
      - wget -qO- https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{{inputs.taxi}}/{{render(vars.file)}}.gz | gunzip > {{render(vars.file)}}

  - id: if_yellow_taxi
    type: io.kestra.plugin.core.flow.If
    condition: "{{inputs.taxi == 'yellow'}}"
    then:
      - id: yellow_create_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              tpep_pickup_datetime   timestamp,
              tpep_dropoff_datetime  timestamp,
              passenger_count        integer,
              trip_distance          double precision,
              RatecodeID             text,
              store_and_fwd_flag     text,
              PULocationID           text,
              DOLocationID           text,
              payment_type           integer,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              congestion_surcharge   double precision
          );

      - id: yellow_create_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.staging_table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              tpep_pickup_datetime   timestamp,
              tpep_dropoff_datetime  timestamp,
              passenger_count        integer,
              trip_distance          double precision,
              RatecodeID             text,
              store_and_fwd_flag     text,
              PULocationID           text,
              DOLocationID           text,
              payment_type           integer,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              congestion_surcharge   double precision
          );

      - id: yellow_truncate_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          TRUNCATE TABLE {{render(vars.staging_table)}};

      - id: yellow_copy_in_to_staging_table
        type: io.kestra.plugin.jdbc.postgresql.CopyIn
        format: CSV
        from: "{{render(vars.data)}}"
        table: "{{render(vars.staging_table)}}"
        header: true
        columns: [VendorID,tpep_pickup_datetime,tpep_dropoff_datetime,passenger_count,trip_distance,RatecodeID,store_and_fwd_flag,PULocationID,DOLocationID,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,improvement_surcharge,total_amount,congestion_surcharge]

      - id: yellow_add_unique_id_and_filename
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          UPDATE {{render(vars.staging_table)}}
          SET 
            unique_row_id = md5(
              COALESCE(CAST(VendorID AS text), '') ||
              COALESCE(CAST(tpep_pickup_datetime AS text), '') || 
              COALESCE(CAST(tpep_dropoff_datetime AS text), '') || 
              COALESCE(PULocationID, '') || 
              COALESCE(DOLocationID, '') || 
              COALESCE(CAST(fare_amount AS text), '') || 
              COALESCE(CAST(trip_distance AS text), '')      
            ),
            filename = '{{render(vars.file)}}';

      - id: yellow_merge_data
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          MERGE INTO {{render(vars.table)}} AS T
          USING {{render(vars.staging_table)}} AS S
          ON T.unique_row_id = S.unique_row_id
          WHEN NOT MATCHED THEN
            INSERT (
              unique_row_id, filename, VendorID, tpep_pickup_datetime, tpep_dropoff_datetime,
              passenger_count, trip_distance, RatecodeID, store_and_fwd_flag, PULocationID,
              DOLocationID, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount,
              improvement_surcharge, total_amount, congestion_surcharge
            )
            VALUES (
              S.unique_row_id, S.filename, S.VendorID, S.tpep_pickup_datetime, S.tpep_dropoff_datetime,
              S.passenger_count, S.trip_distance, S.RatecodeID, S.store_and_fwd_flag, S.PULocationID,
              S.DOLocationID, S.payment_type, S.fare_amount, S.extra, S.mta_tax, S.tip_amount, S.tolls_amount,
              S.improvement_surcharge, S.total_amount, S.congestion_surcharge
            );

  - id: if_green_taxi
    type: io.kestra.plugin.core.flow.If
    condition: "{{inputs.taxi == 'green'}}"
    then:
      - id: green_create_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              lpep_pickup_datetime   timestamp,
              lpep_dropoff_datetime  timestamp,
              store_and_fwd_flag     text,
              RatecodeID             text,
              PULocationID           text,
              DOLocationID           text,
              passenger_count        integer,
              trip_distance          double precision,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              ehail_fee              double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              payment_type           integer,
              trip_type              integer,
              congestion_surcharge   double precision
          );

      - id: green_create_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          CREATE TABLE IF NOT EXISTS {{render(vars.staging_table)}} (
              unique_row_id          text,
              filename               text,
              VendorID               text,
              lpep_pickup_datetime   timestamp,
              lpep_dropoff_datetime  timestamp,
              store_and_fwd_flag     text,
              RatecodeID             text,
              PULocationID           text,
              DOLocationID           text,
              passenger_count        integer,
              trip_distance          double precision,
              fare_amount            double precision,
              extra                  double precision,
              mta_tax                double precision,
              tip_amount             double precision,
              tolls_amount           double precision,
              ehail_fee              double precision,
              improvement_surcharge  double precision,
              total_amount           double precision,
              payment_type           integer,
              trip_type              integer,
              congestion_surcharge   double precision
          );

      - id: green_truncate_staging_table
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          TRUNCATE TABLE {{render(vars.staging_table)}};

      - id: green_copy_in_to_staging_table
        type: io.kestra.plugin.jdbc.postgresql.CopyIn
        format: CSV
        from: "{{render(vars.data)}}"
        table: "{{render(vars.staging_table)}}"
        header: true
        columns: [VendorID,lpep_pickup_datetime,lpep_dropoff_datetime,store_and_fwd_flag,RatecodeID,PULocationID,DOLocationID,passenger_count,trip_distance,fare_amount,extra,mta_tax,tip_amount,tolls_amount,ehail_fee,improvement_surcharge,total_amount,payment_type,trip_type,congestion_surcharge]

      - id: green_add_unique_id_and_filename
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          UPDATE {{render(vars.staging_table)}}
          SET 
            unique_row_id = md5(
              COALESCE(CAST(VendorID AS text), '') ||
              COALESCE(CAST(lpep_pickup_datetime AS text), '') || 
              COALESCE(CAST(lpep_dropoff_datetime AS text), '') || 
              COALESCE(PULocationID, '') || 
              COALESCE(DOLocationID, '') || 
              COALESCE(CAST(fare_amount AS text), '') || 
              COALESCE(CAST(trip_distance AS text), '')      
            ),
            filename = '{{render(vars.file)}}';

      - id: green_merge_data
        type: io.kestra.plugin.jdbc.postgresql.Queries
        sql: |
          MERGE INTO {{render(vars.table)}} AS T
          USING {{render(vars.staging_table)}} AS S
          ON T.unique_row_id = S.unique_row_id
          WHEN NOT MATCHED THEN
            INSERT (
              unique_row_id, filename, VendorID, lpep_pickup_datetime, lpep_dropoff_datetime,
              store_and_fwd_flag, RatecodeID, PULocationID, DOLocationID, passenger_count,
              trip_distance, fare_amount, extra, mta_tax, tip_amount, tolls_amount, ehail_fee,
              improvement_surcharge, total_amount, payment_type, trip_type, congestion_surcharge
            )
            VALUES (
              S.unique_row_id, S.filename, S.VendorID, S.lpep_pickup_datetime, S.lpep_dropoff_datetime,
              S.store_and_fwd_flag, S.RatecodeID, S.PULocationID, S.DOLocationID, S.passenger_count,
              S.trip_distance, S.fare_amount, S.extra, S.mta_tax, S.tip_amount, S.tolls_amount, S.ehail_fee,
              S.improvement_surcharge, S.total_amount, S.payment_type, S.trip_type, S.congestion_surcharge
            );
  
  - id: purge_files
    type: io.kestra.plugin.core.storage.PurgeCurrentExecutionFiles
    description: To avoid cluttering your storage, we will remove the downloaded files

pluginDefaults:
  - type: io.kestra.plugin.jdbc.postgresql
    values:
      url: jdbc:postgresql://host.docker.internal:5432/postgres-zoomcamp
      username: kestra
      password: k3str4

triggers:
  - id: green_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 1 * *"
    inputs:
      taxi: green

  - id: yellow_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 10 1 * *"
    inputs:
      taxi: yellow

```
* To use backfill feature : Flows -> Create -> Flows -> Select the created flow -> Triggers -> Backfill execution -> Select Start and End dates (date of the data for which we want to run the flow)


### Doubts
1. What exactly is a UUID and where is it used? How is it useful since it is randomly generated?

2. Alternative for backfill is to use ForEach loop. How to use ForEach task in Kestra?

### References
1. https://www.reddit.com/r/PostgreSQL/comments/1ckzc8f/uuid_versus_sequence_why_is_a_uuid_bad_for/
2. https://cloud.google.com/scheduler/docs/configuring/cron-job-schedules



## Day 10
### Duration : 0.75 hours

### Learnings
* Google Cloud Storage : Cloud storage is an object storage. Think of it like a directory on your local computer. However, this file system is in the cloud, so it’s infinitely scalable and can be accessed anywhere. GCS can be used as a data lake but it is not a data warehouse. GCS does not care what you put in it, you do not need to specify the data types or anything when loading data into GCS. A data warehouse does require specified data types and fields on write. Generally, we store raw data in GCS, then apply processing to load it into a data warehouse.

* A data lake is just a glorified file system

* Staging table : A temporary location where data from source systems is copied, so that we can process data (eg. add a new column, delete rows) before we load into the actual table. We need the staging table, because we need some place to store the data before we can load into the final table. In a tool like Pandas or Alteryx, we do not need a staging table as the data is already stored in the memory for these tools, and we can just focus on applying transformations

* Key Value store in Kestra is similar to enviroment (.env) files. For example, to create a set of key value pairs to store credentials and configuration info for GCP, we can create a YAML file as shown below (to see steps to create service account and generate a key refer Day 7)

```
# gcp_kv.yml
id: 04_gcp_kv
namespace: zoomcamp

tasks:
  - id: gcp_creds
    type: io.kestra.plugin.core.kv.Set
    key: GCP_CREDS
    kvType: JSON
    value: |
      {
        "type": "service_account",
        "project_id": "...",
      }

  - id: gcp_project_id
    type: io.kestra.plugin.core.kv.Set
    key: GCP_PROJECT_ID
    kvType: STRING
    value: kestra-sandbox # TODO replace with your project id

  - id: gcp_location
    type: io.kestra.plugin.core.kv.Set
    key: GCP_LOCATION
    kvType: STRING
    value: europe-west2

  - id: gcp_bucket_name
    type: io.kestra.plugin.core.kv.Set
    key: GCP_BUCKET_NAME
    kvType: STRING
    value: your-name-kestra # TODO make sure it's globally unique!

  - id: gcp_dataset
    type: io.kestra.plugin.core.kv.Set
    key: GCP_DATASET
    kvType: STRING
    value: zoomcamp

```

* For detailed description on Pros and Cons of Alteryx refer 3

### Doubts
1. Can cloud storage be considered a data lake?
2. What do you think about the comparison: A data lake is to a file system is what a data warehouse is to a relational database?
3. Why does a tool like ALteryx not require a staging table?
4. What is ACID compliance and how do we ensure it? (refer 3)
5. Can we set a cost limit in Google Cloud i.e. cap resource or API consumption based on cost?
6. What is the difference bw using Terraform vs using Kestra to create GCP resources like GCS bucket?

### References
1. https://www.reddit.com/r/dataengineering/comments/txwnlu/what_is_s3_do_you_put_a_data_lake_or_cloud_data/?rdt=56603
2. https://www.reddit.com/r/dataengineering/comments/t4kz8u/wtf_is_a_datalake/
3. https://www.reddit.com/r/dataengineering/comments/14qi60z/anybody_use_alteryx/
4. https://stackoverflow.com/questions/27616776/how-do-i-set-a-cost-limit-in-google-developers-console
5. https://cloud.google.com/billing/docs/how-to/budgets
6. https://www.reddit.com/r/dataengineering/comments/ygieh8/data_engineering_projects_with_template_airflow/
7. https://www.youtube.com/watch?v=KiTg8RPpGG4

## Day 11
### Duration : 3 hour 
### Learnings
* Airflow is an orchestration tool

* Task : Any action that has to be performed. It is basic unit of execution in Airflow, and of 3 types
  * Operator :  A template for a predefined task (eg. PythonOperator for running python function, BashOperator for executing bash command)
  * Sensor: Type of opearator which waits for something to occur and allows worflow to poceed o
  * Taskflow

* Directed Acyclic Graph : Collection of tasks and relationships between those tasks (i.e. order in which tasks will be run)

* DAG Run : An object. It is created any time a DAG is executed, and all tasks inside the DAG Run are then executed. We can have multiple DAG Runs for the same DAG at the same time (all run independently) (eg. For daily frequency, there is a new DAG run object created for each day)

* DAG Run Status : The status  assigned to the DAG Run when all of the tasks are in the one of the terminal states (i.e. success, failed, skipped). The two poosible value are 
  * success if all of the leaf nodes states are either success or skipped
  * failed if any of the leaf nodes state is either failed or upstream_failed

* Catchup : Scheduler by default kicks off a DAG Run for any data interval that has not been run since last data interval. For example if 
  1. a start date is datetime(2025,1,1) and today is 2025-01-21, then all non-triggered DAG Runs between start date and current date are triggered. 
  2. Consider that workflow is suppose to run hourly, but it missed the previous 3 runs. Then the scheduler will invoke the 3 runs which were missed previously
  To avoid this we can set `catchup=False`. When catchup is turned off, the scheduler creates a DAG run only for the latest interval.

* When we have historical data to be ingested, catchup is useful

* Scheduler :  Monitors all tasks and DAGs, then triggers the task instances once their dependencies are complete

* Executor : Handle running of tasks (include sequential executor, local, dask executor, celery, kubernetes executor)

* Bit shift operator (>>) : If we want to execute 2 tasks in a sequence, one after another, we use >> 

* Xcom : Allows tasks to communicate with each other i.e. share data between tasks (by default Tasks are entirely isolated).  Airflow works like this: It will execute Task1, then populate xcom and then execute the next task.

* A Jinja template is simply a text file. It is very similar to a static file like HTML, XML with the only difference that you can add variables and expressions. These variables and expression get replaced with the variable value when the template is rendered i.e. converted to the static HTML or XML file 
```
## template file
<li><a href="{{ item.href }}">{{ item.caption }}</a></li>

## final html file generated on rendering
<li><a href="www.amazon.in/toys">Toys</a></li>

```

* Dynamic DAG: Suppose we have a different file for each country (or even each month), but all have the same format. Instead of creating a new DAG for each file, we can create a single DAG and then use dynamic DAG to generate multiple similar DAGs from the base DAG, one for each input file, using variables. We use DAG Factory to generate that

* Name of the DAG will be what you give in the DAG context manager - in the example below mydag

* Error encountered
  *  Airflow already exist in the db. Exited with code 0 - What you see as exiting is the "init" process (which is absolutely expected). The fact that it exited with 0 exit code is a good sign actually (which means that the init process did it job and completed successfully) (refer 7)
  * Broken DAG: [/opt/airflow/dags/exampledag.py] - TypeError: 'type' object is not subscriptable (refer 14)
  * ERROR - Failed to execute job 15 for task print_astronauts (xcom_pull() got an unexpected keyword argument 'taks_ids'; 280)
  * ERROR - Failed to execute job 26 for task print_astronauts (list indices must be integers or slices, not str; 618) - Because output was list within list, hence had to do person_in_space[0] to get inner list

```
## simple_dag.py
import requests

from datetime import datetime
from airflow import Dataset
from airflow import DAG
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.decorators import task, dag

def get_astronauts():
    number_of_people_in_space = 3
    list_of_people_in_space = [
        {"craft": "ISS", "name": "Oleg Kononenko"},
        {"craft": "ISS", "name": "Nikolai Chub"},
        {"craft": "ISS", "name": "Tracy Caldwell Dyson"}
    ]


    return list_of_people_in_space # return value is populated in xcom

def print_astronauts(ti):
    person_in_space = ti.xcom_pull(task_ids=["get_astronauts"])
    # Output : [[{'craft': 'ISS', 'name': 'Oleg Kononenko'}, {'craft': 'ISS', 'name': 'Nikolai Chub'}, {'craft': 'ISS', 'name': 'Tracy Caldwell Dyson'}]]
    person_in_space = person_in_space[0]
    for person in person_in_space:
        craft = person["craft"]
        name = person["name"]
        print(f"{name} is currently in space flying on the {craft}")

with DAG(
  "mydag",
  start_date=datetime(2024,1,1),
  schedule="@daily",
  catchup=False
) as dag:
    get_astronaut_data = PythonOperator(
        task_id="get_astronauts",
        python_callable=get_astronauts
        )
    print_astronaut_data = PythonOperator(
        task_id="print_astronauts",
        python_callable=print_astronauts
        )
    
    get_astronaut_data >> print_astronaut_data

```

### Doubts
1. What exactly is the constraints file in Airflow and how do we use it?
2. When are xcoms useful in Airflow?
3. What is the project folder structure in airflow - what do folders like dags, logs, plugins do?
4. What is advantage of using pendulum over datetime library in Python?
5. Why use catchup in Airflow dag?
6. When to use catchup vs macros vs dynamic dags in Airflow?
7. What is the use of task decorator in airflow?
8. How is data passed from 1 task to next task in a daag?
9. How does Docker volume map to host file location? How is it that any change we make in host system is immediately reflected in Docker?
10. Is bit shift operator must to setup order in which tasks are executed i.e. specify task dependency?
11. What is Astronomer Airflow?

### References
1. https://www.youtube.com/watch?v=Fl64Y0p7rls (Airflow setup using DOcker)
2. https://www.reddit.com/r/dataengineering/comments/18ad1du/airflow_python_task_vs_custom_hooksoperators/
3. https://jinja.palletsprojects.com/en/stable/templates/
4. https://aws.amazon.com/blogs/big-data/dynamic-dag-generation-with-yaml-and-dag-factory-in-amazon-mwaa
5. https://stackoverflow.com/questions/46059161/airflow-how-to-pass-xcom-variable-into-python-function
6. https://medium.com/@chanon.krittapholchai/apache-airflow-dynamic-dag-with-jinja-ffc1c90910bf
7. https://stackoverflow.com/questions/68714224/airflow-exiting-after-initilalization
8. https://stackoverflow.com/questions/76538956/airflow-backfill-and-catchup-how-is-it-useful
9. https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html
10. https://www.youtube.com/watch?v=IH1-0hwFZRQ (Data With Marc)
11. https://robust-dinosaur-2ef.notion.site/Your-First-DAG-in-5-minutes-5d15bb2c51b044ea9b8266b2ac07c1fe
12. https://github.com/krishnaik06/ETLWeather/blob/main/dags/exampledag.py
13. https://stackoverflow.com/questions/50149085/python-airflow-return-result-from-pythonoperator
14. https://stackoverflow.com/questions/75202610/typeerror-type-object-is-not-subscriptable-python

## Day 12

### Duration : 1.5 hours

### Learnings

* OLTP (Transactional) vs OLAP (Analytical) : OLTP is optimized for transactional processing and OLAP for data analysis and reporting. For example, a Postgres database which stores ATM transactions or ecommerce purchases or text messages is an OLTP database, whereas when multiple OLTP databases are joined together for reporting purpose, and stored into another postgres database, that database in an OLAP database

* Some key differences in OLTP vs OLAP:
  * Historical data
  * Different end users (external customers vs internal analysts)
  * Types of operations on database (INSERT, UPDATE, DELETE vs SELECT)
  * Normalization (Normalized vs Denormalized)

* Data Warehouse :  A data warehouse is a centralized system that aggregates data from multiple sources into a single, central and consistent data store (basically for OLAP)

* Data mart :  Subset of the data in the data warehouse that focuses on a specific business line, department, subject area, or team.

* ETL is the bridge bw OLTP and OLAP. Overall flow is : Operational systems > Staging area > Data warehouse > Data marts (and in between each stage there are multiple transformations)

* EXTERNAL TABLE : A link to data residing in a table outside big query. Once created, external datasets contain tables from a referenced external data source. Data from these tables aren't copied into BigQuery, but queried every time they are used. (say for example a csv file in google storage or a postgres db)

```

CREATE OR REPLACE EXTERNAL TABLE 'taxi-rides-ny.mytaxi.external_yellow_tripdata'
OPTIONS(
  format='CSV',
  uris = ['gs://nyc-tl-data/trip data/yellow_tripdata_2019-*.csv']
)


```
* Since data is not within BigQuery, it cannot determine no. of rows or size of data

* Partitioning : Dividing a table into segments. Major reason to use partition is it makes searching/filtering easier as it has to scan only subset of data (i.e. partition) based on the filter clause rather than scan the entire data

* Clustering : Clustering sorts the table based on user-defined columns

* Clustering is helpful because when rows are sorted, query engine can do data skipping more efficiently and hence has to scan lesser data (linear search vs binary search OR if it has to search one value, as soon as it reaches the last occurence of that value, it can stop there and not scan till the end). It also can get min-max statistics easily. Disdvantage is cost estimate cannot be computed upfront, unlike partitioning


* Partitioning can be done only on 1 column, clustering upto 4 columns as shown below

```
-- not external, as we cannot partition external table
CREATE OR REPLACE TABLE 'taxi-rides-ny.mytaxi.yellow_tripdata_partitioned'
PARTITION BY
DATE(tpep_pickup_datetime) AS
SELECT * FROM taxi-rides-ny.nytaxi.external_yellow_tripdata;

-- Scanning 1.6GB of data
SELECT DISTINCT(VendorID)
FROM taxi-rides-ny.nytaxi.yellow_tripdata_non_partitoned
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2019-06-30';

-- Scanning ~106 MB of DATA
SELECT DISTINCT(VendorID)
FROM taxi-rides-ny.nytaxi.yellow_tripdata_partitioned
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2019-06-30';
```

* To see amount of data in each partition use `INFORMATION_SCHEMA.PARITIONS` as shown below
```
select table_name, partition_id, total_rows
FROM 'nytaxi.INFORMATION_SCHEMA.PARTITIONS'
WHERE table_name = 'taxi-rides-ny.nytaxi.yellow_tripdata_partitioned'
ORDER BY total_rows DESC

```
* Clustering + Partitioning : Very powerful technique, as partitioning reduces amount of data engine has to scan, and by sorting data within each partition, it can scan data more efficiently/ better data skipping, hence further improving performance. This can be done as follows (once data size expands beyond 1gb, then we can see benefits not before)


```

-- Creating a partition and cluster table
CREATE OR REPLACE TABLE taxi-rides-ny.nytaxi.yellow_tripdata_partitioned_clustered
PARTITION BY DATE(tpep_pickup_datetime)
CLUSTER BY VendorID AS
SELECT * FROM taxi-rides-ny.nytaxi.external_yellow_tripdata;

-- Query scans 1.1 GB
SELECT count(*) as trips
FROM taxi-rides-ny.nytaxi.yellow_tripdata_partitoned
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;

-- Query scans 864.5 MB
SELECT count(*) as trips
FROM taxi-rides-ny.nytaxi.yellow_tripdata_partitoned_clustered
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-06-01' AND '2020-12-31'
  AND VendorID=1;

```
* Clustering helps queries that have filters on clustering columns. If you have such a query, compare cost of a query shown after query execution with and without such filter. The ratio is how much clustering saves you.

* Block Pruning : BigQuery sorts the data in a clustered table based on the values in the clustering columns and organizes them into blocks

* When we use clustering over partitioning:
  * When cardinality of column is high, because partitioning will create too many partitions
  * When partioning leads to small size per partition (less than 1 GB)


* Automatic Reclustering : When there is any insert, update or delete in clustered table, Big Query automatically reculsters data

* For loading data from Postgres db into BigQuery database, Airflow is one of the best options, as options with BigQuery to do the same are limited (refer 4)


* Sharding : Sharding and partitioning are both about breaking up a large data set into smaller subsets. The difference is that sharding implies the data is spread across multiple computers while partitioning does not.

### Doubts
1. In what way is OLTP optimized for transactional processing?
2. What is external table in bigquery?
3. WHen to use temporary table?
4. How to load data from a postgres database into Bigquery table?
5. Why does clustering (in other words sorting) columns improve query performance?
6. What is block pruning?
7. Why cant we

### References
1. https://www.stitchdata.com/resources/oltp-vs-olap/
2. https://en.wikipedia.org/wiki/Data_mart
3. https://cloud.google.com/bigquery/docs/datasets-intro#external_datasets
4. https://stackoverflow.com/questions/66901681/streaming-postgresql-tables-into-google-bigquery
5. https://docs.rivery.io/docs/partitioning-and-clustering-in-bigquery
6. https://e6data.com/enhancing-query-performance-apache-iceberg-sorting-within-partitions
7. https://www.reddit.com/r/bigquery/comments/xc2z7z/how_do_i_know_if_clustering_of_a_particular
8. https://cloud.google.com/bigquery/docs/clustered-tables
9. https://www.youtube.com/watch?v=-CqXf7vhhDs&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb
10. https://hoffa.medium.com/bigquery-optimized-cluster-your-tables-65e2f684594b

## Day 13
### Duration : 3.5 hours

### Learnings
* Created GCS bukcet and Big Query dataset using Terraform

```
## main.tf

terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "5.6.0"
    }
  }
}

provider "google" {
  credentials = "./my-creds.json"
  project     = "terraform-demo-448805"
  region      = "us-central1"
}

# demo-bucket need not be globally unique, but name must be globally unique
resource "google_storage_bucket" "demo-bucket" {
  name          = "terraform-demo-448805-terra-bucket"
  location      = "US"
  force_destroy = true

  lifecycle_rule {
    condition {
      age = 3
    }
    action {
      type = "Delete"
    }
  }

  lifecycle_rule {
    condition {
      age = 1
    }
    action {
      type = "AbortIncompleteMultipartUpload"
    }
  }
}

resource "google_bigquery_dataset" "dataset" {
  dataset_id                  = "trips_data_all"
  friendly_name               = "trips_data_all"
  description                 = "This dataset contains trip data"
  location                    = "US"
  default_table_expiration_ms = 3600000
}


### To run the terraform script - terraform plan > terraform apply
```

* Create DAG in Airflow to download zip file from Github, unzip it to csv, convert it from csv to Parquet

```
import os
import logging

from airflow import DAG
from airflow.utils.dates import days_ago
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

from google.cloud import storage
from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateExternalTableOperator
import pyarrow.csv as pv
import pyarrow.parquet as pq

#PROJECT_ID = os.environ.get("GCP_PROJECT_ID")
#BUCKET = os.environ.get("GCP_GCS_BUCKET")
PROJECT_ID = "terraform-demo-448805"
BUCKET = "terraform-demo-448805-terra-bucket"


dataset_file = "yellow_tripdata_2021-01.csv"
dataset_url = f"https://s3.amazonaws.com/nyc-tlc/trip+data/{dataset_file}"
path_to_local_home = os.environ.get("AIRFLOW_HOME", "/opt/airflow/")
parquet_file = dataset_file.replace('.csv', '.parquet')
#BIGQUERY_DATASET = os.environ.get("BIGQUERY_DATASET", 'trips_data_all')
BIGQUERY_DATASET = 'trips_data_all'

def format_to_parquet(src_file):
    if not src_file.endswith('.csv'):
        logging.error("Can only accept source files in CSV format, for the moment")
        return
    table = pv.read_csv(src_file)
    pq.write_table(table, src_file.replace('.csv', '.parquet'))

def get_current_dir():
    print(os.getcwd())

default_args = {
    "owner": "airflow",
    "start_date": days_ago(1),
    "depends_on_past": False,
    "retries": 1,
}

# DAG declaration - using a Context Manager (an implicit way)
with DAG(
    dag_id="data_ingestion_gcs_dag",
    schedule_interval="@daily",
    default_args=default_args,
    catchup=False,
    max_active_runs=1,
    tags=['dtc-de'],
) as dag:

    download_dataset_task = BashOperator(
        task_id="download_dataset_task",
        bash_command=f"curl -sSL https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz | gunzip > {path_to_local_home}/yellow_tripdata_2021-01.csv"
    )

    format_to_parquet_task = PythonOperator(
        task_id="format_to_parquet_task",
        python_callable=format_to_parquet,
        op_kwargs={
            "src_file": f"{path_to_local_home}/{dataset_file}",
        },
    )

    download_dataset_task >> format_to_parquet_task 

```


* To kill Airflow task from UI steps are : DAGs > Select DAG > Graph > Select task > Mark as Failed (refer 10)

* Challenges faced which took time to solve
  * Finding the right bash command. Wget command was not working as not installed. Then took lot of time to find the right curl command to download file from Github and unzip it as csv. Was finally able to figure it (refer 9)
  * Finding where the BashOperator stores the output csv file. Tried searching many articles, some said in /tmp folder, but was not able to find it there. Finally changed the storage path to AIRFLOW_HOME/file_name
  * Finding out which Airflow container has the output file. Airflow has the following services - Scheduler, Triggerer, Worker, Web Server, Redis, Postgres. To find out ran `docker exec -it <container_id> sh` for all the services/containers. Finally found out that Worker container has the csv file and not the other containers
  * Trying to connect Airflow to GCP to create a GCS bucket. Tried using Admin > Connection > Add Connection but was not successful

* To find out file contents of each Airflow service

```
docker ps

docker exec -it bffc64fabd8c sh

ls

```


* To execute a bash script, place it in a location relative to the directory containing the DAG file. So if your DAG file is in /usr/local/airflow/dags/test_dag.py, you can move your test.sh file to any location under /usr/local/airflow/dags/ (Example: /usr/local/airflow/dags/scripts/test.sh) and pass the relative path to bash_command (bash_command="scripts/test.sh")

* Most operators define tasks to be executed in AIRFLOW_HOME, save a few such as PythonVirtualenvOperator and BashOperator that execute inside a temporary folder. For your task (GCSToLocalFilesystemOperator) it's easily the AIRFLOW_HOME.

### Doubts
1. What are bind mounts?
2. Do all docker services use the same set of folders to store data? Or only the volumes shared across all services, and others or not? Why is the output file only stored in worker container, not in others?
3. Is there anyway I can set the working directory in airflow where my codes will run?

### References
1. https://stackoverflow.com/questions/71897448/why-is-airflow-not-recognizing-my-bash-command
2. https://medium.com/@pyramidofmerlin/how-to-maker-airflow-be-able-to-manage-files-in-your-local-computer-371ded7d0804
3. https://stackoverflow.com/questions/61344852/how-to-change-airflows-tmp-data-directory
4. https://airflow.apache.org/docs/apache-airflow/stable/howto/operator/bash.html
5. https://stackoverflow.com/questions/58132463/how-to-change-airflow-home-from-docker-to-local-system
6. https://stackoverflow.com/questions/64866966/cannot-find-local-files-placed-in-airflow-gcstolocalfilesystemoperator
7. https://stackoverflow.com/questions/55292629/is-there-anyway-i-can-set-the-working-directory-in-airflow-where-my-codes-will-r
8. https://stackoverflow.com/questions/53960327/save-result-of-operator-in-apache-airflow
9. https://superuser.com/questions/1235085/how-to-use-gzip-or-gunzip-in-a-pipeline-with-curl-for-binary-gz-files
10. https://stackoverflow.com/questions/43631693/how-to-stop-kill-airflow-tasks-from-the-ui

### TO DOS
1. Create a Dataproc cluster from Airflow - GCP (https://www.youtube.com/watch?v=LkGFyi8S4Ys)
2. Create a Pub Sub with Cloud Function to limit Bills on Google (https://stackoverflow.com/a/65611211)
